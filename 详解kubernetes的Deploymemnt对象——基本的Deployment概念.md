kubernetes已经迅速成为容器化应用快速部署、扩缩、管理工具的选择。

容器化应用通过给kubernetes部署控制器一个需求状态来将自己部署到kubernetes中。而当我们给需求状态一个新的声明时，kubernetes会继续将系统从当前状态改为新需求的状态。

将yaml文件作为需求状态的声明是非常常见的。在本文中，我们将会检验不同的配置，这些配置在一个yaml文件中被具体的描述。

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

假设如上配置的yaml存储为名为“deploment.yaml”的文件。那么我们可以按如下来初始化一个deployment到kubernetes中。

```
$ kubectl create deployment -f deployment.yml
```

让我们通过这个规格说明来准确的了解需求规格文件是如何与kubernetes交流的：

# apiVersion
apiVersion是一个可选字符串值，它通过yaml规格文件描述了deployment对象版本的纲要，然而apiVersion不是必不可少的。从实践上来说，deployment对象是被kubernetes部署规格所定义的，准确说明deployment对象的版本信息是为了避免kubernetes控制器所引起的序列化问题。
在上例中，我们使用“extensions/v1beta1”deployment对象纲要。

# kind
kind是一个可选字符串值，它描述被yaml规格文件定义的资源。kubernetes可以基于endpoint推断出哪一个规格被提交了。然而，就实践来说，具体说明kind是有好处的，这是为了让规格的读者快速的分别出什么对象的规格正在被描述。
在上例中，我们描述了一个deployment对象，所以kind被设置为“Deployment”。

# metadata
metadata包含持续存在的kubernetes实体的重要信息。有许多属性可以在metadata中被具体的描述。接下来本文会对它们进行更深入的介绍。
##metadata.name
当创建或修改一个kubernetes deployment时，metadata.name是必不可少的字符串。kubernetes中的每个deployment都在命名空间中被规定。每个deployment在命名空间中被name所唯一标识。所以，在相同命名空间中，不同deployment不能共享同一个name。

在上例中，我们将deployment命名为“nginx-deployment”。通过命名deployment，我们可以通过kubectl命令来查询它，例如如下命令：
```
$ kubectl get deployments nginx-deployment
$ kubectl describe deployments nginx-deployment
```
大家可以看这些资料以进一步学习kubernetes的[命名空间][1]与[name][2]的概念。

若要使用一个不同的kubernetes命名空间而不使用“默认”的命名空间，你可以简单地指定metadata.namespace属性。示例如下：
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
metadata.namespace域指定deployment属于哪个命名空间。命名空间允许你在集群中创建一个独立区域。典型地，对象只能与同一命名空间下的对象进行通信，如果命名空间从你的规格中删除了，那么会使用“default”命名空间。
# spec
spec对象是yaml deployment规格文件的关键。它是定义kubernetes deployment行为的地方。为了kubernetes产生有意义的行为，很有必要描述spec对象，虽然spec对象不是kubernetesAPI所严格要求的。

spec对象有许多域。稍后将在本文中详述各个域。spec对象的所有域中只有spec.template域是必须定义的。
## spec.template
spec.template是一个必不可少的对象。它描述kubernetes创造的pod，pod是deployment的一部分。在kubernetes中运行的容器化应用是跑在pod中的，pod是一个逻辑单位，它可以共享存储与管理指令。pod被统一定位且统一调度，并且运行在共享的空间中。一个pod可以包含一个或多个容器化应用。
例如，我们可以运行如下命令来查询pod：
```
$ kubectl get pods
$ kubectl get pods {POD_NAME}
$ kubectl describe pods {POD_NAME}
```
### spec.template.spec
spec.template.spec是spec.template中的一个对象，它为Pod的行为做详细的描述，它包含许多域，接下来会在本文进行描述。在这些域中，只有spec.template.spec.containers域是必须的。
#### spec.template.spec.containers
spec.template.spec.containers是一个数组对象，它必须包含至少一个容器。在这个数组中的每一个containers描述pod中一个容器化应用的行为。一个pod中仅有一个容器是非常常见的，然而我们可以为pod指定任意数量的容器。
##### spec.template.spec.containers[i].name
spec.template.spec.containers[i].name是一个必要的字符串。在pod中的每一个容器都需要一个独一无二的名字。name作为字符串要遵从DNS_LABEL定义，不能超过63个字符的长度，这要遵守RFCs1035和1123中“label”的定义。命令要符合正则表达式[a-z0–9]([-a-z0–9]*[a-z0–9])。
在上例中，命名我们的容器为“nginx”。
##### spec.template.spec.containers[i].image
spec.template.spec.containers[i].image是一个可选字符串，它规定哪个镜像在已经部署的容器中运行。如此一来，当部署一个docker镜像为容器化应用时，我们需要包含这个域。

在上例中，我们部署了一个公开的nginx镜像，这个镜像在DockerHub中存储，标签为1.7.9。

我们也可以通过指定一个私有的docker仓库来部署私有镜像。如下是一个被修改过的spec，它指定一个存储在Distelli的私有仓库中的nginx镜像。
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
想应用这个改变到kubernetes 的deployment规格中是非常简单的，运行如下命令：
```
$ kubectl apply -f deployment.yml
```
正如我们所见，使用yaml配置文件的短短几行就可以与kubernetes有大量的交互。请继续关注本系列的下一个部分，探索更多高级的deployment配置，体会不同配置之间的差异。


  [1]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
  [2]: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/
