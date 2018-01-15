# Using kubectl to Create a Deployment

- Learn about applications Deployment
- Deploy your first app on Kubernetes with kubectl


## Kubernetes 部署
一旦运行了 Kubernetes 集群，就可以在其上部署集装箱化的应用程序。为此，您创建一个 Kubernetes 部署配置。部署指示 Kubernetes 如何创建和更新应用程序的实例。创建部署后，Kubernetes 主节点计划会将应用程序实例提到群集中的单个节点上。

一旦创建了应用程序实例，Kubernetes 部署控制器将持续监视这些实例。如果托管实例的节点出现故障或被删除，则部署控制器将替换它。这提供了一种自我修复机制来解决机器故障或维护问题。

在 pre-orchestration 的世界中，安装脚本经常用于启动应用程序，但是它们不允许从机器故障中恢复。通过创建应用程序实例并使其跨节点运行，Kubernetes 部署为应用程序管理提供了一种根本不同的方法。

> 部署负责创建和更新应用程序的实例

## Deploying your first app on Kubernetes
![Kubernetes cluster](module_02_first_app.svg)

你可以使用 Kubernetes 命令行界面 Kubectl 创建和管理部署。 Kubectl 使用 Kubernetes API 与集群进行交互。

在创建部署时，您需要指定应用程序的容器映像以及要运行的副本数量。稍后您可以通过更新部署来更改该信息;

> 用程序需要打包到一个支持的容器格式中，以便部署在 Kubernetes 上

对于我们的第一个部署，我们将使用打包在 Docker 容器中的 [Node.js](https://nodejs.org/) 应用程序。源代码和 Dockerfile 在 Kubernetes Bootcamp 的 [GitHub 存储库](https://github.com/kubernetes/kubernetes-bootcamp) 中可用。

## kubectl basics

Like minikube, kubectl comes installed in the online terminal. Type kubectl in the terminal to see its usage. The common format of a kubectl command is: kubectl action resource This performs the specified action (like create, describe) on the specified resource (like node, container). You can use --help after the command to get additional info about possible parameters (kubectl get nodes --help).

Let’s run our first app on Kubernetes with the kubectl run command. The run command creates a new deployment. We need to provide the deployment name and app image location (include the full repository url for images hosted outside Docker hub). We want to run the app on a specific port so we add the  --port parameter:

`kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080`

Great! You just deployed your first application by creating a deployment. This performed a few things for you:

- searched for a suitable node where an instance of the application could be run (we have only 1 available node)
- scheduled the application to run on that Node
- configured the cluster to reschedule the instance on a new Node when needed
To list your deployments use the get deployments command:

`kubectl get deployments`

We see that there is 1 deployment running a single instance of your app. The instance is running inside a Docker container on your node.

### View our app
Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network. When we use kubectl, we're interacting through an API endpoint to communicate with our application.

The kubectl command can create a proxy that will forward communications into the cluster-wide, private network. The proxy can be terminated by pressing control-C and won't show any output while its running.

We will open a second terminal window to run the proxy.

`kubectl proxy`

We now have a connection between our host (the online terminal) and the Kubernetes cluster. The proxy enables direct access to the API from these terminals.

You can see all those APIs hosted through the proxy endpoint, now available at through `http://localhost:8001`. For example, we can query the version directly through the API using the curl command:

`curl http://localhost:8001/version`

The API server will automatically create an endpoint for each pod, based on the pod name, that is also accessible through the proxy.

First we need to get the Pod name, and we'll store in the environment variable POD_NAME:

`export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME`

Now we can make an HTTP request to the application running in that pod:

`curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/`

The url is the route to the API of the Pod.

Note: Check the top of the terminal. The proxy was run in a new tab (Terminal 2), and the recent commands were executed the original tab (Terminal 1). The proxy still runs in the second tab, and this allowed our curl command to work using localhost:8001.

terminal 1
```
$$ sleep 1; ~/.bin/launch.sh
Starting Kubernetes...
Kubernetes Started
$ kubect1 version
bash: kubect1: command not found
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.0", GitCommit:"6e937839ac04a38cac63e6a7a306c5d035fe7b0a", GitTreeState:"clean", BuildDate:"2017-09-28T22:57:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"08e099554f3c31f6e6f07b448ab3ed78d0520507", GitTreeState:"clean", BuildDate:"1970-01-01T00:00:00Z", GoVersion:"go1.7.1", Compiler:"gc", Platform:"linux/amd64"}
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    1m        v1.5.2
$ kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
deployment "kubernetes-bootcamp" created
$ kubectl get deployments
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1         1         1            1           49s
$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "5",
  "gitVersion": "v1.5.2",
  "gitCommit": "08e099554f3c31f6e6f07b448ab3ed78d0520507",
  "gitTreeState": "clean",
  "buildDate": "1970-01-01T00:00:00Z",
  "goVersion": "go1.7.1",
  "compiler": "gc",
  "platform": "linux/amd64"
.metadata.name}}{{"\n"}}{{end}}')pods -o go-template --template '{{range .items}}{{
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-390780338-zg4xg
.metadata.name}}{{"\n"}}{{end}}')ods -o go-template --template '{{range .items}}{{
$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-390780338-zg4xg
$ curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-390780338-zg4xg | v=1
$

```

terminal 2
```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001

```
