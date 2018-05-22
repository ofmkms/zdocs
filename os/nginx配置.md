# install nginx

在ubuntu系统：

## 安装
```
apt install nginx
```

## 默认document root
``` 
/var/www/html
```

## 配置文件：
```
/etc/nginx/nginx.conf
/etc/nginx/conf.d/*.conf
/etc/nginx/sites-enabled/*
```

## 日志文件
```
/var/log/nginx/access.log
/var/log/nginx/error.log
```

## 系统服务方式启动：
```
systemctl status nginx
```

## 配置证书

参照openssl配置。


