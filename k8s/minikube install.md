minikube install

1. 首先修改docker配置
	修改文件是：
	
2. 
``` shell
	minikube start --vm-driver none
```
3. 说明minikube(boot2docker)的默认用户密码是
``` txt
user: docker
pass: tcuser
```
常见minikube vm的方式：
minikube ssh
or
minikube ip
ssh docker@ip

ssh -i ./id_rsa docker@192.168.39.158
其中id_rsa文件在：~/.minikube/machines/minikube



/etc/docker/daemon.json

{
  "bip": "192.168.1.5/24",
  "fixed-cidr": "192.168.1.5/25",
  "fixed-cidr-v6": "2001:db8::/64",
  "mtu": 1500,
  "default-gateway": "192.168.1.1",
  "default-gateway-v6": "2001:db8:abcd::89",
  "dns": ["10.20.1.2","10.20.1.3"],
  "insecure-registries":["192.168.178.167"]
 }

检查代理
	/etc/systemd/system/docker.service.d/http-proxy.conf
检查daemon.json
	/etc/docker/daemon.json
