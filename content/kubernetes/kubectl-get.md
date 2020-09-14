---
title: "Kubectl Get"
date: 2020-09-15T01:35:03+08:00
draft: true
Summary: Kubectl get 相关lab测试与集锦
categories:
  - "Kubernetes"
---

### kubectl 基本操作
---
#### 开启kubectl <tab> 自动补全
```bash
[root@docker-host0 ~] yum install bash-completion bash-completion-extras
[root@docker-host0 ~] source <(kubectl completion bash)
```

#### 查看cluster状态
```bash
[root@docker-host0 ~]# kubectl cluster-info
Kubernetes master is running at https://192.168.99.103:8443
KubeDNS is running at https://192.168.99.103:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

#### 查看node
```bash
[root@docker-host0 ~]# kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   21h   v1.14.0
[root@docker-host0 ~]# kubectl get nodes -o wide
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION   CONTAINER-RUNTIME
minikube   Ready    master   21h   v1.14.0   10.0.2.15     <none>        Buildroot 2018.05   4.15.0           docker://18.6.2
```
#### 查看pods
```bash
[root@docker-host0 ~]# kubectl get pods
NAME           READY   STATUS              RESTARTS   AGE
kubia-6bgdw    1/1     Running             0          4h56m
kubia-bm77c    1/1     Running             0          4h56m
kubia-fhb5x    1/1     Running             0          4h52m
kubia-g4rxw    1/1     Running             0          4h52m
kubia-jjc7l    1/1     Running             0          18h
kubia-manual   0/1     ContainerCreating   0          3m13s
[root@docker-host0 ~]# kubectl get pods -o wide
NAME           READY   STATUS              RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
kubia-6bgdw    1/1     Running             0          4h56m   172.17.0.5    minikube   <none>           <none>
kubia-bm77c    1/1     Running             0          4h56m   172.17.0.6    minikube   <none>           <none>
kubia-fhb5x    1/1     Running             0          4h52m   172.17.0.10   minikube   <none>           <none>
kubia-g4rxw    1/1     Running             0          4h52m   172.17.0.15   minikube   <none>           <none>
kubia-jjc7l    1/1     Running             0          18h     172.17.0.4    minikube   <none>           <none>
kubia-manual   0/1     ContainerCreating   0          3m20s   <none>        minikube   <none>           <none>
```

#### 查看RC
```bash
[root@docker-host0 ~]# kubectl get replicationcontrollers 
NAME    DESIRED   CURRENT   READY   AGE
kubia   5         5         5       18h
[root@docker-host0 ~]# kubectl get replicationcontrollers -o wide
NAME    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES          SELECTOR
kubia   5         5         5       18h   kubia        ccieliu/kubia   run=kubia
[root@docker-host0 ~]# kubectl get rc -o wide
NAME    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES          SELECTOR
kubia   5         5         5       18h   kubia        ccieliu/kubia   run=kubia
```

#### 用yaml/json格式查看 object
YAML 语言（发音 /ˈjæməl/ ）
```bash
[root@docker-host0 ~]# kubectl get pod kubia-manual -o yaml
[root@docker-host0 ~]# kubectl get pod kubia-manual -o json
```

#### 使用 kubectl create 命令创建对象

* 这里的`kubia-manual.yaml`在命令执行的同目录下。里面包含任意对象的spec文件。

```bash
[root@docker-host0 ~]# kubectl create -f kubia-manual.yaml
```

#### 查看 pod的运行日志
 * 另外docker查看日志的命令为 `docker logs <container id>`
 * 每天或者每次日志文件到达10MB之后，文件会被Rollover(轮转)。因此这个命令每次查看的都是当前日志，未被轮转的日志。
 * 如果pod被删除之后，日志也随之删除，如需保存大量日志，需要存储到中心存储。后续设置集群的日志系统。
```bash
[root@docker-host0 ~]# kubectl logs kubia-manual
Kubia server starting...
```
* 如果这个pod中包含了多个容器，我们需要使用`-c`命令来进行container名字的指定。
```
[root@docker-host0 ~]# kubectl logs kubia-manual -c kubia
Kubia server starting...
```
#### 处于调试的目的，映射pod的某个port到kubectl 主机上。
* 如下例子中，映射了pod `kubia-manual`的8080 port到了本机的8888 port
* 这个功能只作为调试，而且也只是映射到了127.0.0.1， 换句话说，不支持远程用户访问kubectl host ip地址进行访问pod
```bash
[root@docker-host0 ~]# kubectl port-forward kubia-manual 8888:8080
Forwarding from 127.0.0.1:8888 -> 8080
Forwarding from [::1]:8888 -> 8080
Handling connection for 8888        <<< take a test from another terminal "curl localhost 8888"
```