# Kubernetes介绍
## Kubernetes是什么？
Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。

### 通过Kubernetes可以：
* 快速部署应用
* 快速扩展应用
* 无缝对接新的应用功能
* 节省资源，优化硬件资源的使用


## Kubernetes 特点

* 可移植: 支持公有云，私有云，混合云，多重云（multi-cloud）
* 可扩展: 模块化, 插件化, 可挂载, 可组合
* 自动化: 自动部署，自动重启，自动复制，自动伸缩/扩展

Kubernetes是Google 2014年创建管理的，是Google 10多年大规模容器管理技术Borg的开源版本。

### 与Swarm、Springcloud和dubbo的区别
 略...
 
## 使用Kubernetes能做什么？

* 多个进程（作为容器运行）协同工作。（Pod）
* 存储系统挂载
* Distributing secrets
* 应用健康检测
* 应用实例的复制
* Pod自动伸缩/扩展
* Naming and discovering
* 负载均衡
* 滚动更新
* 资源监控
* 日志访问
* 调试应用程序
* 提供认证和授权

## Kubernetes不是什么？

Kubernetes并不是传统的PaaS（平台即服务）系统。

* Kubernetes不限制支持应用的类型，不限制应用框架。不限制受支持的语言runtimes (例如, Java, Python, Ruby)，满足12-factor applications 。不区分 “apps” 或者“services”。 Kubernetes支持不同负载应用，包括有状态、无状态、数据处理类型的应用。只要这个应用可以在容器里运行，那么就能很好的运行在Kubernetes上。
* Kubernetes不提供中间件（如message buses）、数据处理框架（如Spark）、数据库(如Mysql)或者集群存储系统(如Ceph)作为内置服务。但这些应用都可以运行在Kubernetes上面。
* Kubernetes不部署源码不编译应用。持续集成的 (CI)工作流方面，不同的用户有不同的需求和偏好的区域，因此，我们提供分层的 CI工作流，但并不定义它应该如何工作。
* Kubernetes允许用户选择自己的日志、监控和报警系统。
* Kubernetes不提供或授权一个全面的应用程序配置 语言/系统（例如，jsonnet）。
* Kubernetes不提供任何机器配置、维护、管理或者自修复系统。

另一方面，大量的Paas系统都可以运行在Kubernetes上，比如Openshift、Deis、Gondor。可以构建自己的Paas平台，与自己选择的CI系统集成。

由于Kubernetes运行在应用级别而不是硬件级，因此提供了普通的Paas平台提供的一些通用功能，比如部署，扩展，负载均衡，日志，监控等。这些默认功能是可选的。

另外，Kubernetes不仅仅是一个“编排系统”；它消除了编排的需要。“编排”的定义是指执行一个预定的工作流：先执行A，之B，然C。相反，Kubernetes由一组独立的可组合控制进程组成。怎么样从A到C并不重要，达到目的就好。当然集中控制也是必不可少，方法更像排舞的过程。这使得系统更加易用、强大、弹性和可扩展。

# Kubernetes基本概念

## Kubernetes 组件

### 1.Master 组件
Master组件提供集群的管理控制中心。

Master组件可以在集群中任何节点上运行。但是为了简单起见，通常在一台VM/机器上启动所有Master组件，并且不会在此VM/机器上运行用户容器。

#### 1.1 kube-apiserver

用于暴露Kubernetes API。资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制

#### 1.2 ETCD

etcd是Kubernetes提供默认的存储系统，保存所有集群数据，使用时需要为etcd数据提供备份计划。

#### 1.3 kube-controller-manager

运行管理控制器，它们是集群中处理常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成单个二进制文件，并在单个进程中运行。

这些控制器包括：
* 节点（Node）控制器。
* 副本（Replication）控制器：负责维护系统中每个副本中的pod。
* 端点（Endpoints）控制器：填充Endpoints对象（即连接Services＆Pods）。
* Service Account和Token控制器：为新的Namespace 创建默认帐户访问API Token。

#### 1.4 cloud-controller-manager
云控制器管理器负责与底层云提供商的平台交互。云控制器管理器是Kubernetes版本1.6中引入的，目前还是Alpha的功能。

云控制器管理器仅运行云提供商特定的（controller loops）控制器循环。可以通过将--cloud-provider flag设置为external启动kube-controller-manager ，来禁用控制器循环。

cloud-controller-manager 具体功能：
* 节点（Node）控制器
* 路由（Route）控制器
* Service控制器
* 卷（Volume）控制器

#### 1.5 kube-scheduler

kube-scheduler 负责资源的调度，监视新创建没有分配到Node的Pod，按照预定的调度策略为Pod选择一个Node。

#### 1.6 插件 addons

插件（addon）是实现集群pod和Services功能的 。Pod由Deployments，ReplicationController等进行管理。Namespace 插件对象是在kube-system Namespace中创建。

* DNS 负责为整个集群提供DNS服务
* Dashboard提供用户界面GUI
* 容器资源监测
* Cluster-level Logging



### 2.节点（Node）组件

节点组件运行在Node，提供Kubernetes运行时环境，以及维护Pod。

#### 2.1 kubelet
负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理
#### 2.2 kube-proxy
负责为Service提供cluster内部的服务发现和负载均衡
#### 2.3 docker
docker用于运行容器
#### 2.4 RKT
rkt运行容器，作为docker工具的替代方案
#### 2.5 supervisord
supervisord是一个轻量级的监控系统，用于保障kubelet和docker运行
#### 2.6 fluentd
fluentd是一个守护进程，可提供cluster-level logging

## Kubernetes对象
Kubernetes对象是Kubernetes系统中的持久实体。Kubernetes使用这些实体来表示集群的状态。一旦创建了对象，Kubernetes系统会确保对象存在。通过创建对象，可以有效地告诉Kubernetes系统你希望集群的工作负载是什么样的。具体来说，他们可以描述：

* 容器化应用正在运行(以及在哪些节点上)
* 这些应用可用的资源
* 关于这些应用如何运行的策略，如重新策略，升级和容错

### 对象（Object）规范和状态
每个Kubernetes对象都包含两个嵌套对象字段，用于管理Object的配置：Object Spec和Object Status
* Spec描述了对象所需的状态 - 希望Object具有的特性
* Status描述了对象的实际状态，并由Kubernetes系统提供和更新

例如，通过Kubernetes Deployment 来表示在集群上运行的应用的对象。创建Deployment时，可以设置Deployment Spec，来指定要运行应用的三个副本。Kubernetes系统将读取Deployment Spec，并启动你想要的三个应用实例 - 来更新状态以符合之前设置的Spec。如果这些实例中有任何一个失败（状态更改），Kuberentes系统将响应Spec和当前状态之间差异来调整，这种情况下，将会开始替代实例

## 描述Kubernetes对象
在Kubernetes中创建对象时，必须提供描述其所需Status的对象Spec，以及关于对象（如name）的一些基本信息。当使用Kubernetes API创建对象（直接或通过kubectl）时，该API请求必须将该信息作为JSON包含在请求body中。通常，可以将信息提供给kubectl .yaml文件，在进行API请求时，kubectl将信息转换为JSON。

以下示例是一个.yaml文件，显示Kubernetes Deployment所需的字段和对象Spec：

nginx-deployment.yaml

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
使用上述.yaml文件创建Deployment，是通过在kubectl中使用kubectl create命令来实现。将该.yaml文件作为参数传递。如下例子：

```
$ kubectl create -f nginx-deployment.yaml --record
```
### 必填字段

对于要创建的Kubernetes对象的yaml文件，需要为以下字段设置值：

* apiVersion - 创建对象的Kubernetes API 版本
* kind - 要创建什么样的对象？
* metadata- 具有唯一标示对象的数据，包括 name（字符串）、UID和Namespace（可选项）

## Kubernetes Names

Kubernetes REST API中的所有对象都用Name和UID来明确地标识。

对于非唯一用户提供的属性，Kubernetes提供labels和annotations。

## kubectl命令列表

* kubectl run（创建容器镜像）
* kubectl expose（将资源暴露为新的 Service）
* kubectl annotate（更新资源的Annotations信息）
* kubectl autoscale（Pod水平自动伸缩）
* kubectl convert（转换配置文件为不同的API版本）
* kubectl create（创建一个集群资源对象
* kubectl create clusterrole（创建ClusterRole）
* kubectl create clusterrolebinding（为特定的ClusterRole创建ClusterRoleBinding）
* kubectl create configmap（创建configmap）
* kubectl create deployment（创建deployment）
* kubectl create namespace（创建namespace）
* kubectl create poddisruptionbudget（创建poddisruptionbudget）
* kubectl create quota（创建resourcequota）
* kubectl create role（创建role）
* kubectl create rolebinding（为特定Role或ClusterRole创建RoleBinding）
* kubectl create service（使用指定的子命令创建 Service服务）
* kubectl create service clusterip
* kubectl create service externalname
* kubectl create service loadbalancer
* kubectl create service nodeport
* kubectl create serviceaccount
* kubectl create secret（使用指定的子命令创建 secret）
* kubectl create secret tls
* kubectl create secret generic
* kubectl create secret docker-registry
* kubectl delete（删除资源对象）
* kubectl edit（编辑服务器上定义的资源对象）
* kubectl get（获取资源信息）
* kubectl label（更新资源对象的label）
* kubectl patch（使用patch更新资源对象字段）
* kubectl replace（替换资源对象）
* kubectl rolling-update（使用RC进行滚动更新）
* kubectl scale（扩缩Pod数量）
* kubectl rollout（对资源对象进行管理）
* kubectl rollout history（查看历史版本）
* kubectl rollout pause（标记资源对象为暂停状态）
* kubectl rollout resume（恢复已暂停资源）
* kubectl rollout status（查看资源状态）
* kubectl rollout undo（回滚版本）
* kubectl set（配置应用资源）
* kubectl set resources（指定Pod的计算资源需求）
* kubectl set selector（设置资源对象selector）
* kubectl set image（更新已有资源对象中的容器镜像）
* kubectl set subject（更新RoleBinding / ClusterRoleBinding中User、Group 或 ServiceAccount）

* https://github.com/AliyunContainerService/k8s-for-docker-desktop 
* https://github.com/AliyunContainerService/minikube