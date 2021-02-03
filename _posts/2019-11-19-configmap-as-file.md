---
layout: post
title: 以 File 的方式挂载 Configmap 中的配置
categories: kubernetes
description: 以 File 的方式挂载 Configmap 中的配置
keywords: kubernetes, configmap, file
---

配置文件挂载到 K8s 的 Pod 中有多种方式，可以用 hostPath 的方式将配置文件挂载到容器内，这种方式如果配置文件比较经常修改就不太适用，修改配置文件后，需要重新build 和部署镜像才能使修改生效。比较推荐的是用 configMap 的方式来挂载配置。使用 configMap 方式挂载的配置，可以通过 ```kubectl edit cm xxxx-config``` 来修改配置，然后 ```kubectl delete pods xxxxx``` 后， pod 自动重启后生效配置。  

## 问题
以 configMap 方式挂载配置默认是将 configMap 中的配置挂载到某个 ```目录``` 下，比如下面的配置会把 configMap 中的 config.json 配置挂载到 ```/myproject/config1/config.json``` ，而且有个附带的效果，如果之前有其他的文件通过 ```COPY``` 命令 COPY 到了/myproject/config/下面，这些文件会被抹去。
```yaml
containers:
- image: alpine:latest
  name: myproject
  volumeMounts:
  - mountPath: /myproject/config1/
    name: myproject
volumes:
- name: myproject
  configMap:
    items:
      - key: config.json
        path: config.json
```

### 例子
比如 ```COPY``` 了另外一个配置文件 ```test.json``` 到 /myproject/config1/目录下，然后通过上面的 configMap 方式挂载主要的配置文件 config.json 到 ```/myproject/config1/config.json``` ，结果config1下原本的 ```test.json``` 被删除了。

<img src="/images/posts/configmap/configmap-01.png">
<img src="/images/posts/configmap/configmap-02.png">

## 解决方法
如何才能够以文件的方式来挂载 configMap 中的配置呢？怎么把 configMap 中的配置挂载到 ```/myproject/config1/config.json``` 并且不删除 ```/myproject/config1/``` 目录下的其他配置呢？可以通过 subPath 来实现，添加 ```subPath: config.json``` 配置，同时需要修改 mountPath， ```mountPath: /myproject/config1/``` -> ```mountPath: /myproject/config1/config.json```  详见如下配置：

```yaml
containers:
- image: alpine:latest
  name: myproject
  volumeMounts:
  - mountPath: /myproject/config1/config.json
    subPath: config.json
    name: myproject
volumes:
- name: myproject
  configMap:
    items:
      - key: config.json
        path: config.json
```

通过这种方式挂载后是不会删除 ```/myproject/config1/``` 目录下的其他配置的。
<img src="/images/posts/configmap/configmap-03.png">