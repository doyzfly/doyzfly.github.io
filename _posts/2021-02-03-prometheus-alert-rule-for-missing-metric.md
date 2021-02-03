---
layout: post
title: Prometheus 编写告警规则应对 metric 丢失的问题
categories: prometheus
description: Prometheus 编写告警规则应对 metric 丢失的问题
keywords: prometheus, rule, missing metric, alert
---

# 问题描述
Prometheus 很重要的一个功能是监控告警，比如一个服务 X 暴露了两个 metric：A，B，我们需要 metricA 的取值是 0 或 1，当 metricA == 1 时，说明业务有问题，需要触发告警。当服务 X 一直运行是上面的告警逻辑容易实现，`expr: X{metric="metricA"} == 1` 即可搞定。实际情况是，服务 X 本身可能挂掉，导致 prometheus 在服务 X 挂掉这段时间收到的 metricA 是缺失的，这个时候告警规则就不好写了，针对某些 metric 缺失的情况，如何来编写告警规则呢？

![prometheus missing metric](http://43.243.129.70:8989/pp/blog/posts/prometheus/prometheus-missing-metric-01.png)

<img src="/images/posts/prometheus/prometheus-missing-metric-01.png">

# 如何编写告警规则
Prometheus 告警规则编写语法参考官网：[ALERTING RULES](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) 
告警规则示例：

```bash
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```


## 尽量避免丢失 metric
对于不是 metric exporter 服务本身失败的情况，应该尽量避免程序暴露 metric 的时候出现丢失 metric 的情况。官方也是建议尽量避免丢失 metric 的，因为这会对后续很多处理带来不便。[avoid-missing-metrics](https://prometheus.io/docs/practices/instrumentation/#avoid-missing-metrics)


# 解决办法

## 对于 missing metric 的情况设置默认值
这个想法主要来源于 open-falcon ，open-falcon 是 push 模式的监控系统，在 server 侧可以针对某个 metric 值为空的情况设置一个默认值的功能，通常我会把 metric 为空的时候将对应的 value 设置为 -1 这样的值而不是设置为 0，这样在写告警规则的时候容易进行区别判断。

结果是检查了 prometheus 相关的资料，并无此功能。

## 使用 absent 内置函数
PromQL 内置的 absent 函数可以用来判断某个瞬时向量是否存在，看起来有戏。

```bash
absent(v instant-vector) returns an empty vector if the vector passed to it has any elements and a 1-element vector with the value 1 if the vector passed to it has no elements.
This is useful for alerting on when no time series exist for a given metric name and label combination.
```

对于上面描述的那个问题，服务 X 暴露了 metricA，如果有我们有 5 台服务器，每个服务器上面跑了一个服务 X，当我们使用 PromQL 语句（up{job="x-service"}）去检索的时候，会返回包含 5 个instance 的 metric。但是当 absent 函数作用于 up{job="x-service"}，absent(up{job="x-service"}) 会对所有 instance 做聚合处理，只要 5 个 instance 中有一个 instance 的值非空，absent(up{job="x-service"})返回的就是空，只有当所有 instance 的值为空时，absent(up{job="x-service"}) 才会返回 1。但是我们期望 absent 返回的结果是针对 5 个 instance 分别返回值是否为空的情况。目前 absent 无法解决该问题。

![absent](http://43.243.129.70:8989/pp/blog/posts/prometheus/absent-01.png)
![absent](http://43.243.129.70:8989/pp/blog/posts/prometheus/absent-02.png)

<img src="/images/posts/prometheus/absent-01.png">
<img src="/images/posts/prometheus/absent-02.png">


## unless

vector1 unless vector2 会产生一个新的向量，新向量中的元素由vector1中没有与vector2匹配的元素组成。

### offset + unless

```bash
count(up{job="x-service"} offset 1h) by (instance) unless count(up{job="x-service"}) by (instance)
```

使用本 metric offset 然后 unless 来达到类似取反的结果。但是仔细看这个表达是会发现这个表达式无法达到取反的效果，只能解决一部分告警触发的情况，对于告警解除的触发通常是有问题的。

### 取一个每个 instance 都会暴露一个且不会为空的 metric + unless
这个方法关键是要找到一个每个 instance 都会暴露且不会为空的 metric （metricZ）作为 vector1 来跟 metricA 做 unless，这样就相当于对 metricA 取反。为了确保 metricZ 的 label 能够跟 metricA 对应上，可以使用 label_replace 来确保 metricZ 和 metricA 的 instance 这个 label 能够对应上。

```bash
up{job="federate_http"} == 1 UNLESS ON(instance) label_replace(up{type="x-service"}, "instance", "$1:30900", "instance",  "(.*):.*")
```
