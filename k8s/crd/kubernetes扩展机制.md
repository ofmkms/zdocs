#kubernetes扩展机制

在容器调度领域kubernetes异军突起，除了想google、redhat、ibm这样的大的商业公司支持支持外，自身的架构优势也是其成为这个领域的霸主的关键因素。
在众多特性中，高度可配置化和可扩展化是kubernetes的一大特色，下面将详细介绍k8s在扩展性方面设计特点。

#k8s的层次结构
![k8s生态层次](https://i.imgur.com/SUIYZXT.png)

## 总体架构（自下而上）：
	应用生态
	k8s接口层
	k8s核心层，包括资源对象，控制器，管控策略
	API和extension
	第三方基础设施
## 第三方基础设施
	计算，container runtime interface（CRI）
	网络，network plugin （CNI）
	存储卷，volume api
	镜像， image api（OCI）
	第三云设施， cloud provider
	身份认证， rbac sso .etc.
## API和extension
	addon机制
	api主要起到聚合作用，增加生态融合，包括CRI，CNI，CSI，OAuth，AA（API aggregate）
	extens主要起到丰富自身作用，同时释放k8s核心的扩展能力，包括custom resource和custom controller，甚至custom scheduler等
##K8S核心层

	自身组成分为master和slave，并且master和slave可以水平扩展
	确定了基于状态声明的松耦合架构，设计了controller，resource组件以及标准化了基于yaml的表述机制。
##K8s接口层
	提供了rest api接口
	提供kubectl命令行接口
	提供rbac管控机制
##应用生态
	Docker
	TensorFlow
	Devops
	Bigdata
	... ...

**本次聚焦在api和extension部分

##k8s

k8s自身的扩展性
k8s


