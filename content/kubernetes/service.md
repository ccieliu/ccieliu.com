---
title: "Kubernetes Service"
date: 2020-09-15T01:50:39+08:00
draft: true
summary: Kubernetes 中 Service 解决了什么问题?
categories:
  - "Kubernetes"
---

### Service
---

#### What's Service?
* Pod可以响应Client或者其他Pod的请求
* Pod随时会被开启或者关闭
* 每个Pod都有随机分配的地址，Pod的地址不是固定的
* 所有相同的Pod提供的服务，地址应该使用一个，而不是多个随机地址
* 为了解决上述问题，K8s提供了资源：Service

#### Service yaml example
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubia
```
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        ports:
        - containerPort: 8080     #这里container开放的port就是service的targetPort
```
* kubia service被分配了一个cluster-ip, 在cluster内部可路由可访问。
* service之间可以用这个地址进行通讯，<u>无需关注pods的随机地址</u>。
* <u>只要service稳定，这个地址不会变</u>。
```bash
[root@docker-host0 kube]# kubectl get all
NAME              READY   STATUS    RESTARTS   AGE
pod/kubia-62nt2   1/1     Running   0          113s
pod/kubia-dhlqw   1/1     Running   0          113s
pod/kubia-qd299   1/1     Running   0          113s

NAME                          DESIRED   CURRENT   READY   AGE
replicationcontroller/kubia   3         3         3       113s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   4h6m
service/kubia        ClusterIP   10.110.84.76   <none>        80/TCP    39s
```

#### 在kubectl 远程执行pod上的命令
```bash
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubia        ClusterIP   10.110.84.76   <none>        80/TCP    13m
[root@docker-host0 kube]# kubectl exec kubia-62nt2 -- curl -s http://10.110.84.76
You've hit kubia-qd299
```
映射一个shell回来？
```bash
[root@docker-host0 kube]# kubectl exec -it kubia-62nt2 -- bash
root@kubia-62nt2:/# id
uid=0(root) gid=0(root) groups=0(root)
```

* 为什么使用两个中横线？ `--`  kubectl命令的结束符，防止后续命令中的option选项和kubectl造成歧义。例如上述例子中的 `-s`

#### 这里有一些需要解决的问题：
 * cluster-ip是随机分配的，也就是说，在部署的时候，需要访问某服务的pod无法知道这个服务的静态ip地址是多少。
 * Client Pods如何知道 某个service 的IP地址？
 * 例如，负责Web的Pods如何知道database service的ip地址是多少？如何学习到其他service的地址？

##### 通过环境变量来学习
* 如果pod建立的时间晚于service，pod创建的时候 环境变量会自动加入已经存在的服务地址和服务port
* 否则使用DNS来解析

##### 通过DNS来学习service
在`kube-system` 的namespace中，有一个dns服务的pod，这个pod知道所有的service和ip地址的对应关系，而且所有的pod中的`resolve.conf`中也添加了这个pod的dns server ip. 所以只需要知道service的name就可以访问对应的服务，而且service的名字是静态的。这就解决了之前提到的问题。

* 一般的，直接访问hostname就可以, 但是完整的FQDN如下。
* 
```bash
root@kubia-jbws8:/# ping kubia
PING kubia.default.svc.cluster.local (10.110.84.76): 56 data bytes
```
#### service ip ping 不通？
* 是的就是ping不通，请直接测试服务例如（curl），不要测试icmp，这毫无意义。
* service ip只有在 ip+port的时候才有意义。没人会响应icmp。

