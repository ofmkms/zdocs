<!-- MarkdownTOC -->

- 1. kubernetes扩展机制
	- 1.1 k8s的层次结构
		- 1.1.1 第三方基础设施
		- 1.1.2 API和extension
		- 1.1.3 K8S核心层
		- 1.1.4 K8s接口层
		- 1.1.5 应用生态
- 2. k8s在API和extension层面的扩展性
	- 2.1 从哪里扩展 -- K8s API Server
	- 2.2 扩展什么 -- Kubernetes的操作对象和扩展对象
		- 2.2.1 Object
		- 2.2.2 Resource
		- 2.2.3 custom resource
		- 2.2.4 controller
		- 2.2.5 custom controller
	- 2.3. 如何扩展--扩展K8s API 方式
		- 2.3.1 Custom Resource Definition
			- 2.3.1.1 CRD实现原理
			- 2.3.1.2 访问方式
		- 2.3.2 AA \(API Aggregation\)
			- 2.3.2.1 AA实现原理：
			- 2.3.2.2 访问方式：
	- 2.4 扩展Patterns
- 3. 参考资料

<!-- /MarkdownTOC -->

# 1. kubernetes扩展机制

   在容器调度领域kubernetes异军突起，除了想google、redhat、ibm这样的大的商业公司支持支持外，自身的架构优势也是其成为这个领域的霸主的关键因素。
在众多特性中，高度可配置化和可扩展化是kubernetes的一大特色，下面将详细介绍k8s在扩展性方面设计特点。

## 1.1 k8s的层次结构
![k8s生态层次](https://i.imgur.com/SUIYZXT.png)

    总体架构（自下而上）：
	- 应用生态
	- k8s接口层
	- k8s核心层，包括资源对象，控制器，管控策略
	- API和extension
	- 第三方基础设施
### 1.1.1 第三方基础设施

	- 计算，container runtime interface（CRI）
	- 网络，network plugin （CNI）
	- 存储卷，volume api
	- 镜像， image api（OCI）
	- 第三云设施， cloud provider
	- 身份认证， rbac sso .etc.
### 1.1.2 API和extension

	- addon机制
	- api主要起到聚合作用，增加生态融合，包括CRI，CNI，CSI，OAuth，AA（API aggregate）
	- extens主要起到丰富自身作用，同时释放k8s核心的扩展能力，包括custom resource和custom controller，甚至custom scheduler等
### 1.1.3 K8S核心层

	- 自身组成分为master和slave，并且master和slave可以水平扩展
	- 确定了基于状态声明的松耦合架构，设计了controller，resource组件以及标准化了基于yaml的表述机制。
![申明式](https://i.imgur.com/zkT2UcV.png)
### 1.1.4 K8s接口层

	- 提供了rest api接口
	- 提供kubectl命令行接口
	- 提供rbac管控机制
### 1.1.5 应用生态

	- Docker
	- TensorFlow
	- Devops
	- Bigdata
	... ...

**本次聚焦在api和extension部分**


# 2. k8s在API和extension层面的扩展性
	Kubernetes在这一部分提供了3种扩展方式：
	- 1. Custom resources and controllers
	- 2. CRD (Custom Resource Definitions)
	- 3. AA（API server aggregation）(API聚合)

在2016年，CoreOS公司基于CRD概念推出了Operator模式。
![Operator模式](https://coreos.com/sites/default/files/inline-images/Overview-etcd_0.png)
	
	Operator模式建立在 Kubernetes 资源和控制器概念之上，在用户实现应用业务逻辑以外，用户通过自己编写的管理器，
	以编程的方式把“运维”逻辑写进了管理器的代码当中，从而实现了应用的运维代码化，而不是手动配置与应用生命周期管理相关的细节。
	从而降低了分布式系统的负责性。


## 2.1 从哪里扩展 -- K8s API Server

	首先说明k8s api（server）是什么，一句话，所谓的k8s的api扩展，就是增强原有apiserver的能力，
	● API-Server is a REST Application with CRUD interface
	● API-Server doesn’t know anything about infrastructure
	● API-Server manages Pods/Deployments/Services like apples and pears (no special meaning)
	● Object Properties:
		apiversion
		kind
		metadata
		spec
		status---不需要用户指定，K8s负责更新


提前说明：kube-aggregator 组件是在apiserver进行功能拆分的背景下产生的，这里面有apiserver功能复杂度越来远大的问题，同时k8s核心开发人员集中精力在核心功能的需求

![kube-aggregator](https://i.imgur.com/YssnzN6.jpg)
``` 
kube-aggregator provides
    Provide an API for registering API servers.
    Summarize discovery information from all the servers.
    Proxy client requests to individual servers.
https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/kube-aggregator
```


## 2.2 扩展什么 -- Kubernetes的操作对象和扩展对象
### 2.2.1 Object
record of intent(意愿记录)，用户创建一个k8s object，相当于客户要求k8s系统完成一个任务，k8s系统就创建object，并且保持object状态是用户预期状态。

	用户通过kubenetes api创建使用kubectl命令行工具，除此之外也可以使用client libraries.
	K8s object 描述一般使用yaml文件，包括apiversion，kind，metadata，spec，status.

### 2.2.2 Resource
A resource is an endpoint in the Kubernetes API that stores a collection of API objects of a certain kind. For example, the built-in pods resource contains a collection of Pod objects.

	说白了，就是k8s object通过k8s api展现的形式就是resource.常见的k8s自带resource类型包括：

``` shell
root@u-s1:~/workspace/crd# kubectl get
You must specify the type of resource to get. Valid resource types include: 

  * all  
  * certificatesigningrequests (aka 'csr')  
  * clusterrolebindings  
  * clusterroles  
  * componentstatuses (aka 'cs')  
  * configmaps (aka 'cm')  
  * controllerrevisions  
  * cronjobs  
  * customresourcedefinition (aka 'crd')  
  * daemonsets (aka 'ds')  
  * deployments (aka 'deploy')  
  * endpoints (aka 'ep')  
  * events (aka 'ev')  
  * horizontalpodautoscalers (aka 'hpa')  
  * ingresses (aka 'ing')  
  * jobs  
  * limitranges (aka 'limits')  
  * namespaces (aka 'ns')  
  * networkpolicies (aka 'netpol')  
  * nodes (aka 'no')  
  * persistentvolumeclaims (aka 'pvc')  
  * persistentvolumes (aka 'pv')  
  * poddisruptionbudgets (aka 'pdb')  
  * podpreset  
  * pods (aka 'po')  
  * podsecuritypolicies (aka 'psp')  
  * podtemplates  
  * replicasets (aka 'rs')  
  * replicationcontrollers (aka 'rc')  
  * resourcequotas (aka 'quota')  
  * rolebindings  
  * roles  
  * secrets  
  * serviceaccounts (aka 'sa')  
  * services (aka 'svc')  
  * statefulsets (aka 'sts')  
  * storageclasses (aka 'sc')error: Required resource not specified.
Use "kubectl explain <resource>" for a detailed description of that resource (e.g. kubectl explain pods).
See 'kubectl get -h' for help and examples.
root@u-s1:~/workspace/crd#

```
REST请求方式：
![k8s resource url](https://i.imgur.com/SVFC9Xy.jpg)

### 2.2.3 custom resource

首先custom resource是k8s api的扩展。
	A custom resource is an extension of the Kubernetes API that is not necessarily available on every Kubernetes cluster. In other words, it represents a customization of a particular Kubernetes installation.
需要说明的是custom resource可以通过kubectl进行生命周期管理，工作模式和pods类似。

Cusom resource就是k8s本身不提供，用户自己创建的resource
创建custom resource的过程是扩展k8s api的过程，即是本文的描述重点。

### 2.2.4 controller
控制器负责解析用户预期resource状态记录，通过不断调整resource，进而达到用户预期状态。

	- Reconciliation loops: Observe + Analyze + Act
	- Converge desidered state with real state
	- Attached to infrastructure events (getters & listers & informers)
	- Simple Interface to implement: ADD, UPDATE, DELETE
	- Cluster aware
	- Actions can be internal to the cluster or external (beginning of service catalog)

### 2.2.5 custom controller
是用户可以在集群部署和更新的controller，custom controller原则上可以和任何resource搭配工作，
但是Custom controller一般情况下是和custom resource成对出现，即k8s通过custom controller实现对custom resource的控制，进而确定custom resource的预期状态。

其中，Operator模式就是custom controller和custom resource的一种组合方式。


创建custom controller是一个复杂过程，也是扩展k8s api的过程，也是本文的描述重点。

## 2.3. 如何扩展--扩展K8s API 方式

	- CRD(Custom Resource Definitions)
	- AA (API Aggregation)

其中CRD相对AA来说不需要编程序，但是AA自由度更高，要求也更高。


### 2.3.1 Custom Resource Definition
	Previously known as TPR（Third Party Resources） ( 1.2 -- 1.7 )
	Stable from kubernetes 1.8
	C-R-U-D Object
	RBAC
	State definition API: declarative
	Versioning
	Automatic `kubectl` compatibility

#### 2.3.1.1 CRD实现原理

	1．创建CRD，类型是Foo
``` shell
kubectl create -f artifacts/examples/crd.yaml
root@u-s1:~/workspace/crd/sample-controller# cat artifacts/examples/crd.yaml 
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: foos.samplecontroller.k8s.io
spec:
  group: samplecontroller.k8s.io
  version: v1alpha1
  names:
    kind: Foo
    plural: foos
  scope: Namespaced
``` 
	2．创建custom resource，类型是Foo
``` shell
kubectl create -f artifacts/examples/example-foo.yaml
root@u-s1:~/workspace/crd/sample-controller# cat artifacts/examples/example-foo.yaml 
apiVersion: samplecontroller.k8s.io/v1alpha1
kind: Foo
metadata:
  name: example-foo
spec:
  deploymentName: example-foo
  replicas: 1
root@u-s1:~/workspace/crd/sample-controller#

```
	3．查询custom resource
``` shell
root@u-s1:~/workspace/crd/sample-controller# kg Foo
NAME          AGE
example-foo   10m
root@u-s1:~/workspace/crd/sample-controller# kg Foo/example-foo
NAME          AGE
example-foo   11m
root@u-s1:~/workspace/crd/sample-controller# kd Foo
Name:         example-foo
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  samplecontroller.k8s.io/v1alpha1
Kind:         Foo
Metadata:
  Cluster Name:        
  Creation Timestamp:  2018-05-14T13:51:31Z
  Generation:          1
  Resource Version:    147372
  Self Link:           /apis/samplecontroller.k8s.io/v1alpha1/namespaces/default/foos/example-foo
  UID:                 e82e32e0-577d-11e8-afe6-000c2986e8bd
Spec:
  Deployment Name:  example-foo
  Replicas:         1
Events:             <none>

```
#### 2.3.1.2 访问方式

``` shell
root@u-s1:~/workspace/crd/sample-controller# curl -k https://192.168.178.137:6443/apis/samplecontroller.k8s.io/v1alpha1/namespaces/default/foos/example-foo 
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "foos.samplecontroller.k8s.io \"example-foo\" is forbidden: User \"system:anonymous\" cannot get foos.samplecontroller.k8s.io in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "name": "example-foo",
    "group": "samplecontroller.k8s.io",
    "kind": "foos"
  },
  "code": 403
}root@u-s1:~/workspace/crd/sample-controller#

```

### 2.3.2 AA (API Aggregation)

理解
kube-aggregator 
kube-apiserver

aggregation layer
APIService
extension-apiserver
	
说明

kube-apiserver是k8s的关键组成模块，是k8s系统的控制接口。

kube-aggregator k8s模块，负责聚合k8s apiserver，实现api server的动态注册，动态发现以及安全代理。
该模式实现将当天api server进行模块化，一方面提供了普通用户实现自己api server的机制，同时实现了在k8s集群运行时动态增加api的能力，实现这个模块的动因是：
1.	扩展性，允许社区用户实现自己的k8s API
2.	解放核心开发团队review代码压力
3.	给实验性api一个运行的机制
4.	规范社区api开发规范，兼容k8s习惯。


extension-apiserver是k8s api server的扩展功能模块

aggregation layer是k8s api server功能扩展层。
根据实现技术这里存在两种实现api扩展机制，
- 一种是service-catalog，负责聚合已有的第三方方案，
- 第二种是用户自己开发的模块，k8s提供了一个apiserver-builder项目，进而降低用户开发的难度。
aggregation layer运行在kube-apiserver模块中。kube-apiserver负责将API path代理到APIService。

APIService是通过extension-apiserver以集群上pod形式提供的。

总结上面的话，
extension api server请求流程是：
1. 用户通过apiserver访问集群，一般访问方式就是API path。
2. 在apiserver中的aggregation layer模块根据API path将请求代理到APIService。
3. APIService实现k8s resource和k8s controller业务逻辑。
	
通过API聚合方式扩展k8s，涉及的主要组件包括：
aggregation layer和 APIService，
其中
APIservice提供 api service实现对象和URL path定义，
aggregation layer实现url到object的代理。其中APIService通过extension-apiserver形式部署在k8s集群中实现。extension-apiserver一般情况下包括 Controller和resource两部分，这也是k8s的一般做法。

前提条件，增加api server配置参数：
``` shell
kube-apiserver flags：
--enable-aggregator-routing=true


root@u-s1:/etc/kubernetes/manifests# ls kube-apiserver.yaml 
/etc/kubernetes/manifests/kube-apiserver.yaml
    配置需要增加
    - --enable-aggregator-routing=true
#    - --runtime-config=api/all=true,admissionregistration.k8s.io/v1alpha1=true
   
```

#### 2.3.2.1 AA实现原理：

	** 实现方式：
	- 用户基于api开发
		k8s社区提供了应用框架加速开发
		- 应用框架项目： api-builder  https://github.com/kubernetes-incubator/apiserver-builder
		- 应用实例项目：	https://github.com/kubernetes/sample-apiserver
	- service-catalog方式
		同open source service broker实现，参照https://github.com/openservicebrokerapi/servicebroker
		** 借助第三方力量快速建设云平台的利器

	** 用户通过api实现扩展开发的流程：

	1. 创建一个namespace
	2. 创建一个serviceaccount
	3. serviceaccount绑定 clusterrole  system:auth-delegator权限
	4. serviceaccount绑定 role extension-apiserver-authentication-reader
	5. 创建apiserver， 通过deployment或者rc实现
	6. 通过service暴露apiserver的访问地址
	7. 创建APIService，本例中含有两个type flunder（比目鱼）和fischer(惠鱼)
	8. 创建fischer资源
		kubectl create -f artifacts/fischer/01-fischer.yaml	
		kg fischer
    
    ** 演练

设置golang语言环境和代码环境
```
export GOROOT=/usr/lib/go  #设置为go安装的路径
export GOPATH=$HOME/workspace/gocode
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

mkdir -p k8s.io
cd k8s.io
git clone https://github.com/kubernetes/sample-apiserver.git
git checkout -b release-1.11
```

生成APIService部署用的docker镜像。
```
docker build -t 192.168.178.167/base/kube-sample-apiserver:latest ./artifacts/simple-image
docker push 192.168.178.167/base/kube-sample-apiserver:latest
```
部署minikube集群，以及在minkube集群上部署通过AA方式开发的k8s扩展
```
https://github.com/kubernetes/sample-apiserver/blob/master/docs/minikube-walkthrough.md
```
创建脚本：
```
kubectl create ns wardle
kubectl create -f artifacts/example/ns.yaml
kubectl create -f artifacts/example/sa.yaml -n wardle
kubectl create -f artifacts/example/auth-delegator.yaml -n kube-system
kubectl create -f artifacts/example/auth-reader.yaml -n kube-system
kubectl create -f artifacts/example/rc.yaml -n wardle
kubectl create -f artifacts/example/service.yaml -n wardle
kubectl create -f artifacts/example/apiservice.yaml
```

删除脚本：
```
kubectl delete -f artifacts/flunders/01-flunder.yaml
kubectl delete -f artifacts/example/apiservice.yaml
kubectl delete -f artifacts/example/service.yaml -n wardle
kubectl delete -f artifacts/example/rc.yaml -n wardle
kubectl delete -f artifacts/example/auth-reader.yaml -n kube-system
kubectl delete -f artifacts/example/auth-delegator.yaml -n kube-system
kubectl delete -f artifacts/example/sa.yaml -n wardle
kubectl delete -f artifacts/example/ns.yaml
kubectl delete ns wardle
```

结果确认：
```
kg flunder -n wardle -o wide
kg Fischer -n wardle -o wide
```

```
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kubectl create -f artifacts/fischer/01-fischer.yaml -n wardle
fischer.wardle.k8s.io "my-first-fischer" created
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kg fischer
NAME               AGE
my-first-fischer   6s
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kd fischer
Name:         my-first-fischer
Namespace:    
Labels:       sample-label=true
Annotations:  <none>
API Version:  wardle.k8s.io/v1alpha1
Kind:         Fischer
Metadata:
  Creation Timestamp:  2018-05-18T03:19:10Z
  Resource Version:    11
  Self Link:           /apis/wardle.k8s.io/v1alpha1/fischers/my-first-fischer
  UID:                 3b40bea3-5a4a-11e8-8daa-0a580af4016a
Events:                <none>
```

```
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kubectl create -f artifacts/flunders/01-flunder.yaml -n wardle
flunder.wardle.k8s.io "my-first-flunder" created
```

```
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kd flunder
Name:         my-first-flunder
Namespace:    default
Labels:       sample-label=true
Annotations:  <none>
API Version:  wardle.k8s.io/v1alpha1
Kind:         Flunder
Metadata:
  Creation Timestamp:  2018-05-18T02:58:26Z
  Resource Version:    4
  Self Link:           /apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder
  UID:                 55dbf6cb-5a47-11e8-8daa-0a580af4016a
Spec:
Status:
Events:  <none>
```

```
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# kd flunder
Name:         my-first-flunder
Namespace:    default
Labels:       sample-label=true
Annotations:  <none>
API Version:  wardle.k8s.io/v1alpha1
Kind:         Flunder
Metadata:
  Creation Timestamp:  2018-05-18T02:58:26Z
  Resource Version:    4
  Self Link:           /apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder
  UID:                 55dbf6cb-5a47-11e8-8daa-0a580af4016a
Spec:
Status:
Events:  <none>
root@u-s1:~/workspace/gocode/src/k8s.io/sample-apiserver# 

```

其他：
```
kubectl create clusterrolebinding wardle-apiserver-admin --clusterrole=cluster-admin --serviceaccount=wardle:apiserver -n wardle

kubectl delete clusterrolebinding wardle-apiserver-admin -n wardle

```

#### 2.3.2.2 访问方式：

	root@u-s1:~/workspace# curl -k https://192.168.178.137:6443/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder
	{
	  "kind": "Status",
	  "apiVersion": "v1",
	  "metadata": {
	    
	  },
	  "status": "Failure",
	  "message": "flunders.wardle.k8s.io \"my-first-flunder\" is forbidden: User \"system:anonymous\" cannot get flunders.wardle.k8s.io in the namespace \"default\"",
	  "reason": "Forbidden",
	  "details": {
	    "name": "my-first-flunder",
	    "group": "wardle.k8s.io",
	    "kind": "flunders"
	  },
	  "code": 403
	}
	root@u-s1:~/workspace# 


## 2.4 扩展Patterns

- CRD + Volume Plugin + Controller ⇒ Rook
- CRD + Network Plugin ⇒ Calico Canal
- CRD + Controller ⇒ Operator
- CRD + Controller ⇒ core features prototyping

# 3. 参考资料

- Operator模式问世：
	https://coreos.com/blog/introducing-operators.html
- TPR to CRD
	https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-third-party-resource/
	https://kubernetes.io/docs/tasks/access-kubernetes-api/migrate-third-party-resource/
- customer resource
	https://kubernetes.io/docs/concepts/api-extension/custom-resources/
- 通过CRD扩展k8s api
	https://kubernetes.io/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions
- 通过AA扩展k8s api
	https://kubernetes.io/docs/concepts/api-extension/apiserver-aggregation/
- Operator Framework
	https://github.com/operator-framework/getting-started
- AA示例
	https://github.com/kubernetes/sample-controller

