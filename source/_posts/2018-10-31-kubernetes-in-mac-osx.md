---
title: Kubernetes in Mac OSX 初探
date: 2018-10-31 16:05:54
tags: [docker, k8s]
categories: tech
---

折腾了许久，终于在我的苹果系统上安装上了kubernetes（k8s），可以开始实战了。

由于国内网络限制原因，你想安装Google资源下的k8s十分困难，你需要VPN，比较简单直接。
开始在Ubuntu Server上折腾了一下，有点麻烦，而我主要是想体会一下使用效果，最后还是决定采用Mac OSX下Docker中的k8s试试，当然它的安装也需要VPN工具。

通过Docker内部自带的k8s安装，简单方便，不需要考虑minikube和virtual box或者其它虚拟化驱动（这里就直接跳过了，毕竟我也没有深入研究）
安装后的k8s是但节点环境，所以默写细节功能与集群有差别，这个我需要慢慢体会。

<!-- more -->

## 启动Docker 下的 Kubernetes
很简单，如图所示，enable apply running
![](/images/post/20181031/enable.png)

## 查看基本信息
查看版本，得到如下信息
```shell
➜  ~ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.3", GitCommit:"2bba0127d85d5a46ab4b778548be28623b32d0b0", GitTreeState:"clean", BuildDate:"2018-05-21T09:17:39Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.3", GitCommit:"2bba0127d85d5a46ab4b778548be28623b32d0b0", GitTreeState:"clean", BuildDate:"2018-05-21T09:05:37Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
➜  ~ kubectl config current-context
docker-for-desktop
```
分别为Client 和 Server版本，其中Platform，client是darwin/amd64，Server是linux/amd64

```shell
➜  ~ kubectl cluster-info
Kubernetes master is running at https://localhost:6443
KubeDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
➜  ~ kubectl get nodes
NAME                 STATUS    ROLES     AGE       VERSION
docker-for-desktop   Ready     master    20h       v1.10.3
```
本机只有一个节点

## Dashboard
管理上，目前我看到两种方式
- 通过kubectl命令的方式进行管理，主要维护方式
- 通过Dashboard进行管理， 主要是查看方式

安装Dashboard：
```shell
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

查看Dashboard所在namespace中pod信息，需要指明namespace，否则查看的是default namespace下的pods：
```shell
➜  ~ kubectl get pods --namespace=kube-system
NAME                                         READY     STATUS    RESTARTS   AGE
etcd-docker-for-desktop                      1/1       Running   5          1d
kube-apiserver-docker-for-desktop            1/1       Running   5          1d
kube-controller-manager-docker-for-desktop   1/1       Running   3          1d
kube-dns-86f4d74b45-r9w7b                    3/3       Running   0          1d
kube-proxy-mzrx2                             1/1       Running   0          1d
kube-scheduler-docker-for-desktop            1/1       Running   3          1d
kubernetes-dashboard-7b9c7bc8c9-zkgzk        1/1       Running   2          1d
```

开启proxy，开始访问dashboard服务
```shell
➜  ~ kubectl proxy
```
浏览器打开：
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

或者使用kubctl prot-forward 将dashboard pod运行的端口暴露出来进行访问
```shell
➜  ~ kubectl port-forward pods/kubernetes-dashboard-7b9c7bc8c9-zkgzk 8443:8443 --namespace kube-system
```
浏览器访问：https://127.0.0.1:8443，注意chrome提示你证书安全问题，直接unsafe方式访问吧

![](/images/post/20181031/start.png)

选择跳过，具体可以做什么，我还没有弄清楚，如果稍后了解，我再更新

我们可以看到纵览页面

![](/images/post/20181031/overview.png)
这里截图是，我已经尝试部署了nginx pod，所以多了一些信息

## 用nginx部署 实战一下
可以通过以下指令，部署一个pod
```shell
➜  ~ kubectl run hello-nginx --image=nginx --port=80

➜  ~ kubectl get deployment
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-nginx   1         1         1            1           1d
```

说到这里就有三个基本概念不得不说了，通过系统页面中给出的连接，我们针对nginx这个部署，看到以下这三个不同的内容
这里我的理解尚浅，可能有描述不准确地方，欢迎告知
- deployment 用于管理 replica sets 和 pods，您只需要在 Deployment 中描述您想要的目标状态是什么，Deployment controller 就会帮您将 Pod 和ReplicaSet 的实际状态改变到您的目标状态。
![](/images/post/20181031/deployment.png) 
- replica set 副本，用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的Pod来替代；而如果异常多出来的容器也会自动回收。
![](/images/post/20181031/replica set.png) 
- pod 是kubernetes中你可以创建和部署的最小也是最简的单位。一个Pod代表着集群中运行的一个进程。pod里面饱含着docker的容器
![](/images/post/20181031/pod.png)
```shell
➜  ~ docker ps | grep nginx
66f968b67cf5  nginx   "nginx -g 'daemon of…"   7 hours ago  Up 7 hours   k8s_hello-nginx_hello-nginx-6584d58b4c-gvk5b_default_f21fbf54-dce9-11e8-b679-025000000001_0
```
请注意这里的6584d58b4c-gvk5b，和对应的pod的名字中的编号一致

好了，这样几个基本概念就都连起来了。

怎么可以访问到这个nginx服务呢，这就需要引入service概念，我们将pod运行的实例暴露成服务
```shell
➜  ~ kubectl expose deployment hello-nginx --type=NodePort --name=hello-nginx-service
```
![](/images/post/20181031/service.png)
命令行可以通过以下指令查看
```shell
➜  ~ kubectl get service                                                             
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-nginx-service   NodePort    10.99.167.149   <none>        8010:32340/TCP   1m
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          2d
```

问题，是我并没有成功访问localhost:32340，这个我需要再看看。

## 安装 istio
根据官方文档，我们可以很快部署istio pods
https://istio.io/docs/setup/kubernetes/quick-start/

下载istio文件后，在其目录下运行
```shell
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
```
![](/images/post/20181031/istio.png)
这部分我还需要继续实验下去，找时间继续更新。

## 总结
kubernetes 的复杂程度超出了我对其的猜测，里面的知识点很多，需要花很多精力去深入进去，初步感觉还是挺有意思的。

参考网站：[wilbeibi](https://wilbeibi.com/2018/02/2018-02-19_kub_mac/) [jimmysong](https://jimmysong.io/kubernetes-handbook/concepts/pod-overview.html)
