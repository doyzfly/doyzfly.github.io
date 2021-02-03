---
layout: post
title: kubectl exec 遇到 unable to upgrade connection Forbidden 的解决办法
categories: kubernetes
description: kubectl exec 遇到 unable to upgrade connection Forbidden 的解决办法
keywords: kubernetes, kubectl, rbac, k8s
---

通过 Kubernetes 进行业务部署时，经常需要登陆到 Pod 里面进行调试，这个时候可以用 kubectl exec 命令来登陆到 Pod 里面进行操作。比如使用这个命令：
```bash
kubectl exec 123456-7890 -c ruby-container -i -t -- bash
```
可以登陆到 Pod=123456-789, container=ruby-container。
由于我使用的 Kubernetes 集群启用了 RBAC, 执行 kubectl exec 返回了如下错误：
```bash
kubectl exec -it frontend-5l2hn bash
error: unable to upgrade connection: Forbidden (user=system:anonymous, verb=create, resource=nodes, subresource=proxy)
```

## 分析问题
- 从返回的错误信息来看，应该是访问权限的问题，是 kubectl 执行权限问题？但是执行 kubectl get pods 等命令时又能正常返回。
    ```bash
    #kubectl get pods
    NAME                                READY   STATUS    RESTARTS   AGE
    frontend-22hmh                      1/1     Running   0          73d
    frontend-4cpbk                      1/1     Running   0          73d
    frontend-5l2hn                      1/1     Running   0          73d
    redis-master-fpks7                  1/1     Running   1          74d
    redis-slave-44ltt                   1/1     Running   1          73d
    redis-slave-jm86r                   1/1     Running   1          73d
    ```  
    **原因**  
    使用 kubectl exec 命令时，会转到kubelet，需要对 apiserver 调用 kubelet API 的授权。所以跟 kubectl 的其他命令有些区别。  

## 解决办法
1. 解决办法1:
    为 kubectl 创建一个用于鉴权的用户信息，并存在 kubeconfig 中，然后使用 RoleBinding 绑定用户权限，这个方法比较复杂，可参考这边文章配置，[创建用户认证授权的kubeconfig文件](https://jimmysong.io/kubernetes-handbook/guide/kubectl-user-authentication-authorization.html)

2. 解决办法2:
    为 system:anonymous 临时绑定一个 cluster-admin 的权限
    ```bash
    kubectl create clusterrolebinding system:anonymous --clusterrole=cluster-admin --user=system:anonymous
    ```
    这个权限放太松了，很危险。 可以只对 anonymous 用户绑定必要权限即可，修改为：
    ```bash
    kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user=system:anonymous
    ```