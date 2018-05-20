# 关于k8s的几个port的总结：

	环境说明本例子使用minikube进行说明
	minikube的vm drvier是kvm2，环境信息是：
	宿主机： 192.168.178.143
	minikube vm: 192.168.39.62
----------------------------------------------


以kubernetes dashboard为例说明：

``` shell
root@minikube:~/workspace# kgaa
NAMESPACE     NAME                                        READY     STATUS    RESTARTS   AGE
default       pod/hello-minikube-6c47c66d8-bwljq          1/1       Running   0          48m
kube-system   pod/etcd-minikube                           1/1       Running   0          1h
kube-system   pod/kube-addon-manager-minikube             1/1       Running   0          1h
kube-system   pod/kube-apiserver-minikube                 1/1       Running   7          1h
kube-system   pod/kube-controller-manager-minikube        1/1       Running   0          1h
kube-system   pod/kube-dns-86f4d74b45-fs9ph               3/3       Running   0          1h
kube-system   pod/kube-proxy-gwbj5                        1/1       Running   0          1h
kube-system   pod/kube-scheduler-minikube                 1/1       Running   0          1h
kube-system   pod/kubernetes-dashboard-5498ccf677-26w4q   1/1       Running   0          1h
kube-system   pod/storage-provisioner                     1/1       Running   0          1h

NAMESPACE     NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
default       service/hello-minikube         NodePort    10.99.31.210     <none>        8080:30543/TCP   47m
default       service/kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP          1h
kube-system   service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP    1h
kube-system   service/kubernetes-dashboard   NodePort    10.101.236.234   <none>        80:30000/TCP     1h

NAMESPACE     NAME                        DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/kube-proxy   1         1         1         1            1           <none>          1h

NAMESPACE     NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/hello-minikube         1         1         1            1           48m
kube-system   deployment.apps/kube-dns               1         1         1            1           1h
kube-system   deployment.apps/kubernetes-dashboard   1         1         1            1           1h

NAMESPACE     NAME                                              DESIRED   CURRENT   READY     AGE
default       replicaset.apps/hello-minikube-6c47c66d8          1         1         1         48m
kube-system   replicaset.apps/kube-dns-86f4d74b45               1         1         1         1h
kube-system   replicaset.apps/kubernetes-dashboard-5498ccf677   1         1         1         1h
root@minikube:~/workspace# 


```
其中：
``` shell
root@minikube:~/workspace# minikube ip
192.168.39.62

```
pod定义
``` yaml
root@minikube:~/workspace# kg -n  kube-system  pod/kubernetes-dashboard-5498ccf677-26w4q -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2018-05-20T12:39:31Z
  generateName: kubernetes-dashboard-5498ccf677-
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    app: kubernetes-dashboard
    pod-template-hash: "1054779233"
    version: v1.8.1
  name: kubernetes-dashboard-5498ccf677-26w4q
  namespace: kube-system
  ownerReferences:
  - apiVersion: extensions/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: kubernetes-dashboard-5498ccf677
    uid: d8207135-5c2a-11e8-b721-20af3fc84158
  resourceVersion: "451"
  selfLink: /api/v1/namespaces/kube-system/pods/kubernetes-dashboard-5498ccf677-26w4q
  uid: d82664b2-5c2a-11e8-b721-20af3fc84158
spec:
  containers:
  - image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 9090
        scheme: HTTP
      initialDelaySeconds: 30
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 30
    name: kubernetes-dashboard
    ports:
    - containerPort: 9090
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-ngqtg
      readOnly: true
  dnsPolicy: ClusterFirst
  nodeName: minikube
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-ngqtg
    secret:
      defaultMode: 420
      secretName: default-token-ngqtg
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: 2018-05-20T12:39:31Z
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: 2018-05-20T12:39:33Z
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: 2018-05-20T12:39:31Z
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://480cd0e17122770d2359d30753123e0ddaa5b1eb3b08dd0cea73d0e459c44c42
    image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.1
    imageID: docker://sha256:e94d2f21bc0c297cb74c1dfdd23e2eace013f532c60726601af67984d97f718a
    lastState: {}
    name: kubernetes-dashboard
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: 2018-05-20T12:39:32Z
  hostIP: 192.168.122.62
  phase: Running
  podIP: 172.17.0.3
  qosClass: BestEffort
  startTime: 2018-05-20T12:39:31Z
root@minikube:~/workspace# 

```
service定义

``` yaml
root@minikube:~/workspace# kg -n  kube-system  service/kubernetes-dashboard -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"addonmanager.kubernetes.io/mode":"Reconcile","app":"kubernetes-dashboard","kubernetes.io/minikube-addons":"dashboard","kubernetes.io/minikube-addons-endpoint":"dashboard"},"name":"kubernetes-dashboard","namespace":"kube-system"},"spec":{"ports":[{"nodePort":30000,"port":80,"targetPort":9090}],"selector":{"app":"kubernetes-dashboard"},"type":"NodePort"}}
  creationTimestamp: 2018-05-20T12:39:31Z
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    app: kubernetes-dashboard
    kubernetes.io/minikube-addons: dashboard
    kubernetes.io/minikube-addons-endpoint: dashboard
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "432"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard
  uid: d827e2d3-5c2a-11e8-b721-20af3fc84158
spec:
  clusterIP: 10.101.236.234
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30000
    port: 80
    protocol: TCP
    targetPort: 9090
  selector:
    app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

```

结论：

在宿主机上执行：
curl -k http://192.168.39.62   成功

在minikube（192.168.39.62）执行：
curl -k http://172.17.0.3:9090
curl -k http://10.101.236.234:80


** 说明：**

	容器的IP即POD的IP是172.17.0.3端口是9090
	其中172.17.0.3的地址是容器地址或者pod地址。

	服务地址或者cluster地址是10.101.236.234
	/etc/kubernetes/manifests/kube-apiserver.yaml 
	--service-cluster-ip-range=10.96.0.0/12
	96（十进制）=01100000（二进制）
	101（十进制）=01100101（二进制


	综上说明：
		1. k8s dashboard 容器监听在172.17.0.3:9090
		2. k8s创建service，并且通过label selector和pod绑定，生命服务的port是80，目标pod port是9090,服务部署以后，k8s分配service ip是/10.101.236.234
			port: 80
		    protocol: TCP
		    targetPort: 9090
		3. 由于需要从主机访问，所以dashboard的service port通过node port的方式暴露，nodeport是30000
			ports:
			  - nodePort: 30000
			  ... ...
			type: NodePort

	类似的例子， service/hello-minikube访问方式是：
```
root@minikube:~/workspace# curl -k http://192.168.39.62:30543
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.39.62:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=192.168.39.62:30543
user-agent=curl/7.47.0
BODY:
-no body in request-
root@minikube:~/workspace# 
````
