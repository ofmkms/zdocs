# harbor install
## 软件准备

### 下载harbor：
```
https://github.com/vmware/harbor/releases/tag/v1.2.2
http://harbor.orientsoft.cn/

harbor-offline-installer-v1.2.2.tgz
tar xvf harbor-offline-installer-v1.2.2.tgz
```

### 下载docker-compose
介质：
https://github.com/docker/compose/releases
安装方法：
```
curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 规划：

###环境：
	主机： harbor  192.168.178.167 
	安装路径：
```
harbor:/harbor # ll
total 8660
lrwxrwxrwx 1 root root      27 Dec 16 09:35 docker-compose -> docker-compose-Linux-x86_64           #docker compose link
-rwxr-xr-x 1 root root 8856808 Dec 16 09:34 docker-compose-Linux-x86_64                             #docker compose 软件
-rw-r--r-- 1 root root      38 Dec 16 11:30 env.sh                                                  #环境初始化脚本，运行harbor之前运行 source ./env.sh
drwxr-xr-x 1 root root     350 Dec 16 22:44 harbor                                                  #harbor软件

harbor相关的数据在
harbor:/ # ls -alrt
total 0
drwxr-xr-x   1 root root  176 Dec 16 23:14 ..
drwxr-xr-x   1 root root  176 Dec 16 23:14 .
drw-------   1 root root  122 Dec 16 23:14 data                                                    #harbor数据
drwxr-xr-x   1 root root  116 Dec 17 05:25 harbor
```

### 安装步骤(root用户)

1. 初始化环境变量
    source /harbor/env.sh

2. 切换到harbor目录
   1) cd /harbor/harbor

3. 启动
   1) ./install.sh --with-clair
   or
   2) docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml [up|down|ps|stop|start ]
 
```
harbor:/harbor # . env.sh 
harbor:/harbor # cd harbor/
harbor:/harbor/harbor # docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml  stop
Stopping nginx              ... done
Stopping harbor-jobservice  ... done
Stopping harbor-ui          ... done
Stopping clair              ... done
Stopping clair-db           ... done
Stopping harbor-db          ... done
Stopping harbor-adminserver ... done
Stopping registry           ... done
Stopping harbor-log         ... done
harbor:/harbor/harbor # docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml  start
Starting log         ... done
Starting adminserver ... done
Starting mysql       ... done
Starting registry    ... done
Starting ui          ... done
Starting jobservice  ... done
Starting proxy       ... done
Starting postgres    ... done
Starting clair       ... done
harbor:/harbor/harbor # 

```
   
   
4. harbor数据目录
	/data

``` shell
harbor:/harbor/harbor # df /data/
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/sda2       61798400 24859864  36133336  41% /
harbor:/harbor/harbor # 
```

5. 测试（harbor暂时没有配置ssl）

   1) 首先修改docker的配置（/etc/docker/daemon.json），允许使用不安全的镜像库
```
harbor:/harbor/harbor # cat /etc/docker/daemon.json

{ 
	"insecure-registries":["192.168.178.167"] 
}

harbor:/harbor/harbor # 
```
   2) 在harbor上创建项目test
    
   3) 重启docker engine
``` shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```
   4) 给镜像打tag
``` shell
harbor:/harbor/harbor # docker tag f895b3fb9e30 192.168.178.167/test/nginx
```
   5) login镜像仓库
``` shell
harbor:/harbor/harbor # docker login http://192.168.178.167
Username: admin
Password: Harbor12345
Login Succeeded
``` 
   6) 推送镜像
``` shell
harbor:/harbor/harbor # docker push 192.168.178.167/test/nginx
The push refers to a repository [192.168.178.167/test/nginx]
995f02eaa054: Pushed 
938981ec0340: Pushed 
2ec5c0a4cb57: Pushed 
latest: digest: sha256:588baef8983ebc098069f87d43196ae36a20e1bc7adee0ed17b87322557d5aaf size: 948
```
5. Clair

docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml up
docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f ./docker-compose.clair.yml down -v

更新clair data
https://github.com/vmware/harbor/blob/master/docs/import_vulnerability_data.md

docker exec -i clair-db psql -U postgres < clear.sql
docker exec -i clair-db psql -U postgres < vulnerability.sql
 

6. 配置Notary和Clair

docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f ./docker-compose.clair.yml up -v
docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f ./docker-compose.clair.yml down -v

https://github.com/vmware/harbor/tree/master/docs
https://github.com/vmware/harbor/blob/master/docs/installation_guide.md


重启：
1. 登录系统 
   1)  sudo su -
   2)  source  /harbor/env.sh && cd /harbor/harbor
   3)  docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml  restart

harbor:/harbor/harbor # docker-compose -f ./docker-compose.yml -f ./docker-compose.clair.yml  restart
Restarting nginx              ... done
Restarting harbor-jobservice  ... done
Restarting harbor-ui          ... done
Restarting clair              ... done
Restarting clair-db           ... done
Restarting harbor-db          ... done
Restarting harbor-adminserver ... done
Restarting registry           ... done
Restarting harbor-log         ... done
harbor:/harbor/harbor # 

# 问题：

## 问题一
	harbor:/harbor/harbor # docker push 192.168.178.167/test/nginx
	The push refers to a repository [192.168.178.167/test/nginx]
	Get https://192.168.178.167/v1/_ping: dial tcp 192.168.178.167:443: getsockopt: connection refused

解决：
	修改docker的配置（/etc/docker/daemon.json），允许使用不安全的镜像库
```
harbor:/harbor/harbor # cat /etc/docker/daemon.json

{ 
	"insecure-registries":["192.168.178.167"] 
}

harbor:/harbor/harbor # 
```

## 问题二

	harbor:/harbor/harbor # docker push 192.168.178.167/test/nginx
	The push refers to a repository [192.168.178.167/test/nginx]
	995f02eaa054: Preparing 
	938981ec0340: Preparing 
	2ec5c0a4cb57: Preparing 
	denied: requested access to the resource is denied

解决：
	login镜像仓库
	```
	harbor:/harbor/harbor # docker login http://192.168.178.167
	Username: admin
	Password: Harbor12345
	Login Succeeded
	```


