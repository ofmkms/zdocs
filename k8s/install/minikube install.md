# minikube install

# driver为none

## 启动脚本

``` shell
#!/bin/bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl

export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
mkdir $HOME/.kube || true
touch $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/config
sudo -E ./minikube start --vm-driver=none

# this for loop waits until kubectl can access the api server that Minikube has created
for i in {1..150}; do # timeout for 5 minutes
   ./kubectl get po &> /dev/null
   if [ $? -ne 1 ]; then
      break
  fi
  sleep 2
done

# kubectl commands are now able to interact with Minikube cluster

```

其中minikube和kubectl版本是：

``` shell
root@minikube:~/workspace# ./minikube version
minikube version: v0.27.0

root@minikube:~/workspace# ./kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.2", GitCommit:"81753b10df112992bf51bbc2c2f85208aad78335", GitTreeState:"clean", BuildDate:"2018-04-27T09:22:21Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server 192.168.178.143:8443 was refused - did you specify the right host or port?
root@minikube:~/workspace# 

```
	
# driver为kvm2/kvm

## 安装依赖

```
# Install libvirt and qemu-kvm on your system, e.g.
# Debian/Ubuntu (for Debian Stretch libvirt-bin it's been replaced with libvirt-clients and libvirt-daemon-system)
sudo apt install libvirt-bin qemu-kvm


# Add yourself to the libvirtd group (use libvirt group for rpm based distros) so you don't need to sudo
# Debian/Ubuntu (NOTE: For Ubuntu 17.04 change the group to `libvirt`)
sudo usermod -a -G libvirtd $(whoami)


# Update your current session for the group change to take effect
# Debian/Ubuntu (NOTE: For Ubuntu 17.04 change the group to `libvirt`)
newgrp libvirtd

```
### 下载Docker Machine plugin binary
kvm2
``` shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 && chmod +x docker-machine-driver-kvm2 && sudo mv docker-machine-driver-kvm2 /usr/local/bin/
```

kvm
``` shell
curl -LO https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-ubuntu16.04 && chmod +x docker-machine-driver-kvm-ubuntu16.04 && sudo mv docker-machine-driver-kvm-ubuntu16.04 /usr/local/bin/docker-machine-driver-kvm

```
### 启动集群

``` shell
minikube start --vm-driver=kvm2
```
or
``` shell
minikube start --vm-driver=kvm
```
### 停止集群
``` shell
minikube stop
```
### 删除集群
``` shell
minikube delete
```

# 其他

- 说明minikube vm (boot2docker)的默认用户密码是
``` txt
user: docker
pass: tcuser
```
- 登录minikube vm的方式：
```
minikube ssh
```
```
minikube ip
ssh docker@ip
例如
ssh -i ./id_rsa docker@192.168.39.158
其中id_rsa文件在：~/.minikube/machines/minikube
```
# 问题
- 在使用kvm vm-driver的时候发现问题
(minikube) Failed to create the VM: virError(Code=9, Domain=20, Message='operation failed: domain 'minikube' already exists with uuid 33168c71-2c21-42d7-b8ab-6cf36d059de2')
使用-p参数制定新的profile，原因是kvm2启动的时候kvm profile （minikube）已经存在。
```
minikube start --vm-driver kvm --v 9 -p minikube-kvm
```
- 关于libmachine，kvm vm（boot2docker/minikube.iso），localkube等关系

```
macOS / Windows/ linux(kvm/kvm2)

minikube -> libmachine -> virtualbox/hyper V -> linux VM -> localkube
Linux

minikube -> docker -> localkube

Alternatives considered
```


# 参考文档：
https://github.com/kubernetes/minikube/blob/v0.27.0/README.md



