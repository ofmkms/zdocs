# VMWare虚拟机安装
略

**设置：允许虚拟机运行kvm虚拟机，支持vmdriver=kvm2
	
	设置--》处理器--》虚拟化引擎--》勾选 “虚拟化Intel VT-x/EPT或AMD-V/RVI(V)”

# 修改主机名
修改hostname和hosts文件
``` shell
root@minikube:~# cat /etc/hostname 
minikube
root@minikube:~# cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	minikube

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
192.168.178.143 minikube

root@minikube:~# 

```
# 安装必要软件
安装最基本的软件
``` shell
apt-get remove -y vim-common

apt-get install -y openssh-server
apt-get install -y vim
apt-get install -y curl wget
apt-get install -y ifstat
apt-get install -y ifstop


```
# 配置root可以ssh登录
默认情况下ubuntu16.0.4.3不允许 root ssh login

- 修改root密码
```
sudo passwd root 
```
- 修改ssh的配置文件
```
sudo vi /etc/ssh/sshd_config  
#设置PermitRootLogin yes
# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin yes  
StrictModes yes
```
- 重启ssh
```
sudo service ssh restart   
```

# 禁止swap
注释掉swap，避免以后每次使用swapoff –a，文件：/etc/fstab
```
# swap was on /dev/sda5 during installation
#UUID=1ce2c743-20a4-4e02-a9d4-d069f91c0534 none            swap    sw              0       0
```
# 禁止cdrom安装源

修改apt source list，注释掉cdrom (/etc/apt/sources.list)

原因：

E: Failed to fetch cdrom://Ubuntu-Server 16.04.3 LTS _Xenial Xerus_ - Release amd64 (20170801)/dists/xenial/main/binary-amd64/Packages  Please use apt-cdrom to make this CD-ROM recognized by APT. apt-get update cannot be used to add new CD-ROMs

```
#deb cdrom:[Ubuntu-Server 16.04.3 LTS _Xenial Xerus_ - Release amd64 (20170801)]/ xenial main restricted
```

# 安装docker engine
```
apt update
apt install docker.io

```
# 安装minikube和kubectl

```
#install minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

#install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/

```