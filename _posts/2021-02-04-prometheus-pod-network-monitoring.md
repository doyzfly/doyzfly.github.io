---
layout: post
title: Prometheus pod 流量监控
categories: prometheus
description: Prometheus pod 流量监控
keywords: prometheus, pod, network
---


# 1 问题描述
监控某个服务对应 pod 的流量，将 pod 的流量呈现在 dashboard 上，并且作为监控告警的依据。

# 2 解决办法
kube-state-metrics 是 K8S 官方项目，会采集pod、deployment等资源的元信息。使用 `container_network_receive_bytes_total` `container_network_transmit_bytes_total` 来检索进出 pod 的流量。由于 pod 可能会重启，这样检索出来的数据可能会分成多个段，可以简单的使用 sum 这样的函数来聚合。

## 2.1 查询语句

完整的检索语句：

```bash
label_replace(sum by (node)(irate(container_network_receive_bytes_total{pod_name=~"x-service-.*", interface="eth3"}[1m])), "service", "x-service", "", "" )
```


- `irate(container_network_receive_bytes_total{pod_name=~"x-service-.*", interface="eth3"}[1m])` : 计算 `x-service` 这个服务 eth3 网口的进流量。
- `sum by (node)(irate(container_network_receive_bytes_total{pod_name=~"x-service-.*", interface="eth3"}[1m]))` ： 对应 pod 重启的情况，将重启前后多个 pod 的数据做聚合。
- `label_replace` ：用来给查询出来的数据添加 `service:x-service` 的标签。


![prometheus pod network](http://43.243.129.70:8989/pp/blog/posts/prometheus/pod-network.png)

<img src="/images/posts/prometheus/pod-network.png">
