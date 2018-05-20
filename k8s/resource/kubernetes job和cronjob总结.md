# 1. Jobs 和cronjob介绍

# 1.1. 基本概念

	Jobs和cronjob都是kubernetes Controller的一种，
	其中jobs和cronjob和其他controller的区别是一次运行，运行完成即销毁。
	cronjob是一种特殊的jobs是在特定时间运行，根据预先设置的规则可以一次运行也可以周期运行。

# 2. Jobs任务
## 2.1. 概念（run to compltion）：

	A job creates one or more pods and ensures that a specified number of them successfully terminate. 
	As pods successfully complete, the job tracks the successful completions.  
	When a specified number of successful completions is reached, the job itself is complete.  
	Deleting a Job will cleanup the pods it created.

** 这里强调一下事情：

- 创建一个和多个pod，并且保障一定量的pod成功终止。
- Job跟踪pod的运行结果
- 如果指定数量的pod运行成功，则job结束，状态为complete
- 删除job就会删除job运行时产生的pod，如果使用—cascade=false,则不删除pod.

## 2.2. Job Spec编写规范

	一个job必须包括：apiVersion、kind、metadata和spec.
	其中：
- 	 apiVersion值为：batch/v1
- 	 kind值为：Job
- 	 metadata值包括 name，labels，namespace等
- 	 spec值是job规范定义主体部分，是至关重要的。
	
	其中job专属属性是：
		activeDeadlineSeconds
				最长job存在时间
		backoffLimit			默认值是6
				重试（退避）次数，
		completions
				期望执行测试
		manualSelector
				制定jobs和pod的对应关系。
		parallelism
				并行执行job数量，如果parallelism是0，则Job无限等待，可用通过
				kubectl scale  --replicas=$N jobs/zsy-job-helloworld启动
		Controller通用属性：
		selector
			pod调度用的labelselector，可选部分
		template，必须部分
			pod规范，job的业务逻辑。
			需要定义RestartPolicy，数值可选Never和OnFailure

## 2.4. 使用
## 2.5. 语法
-		创建语法
```
kubectl apply -f zsy-jobs-helloworld.yaml
```
-		删除语法
```
kubectl delete -f zsy-jobs-helloworld.yaml
```
-		查看
```
kubectl get jobs
Kubectl describe jobs
```
## 2.6 分类
- 非并行Jobs
	不指定completions和parallelism，他们的默认值是1
- 指定完成数量的Jobs，其中并发是1
	指定completions但是不指定parallelism，其中parallelism是1
- 指定工作队列的Jobs
	指定parallelism，但是不指定completions

## 2.7 特殊用法
- 指定pod selector，实现job定义修改。参照例子：（zsy-jobs.yaml  zsy-jobs-new.yaml）
- 关于Pod和Container失败
- 关于backoffLimit和restartPolicy以及completions的关系

## 2.8. 实现
```	
kubernetes/pkg/controller/job/
kubernetes/pkg/controller/cronjob/
```

## 2.9. 高级
	同时使用job template实现多job管理。参照Multiple Job Objects from Template Expansion
	https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/

# 3. cronJob计划任务

## 3.1.	概念
-	首先解释cron（计划）
	cron是操作系统实现基于时间的job调度工具，job执行计划通过crontab配置，crontab格式如下：

``` shell
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                       7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
#  *  *  *  *  *  command to execute
```
-	举例:
``` shell
schedule: "*/1 * * * *"
```

**Cronjob的限制	
	不能完全保证执行一次，因此需要jobs幂等

## 3.2. cronjob spec编写规范
	一个cronjob必须包括：apiVersion、kind、metadata和spec.
	其中：
-	apiVersion值为：batch/v2alpha1
-	kind值为：cronjob
-	metadata值包括 name，labels，namespace等
-	spec值是cronjob规范定义主体部分，是至关重要的。

	其中cronjob专属属性是：
-		concurrencyPolicy
			值包括：Allow,Forbid,Replace
			其中：
			Allow是上一次运行没完，本次运行照常运行
			Forbid是上一次运行没完，忽略本次运行
			Replace是上一次运行没完，停止上一次，运行本次cronjob.
-		schedule
			必选项，格式参照：https://en.wikipedia.org/wiki/Cron.
-		startingDeadlineSeconds
			缓期启动时间，如果在缓期启动时间之后依然没有启动，就标记为failed
-		successfulJobsHistoryLimit
-		failedJobsHistoryLimit
			成功和失败job数，一般默认值分别是3和1
-		suspend
			默认是false，如果设置为true，则后续执行job是suspended状态。
		
## 3.3.	使用
## 3.3.1.	语法

- 启动

``` shell
$ kubectl run hello --schedule="*/1 * * * *" \
	--restart=OnFailure \
	--image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"


cronjob "hello" created

or
$ kubectl create -f ./cronjob.yaml
cronjob "hello" created
``` 

- 查询
``` shell
root@u-s1:~/workspace/jobs# kg cronjobs hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
hello     */1 * * * *   False     0         37s             5m
root@u-s1:~/workspace/jobs#

root@u-s1:~/workspace/jobs# kg jobs --watch
NAME               DESIRED   SUCCESSFUL   AGE
hello-1526210940   1         1            3m
hello-1526211000   1         1            2m
hello-1526211060   1         1            1m
hello-1526211120   1         0            4s
pi-with-timeout    <none>    5            1h
hello-1526211120   1         1         12s
hello-1526210940   1         1         3m
hello-1526210940   1         1         3m
hello-1526210940   1         1         3m
``` 
- 删除
``` shell
$ kubectl delete cronjob hello
cronjob "hello" deleted
```

## 3.4. 实现
```
kubernetes/pkg/controller/job/
kubernetes/pkg/controller/cronjob/
```
# 4. 应用场景
	不适合紧密通讯的并且计算，比如科学计算，适合彼此独立的批量任务，比如图像处理，大数据处理，邮件发送，文件转码等
# 5. 测试
## 5.1. 试验一:  helloworld

- 创建helloworld yaml文件
```
#root@u-s1:~/workspace/jobs# cat zsy-jobs-helloworld.yaml 
apiVersion: batch/v1
kind: Job
metadata:
 labels:
   app: job
   project: zsy
   version: v1
 name: zsy-job-helloworld
 namespace: default
spec:
 template:
   metadata:
     labels:
       app: job
       job-name: zsy-job-helloworld
       project: zsy
       version: v1
     name: zsy-job-helloworld
   spec:
     containers:
     - command: ['echo','helloworld']
       image: busybox
       name: zsy-job-helloworld
     restartPolicy: Never
```
- 部署

```
root@u-s1:~/workspace/jobs# kubectl apply -f zsy-jobs-helloworld.yaml 
job.batch "zsy-job-helloworld" created
```

- 查看信息

	创建之初：
```
root@u-s1:~/workspace/jobs# kubectl get job
NAME                 DESIRED   SUCCESSFUL   AGE
zsy-job-helloworld   1         0            8s
```
	pod确认：
```
root@u-s1:~/workspace/jobs# kubectl get pod
NAME                         READY     STATUS      RESTARTS   AGE
zsy-job-helloworld-lr9ww     0/1       Completed   0          19s
```

	执行完毕：
```
root@u-s1:~/workspace/jobs# kubectl get job
NAME                 DESIRED   SUCCESSFUL   AGE
zsy-job-helloworld   1         1            16s
```

	确认结果
```
root@u-s1:~/workspace/jobs# kl zsy-job-helloworld-lr9ww
helloworld
```
	删除
```
root@u-s1:~/workspace/jobs# kubectl delete -f zsy-jobs-helloworld.yaml 
job.batch "zsy-job-helloworld" deleted

root@u-s1:~/workspace/jobs# kubectl get job
No resources found.

root@u-s1:~/workspace/jobs# kubectl get pod
NAME                         READY     STATUS    RESTARTS   AGE
httpd-app-77c9c8f99f-bfsrv   1/1       Running   0          6d
nginx-app-786897f7d7-9rjdd   1/1       Running   0          6d

```

## 5.2. 试验二：jobs和pods选择模式（Automatic Mode /Manual Mode）

```
root@u-s1:~/workspace/jobs# cat zsy-jobs.yaml 
apiVersion: batch/v1
kind: Job
metadata:
 labels:
   app: job
   project: zsy
   version: v1
 name: zsy-job
 namespace: default
spec:
 completions: 10
 parallelism: 2
# activeDeadlineSeconds: 60
 template:
   metadata:
     labels:
       app: job
       job-name: zsy-job
       project: zsy
       version: v1
     name: zsy-job
   spec:
     containers:
     - command: ['sleep','10']
       image: nginx
       name: zsy-job
     restartPolicy: Never

```

``` shell
root@u-s1:~/workspace/jobs# cat zsy-jobs-new.yaml 
apiVersion: batch/v1
kind: Job
metadata:
 name: zsy-job-new
 namespace: default
spec:
 manualSelector: true
 completions: 10
 parallelism: 2
# activeDeadlineSeconds: 60
 selector:
   matchLabels:
     controller-uid: 3316e18c-566a-11e8-9941-000c2986e8bd   #这是zsy-job的job uid。
 template:
   metadata:
     name: zsy-job-new
     labels:
       controller-uid: 3316e18c-566a-11e8-9941-000c2986e8bd
   spec: 
     containers:
     - command: ['sleep','10']
       image: nginx
       name: zsy-job
     restartPolicy: Never

root@u-s1:~/workspace/jobs#
```

# 6. 参考：
- Jobs api
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#job-v1-batch
- cronjob api
https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#cronjob-v2alpha1-batch
- jobs定义
https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
- cronjob定义
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

