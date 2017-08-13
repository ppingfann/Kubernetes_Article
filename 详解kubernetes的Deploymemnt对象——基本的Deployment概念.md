Kubernetes已经迅速成为快速部署、扩展和管理容器化应用的选择工具。

容器化应用通过给Kubernetes Deployment Controller一个需求状态来将自己部署到Kubernetes中。而当我们给需求状态一个新的声明时，Kubernetes会继续将系统从当前状态改为新需求的状态。

通常会使用yaml文件声明需求状态。在本文中，我们将会通过多种配置来检验Kubernetes Deployment yaml文件中可以被指定修改的部分。

最基本的kubernetes规格说明yaml文件如下所示：

```
apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    template:
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
```

假设如上配置的yaml存储为名为“deployment.yaml”的文件。那么我们可以按如下来初始化一个Deployment到Kubernetes中。

```
$ kubectl create deployment -f deployment.yml
```

让我们通过这个规格说明来准确地了解需求规格文件是如何与Kubernetes通信的：

# apiVersion
apiVersion是一个可选字符串值，它通过yaml规格文件描述了Deployment对象版本的纲要，然而apiVersion不是必不可少的。从实践上来说，Deployment对象是被Kubernetes部署规格所定义的，准确说明Deployment对象的版本信息是为了避免Kubernetes控制器所引起的序列化问题。
在上例中，我们使用"extensions/v1beta1"deployment对象纲要。

# kind
kind是一个可选字符串值，它描述被yaml规格文件定义的资源。Kubernetes可以基于此推断出哪一种类型的规格被提交了。然而，就实践来说，具体说明kind是有好处的，这是为了让规格的读者快速的分辨出正在描述的是什么对象的规格。
在上例中，我们描述了一个Deployment对象，所以kind被设置为“Deployment”。

# metadata
metadata包含持续存在的Kubernetes实体的重要信息。有许多属性可以在metadata中得到具体的描述。接下来本文会对它们进行更深入的介绍。
##metadata.name
当创建或修改一个Kubernetes Deployment时，metadata.name是必不可少的字符串。Kubernetes中的每个Deployment都在命名空间中被规定。每个Deployment在命名空间中被name所唯一标识。所以，在相同命名空间中，不同Deployment不能共享同一个name。

在上例中，我们将Deployment命名为“nginx-deployment”。通过命名deployment，我们可以通过kubectl命令来查询它，例如如下命令：
```
$ kubectl get deployments nginx-deployment
$ kubectl describe deployments nginx-deployment
```
大家可以看这些资料以进一步学习Kubernetes的[命名空间][1]与[name][2]的概念。

若要使用一个不同的Kubernetes命名空间而不使用“默认”的命名空间，你可以简单地指定metadata.namespace属性。示例如下：
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: some-namespace
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
```
## metadata.namespace
metadata.namespace域指定Deployment属于哪个命名空间。命名空间允许你在集群中创建一个独立区域。典型地，对象只能与同一命名空间下的对象进行通信，如果命名空间从你的规格中删除了，那么会使用“default”命名空间。
# spec
spec对象是Deployment规格yaml文件的关键。它是定义Kubernetes Deployment行为的地方。为了Kubernetes产生有意义的行为，很有必要描述spec对象，虽然spec对象不是KubernetesAPI所严格要求的。

spec对象有许多域。稍后将在本文中详述各个域。spec对象的所有域中只有spec.template域是必须定义的。
## spec.template
spec.template是一个必不可少的对象。它描述Kubernetes创造的Pod，Pod是Deployment的一部分。在Kubernetes中运行的容器化应用是跑在Pod中的，Pod是一个逻辑单位，它可以共享存储与管理指令。Pod被统一定位且统一调度，并且运行在共享的空间中。一个Pod可以包含一个或多个容器化应用。
例如，我们可以运行如下命令来查询Pod：
```
$ kubectl get pods
$ kubectl get pods {POD_NAME}
$ kubectl describe pods {POD_NAME}
```
### spec.template.spec
spec.template.spec是spec.template中的一个对象，它为Pod的行为做详细的描述，它包含许多域，接下来会在本文进行描述。在这些域中，只有spec.template.spec.containers域是必须的。
#### spec.template.spec.containers
spec.template.spec.containers是一个数组对象，它必须包含至少一个容器。在这个数组中的每一个containers描述Pod中一个容器化应用的行为。一个Pod中仅有一个容器是非常常见的，然而我们可以为Pod指定任意数量的容器。
##### spec.template.spec.containers[i].name
spec.template.spec.containers[i].name是一个必要的字符串。在Pod中的每一个容器都需要一个独一无二的名字。name作为字符串要遵从DNS_LABEL定义，不能超过63个字符的长度，这要遵守RFCs1035和1123中“label”的定义。命令要符合正则表达式[a-z0–9]([-a-z0–9]*[a-z0–9])。
在上例中，命名我们的容器为“nginx”。
##### spec.template.spec.containers[i].image
spec.template.spec.containers[i].image是一个可选字符串，它规定哪个镜像在已经部署的容器中运行。如此一来，当部署一个Docker镜像为容器化应用时，我们需要包含这个域。

在上例中，我们部署了一个公开的nginx镜像，这个镜像在DockerHub中存储，标签为1.7.9。

我们也可以通过指定一个私有的Docker仓库来部署私有镜像。如下是一个被修改过的spec，它指定一个存储在Distelli的私有仓库中的nginx镜像。
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: nginx-deployment
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: gcr.io/distelli/nginx:1.7.9
```
想应用这个改变到Kubernetes 的Deployment规格中是非常简单的，运行如下命令：
```
$ kubectl apply -f deployment.yml
```
正如我们所见，使用yaml配置文件的短短几行就可以与Kubernetes有大量的交互。请继续关注本系列的下一个部分，探索更多高级的deployment配置，体会不同配置之间的差异。


  [1]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
  [2]: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/
