# Kubernetes Clusters

* Kubernetes 协调高度可用的计算机集群，并将其作为一个单元进行连接。* Kubernetes 中的抽象允许您将集装箱化的应用程序部署到集群，而不必将它们专门捆绑到单个机器上。为了利用这种新的部署模式，应用程序需要以一种将它们与单个主机分离的方式进行打包：它们需要被集装箱化。与过去的部署模型相比，集装箱化的应用程序更加灵活和可用，应用程序作为深度集成到主机中的软件包直接安装到特定的机器上。* Kubernetes 以更高效的方式自动化跨集群的应用程序容器的分发和调度。* Kubernetes 是一个开放源代码平台，并且可以在生产环境中使用。 Kubernetes 集群由两种类型的资源组成：
- The Master coordinates the cluster
- Nodes are the workers that run applications

## Cluster Diagram

![module_01_cluster](module_01_cluster.svg)

* master 负责管理集群。* 主节点协调集群中的所有活动，如调度应用程序，维护应用程序的所需状态，扩展应用程序以及推出新的更新。

* 节点是用作 Kubernetes 集群中的工作器的 VM 或物理计算机。* 每个节点都有一个 Kubelet，它是管理节点并与 Kubernetes 主站通信的代理。节点也应该有处理容器操作的工具，比如 [Docker](https://www.docker.com/) 或者 [rkt](https://coreos.com/rkt/)。处理生产流量的 Kubernetes 集群应至少有三个节点。

>Masters manage the cluster and the nodes are used to host the running applications.Masters 管理集群，nodes 用于托管正在运行的应用程序。

在 Kubernetes 上部署应用程序时，请通知主服务器启动应用程序容器。主节点将容器安排在集群的节点上运行。这些节点使用主机公开的 Kubernetes API 与主机进行通信。最终用户也可以直接使用 Kubernetes API 与集群进行交互。

Kubernetes 集群可以部署在物理或虚拟机上。要开始 Kubernetes 开发，您可以使用 [Minikube](https://github.com/kubernetes/minikube)。 Minikube 是一个轻量级的 Kubernetes 实现，它在本地机器上创建一个虚拟机，并部署一个简单的只包含一个节点的集群。 Minikube 适用于 Linux，macOS 和 Windows 系统。 Minikube CLI 提供了用于处理集群的基本引导操作，包括启动，停止，状态和删除

```
$ minikkube version
bash: minikkube: command not found
$ minikube version
minikube version: v0.17.1-katacoda
$ minikube start
Starting local Kubernetes cluster...
$ kubectl v
Error: unknown command "v" for "kubectl"

Did you mean this?
        cp
        version

Run 'kubectl --help' for usage.
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
$ kubectl cluster-info
Kubernetes master is running at http://host01:8080
heapster is running at http://host01:8080/api/v1/namespaces/kube-system/services/heapster/proxy
kubernetes-dashboard is running at http://host01:8080/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
monitoring-grafana is running at http://host01:8080/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
monitoring-influxdb is running at http://host01:8080/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    2m        v1.5.2
$

```
