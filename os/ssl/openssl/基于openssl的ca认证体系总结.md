<!-- MarkdownTOC -->

- 概念
    - ca中心概念
- 根ca（rootca/democa）中心的建立
    - [创建ca中心私钥](#必须)
    - [创建ca中心公钥\(也可以通过证书导出\)](#可选)
    - [创建ca中心根证书，自签名ca证书](#必须)
    - [通过证书导出公钥](#可选)
    - 测试ca签发证书-给nginx颁发证书
        - 步骤一：创建nginx私钥
        - 步骤二：创建nginx的证书请求，注意：因为是服务认证CN名字是IP。
        - 步骤三：签发证书
        - 步骤四：导出公钥
        - 步骤五：配置nginx
        - 步骤六：IE设置
        - 步骤七：测试
- 二级ca中心的建立
    - [创建二级CA私钥](#必须)
    - [创建二级CA公钥\(也可以通过证书导出\)](#可选)
    - [创建二级CA根证书](#必须且是rootca签发)
    - [通过证书导出二级CA公钥](#可选)
    - 测试二级CA签发证书-给nginx颁发证书
- 三级ca中心的建立
    - 创建三级级CA私钥
    - 创建三级级CA公钥\(也可以通过证书导出\)
    - 创建三级CA根证书
    - 通过证书导出三级CA公钥
    - 测试三级CA签发证书-给nginx颁发证书
- 总结
- 扩展测试nginx的双向ssl认证
- 问题
    - 问题一
    - 问题二
    - 问题三
- 工具
- 感谢

<!-- /MarkdownTOC -->

# 概念
## ca中心概念
	ca证书的颁发机构。ca中心是分级别的，从根ca到子ca形式一颗倒立的树。
>
<br>
下面说明3级ca的创建过程


# 根ca（rootca/democa）中心的建立
创建根ca中心，使用自签名方式，换句话说，不需要生产证书请求，让其他CA签发。

## 创建ca中心私钥[必须]

根据加密算法不同涉及输入密码问题。在使用des加密算法时，每次使用ca私钥都有输入密码<br>

- 方法一 带密码<br> 
openssl genrsa -des3 -out ca.key 1024/2048 
如果麻烦可以去掉密码限制
openssl rsa -in ca.key -out ca.key

- 方法二 不带密码<br>
```
openssl genrsa -out ca.key 1024/2048  
```

## 创建ca中心公钥(也可以通过证书导出)[可选]
```
openssl rsa -in ca.key -pubout -out ca-pub.key
```

## 创建ca中心根证书，自签名ca证书[必须]
``` shell
root@u-s1:~/workspace/ssl/openssl/rootca# openssl req -new -x509 -key ca.key -out cacert.crt -days 3650
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:CAOrg
Organizational Unit Name (eg, section) []:CAOrgUnit
Common Name (e.g. server FQDN or YOUR name) []:CACN
Email Address []:caadmin@sina.com
root@u-s1:~/workspace/ssl/openssl/rootca# 
```
## 通过证书导出公钥[可选]
```
openssl x509 -in cacert.crt -pubkey >> ca-pub.key
```

到此为止，根CA中心所有需要的私钥，公钥，证书均已经具备。<br>
说明,公钥和CA证书均可以通过ca私钥导出，因此ca私钥很关键，通常证书的ca中心的私钥是保管在硬件中，并且硬件保管在银行保险库里的。


创建完根ca以后，其实ca中心已经具备全部功能了，可以对外颁发证书了，另外cacert.crt需要对外公开，比如预埋在浏览器中。
这样由此ca中心颁发的证书就可以顺利通过验证了。

## 测试ca签发证书-给nginx颁发证书

### 步骤一：创建nginx私钥
``` shell
openssl genrsa -out ./nginx/nginx.key 1024/2048
```

### 步骤二：创建nginx的证书请求，注意：因为是服务认证CN名字是IP。
```
openssl req -new -key ./nginx/nginx.key -out ./nginx/nginx-cert.csr
```

    root@u-s1:~/workspace/ssl/openssl# openssl req -new -key ./nginx/nginx.key -out ./nginx/nginx-cert.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:CN
    State or Province Name (full name) [Some-State]:Beijing
    Locality Name (eg, city) []:Beijing
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:ZOrg
    Organizational Unit Name (eg, section) []:ZOrgUnit                  
    Common Name (e.g. server FQDN or YOUR name) []:192.168.178.137
    Email Address []:

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

### 步骤三：签发证书
``` shell
openssl ca -extensions v3_ca -in ./nginx/nginx-cert.csr -days 365 -out ./nginx/nginx-cert.crt -cert demoCA/cacert.crt -keyfile demoCA/private/ca.key
```
    root@u-s1:~/workspace/ssl/openssl# openssl ca -extensions v3_ca -in ./nginx/nginx-cert.csr -days 365 -out ./nginx/nginx-cert.crt -cert demoCA/cacert.crt -keyfile demoCA/private/ca.key
    Using configuration from /usr/lib/ssl/openssl.cnf
    Check that the request matches the signature
    Signature ok
    Certificate Details:
            Serial Number: 2 (0x2)
            Validity
                Not Before: May 22 08:29:21 2018 GMT
                Not After : May 22 08:29:21 2019 GMT
            Subject:
                countryName               = CN
                stateOrProvinceName       = Beijing
                organizationName          = ZOrg
                organizationalUnitName    = ZOrgUnit
                commonName                = 192.168.178.137
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    63:EF:5D:B0:03:6B:20:FA:37:E6:6E:C6:D9:B3:B1:D7:81:0E:37:6C
                X509v3 Authority Key Identifier: 
                    keyid:B0:A1:EE:B6:79:A6:28:CE:3E:37:E2:B5:29:F9:13:75:7B:C3:B0:4E

                X509v3 Basic Constraints: 
                    CA:TRUE
    Certificate is to be certified until May 22 08:29:21 2019 GMT (365 days)
    Sign the certificate? [y/n]:y


    1 out of 1 certificate requests certified, commit? [y/n]y
    Write out database with 1 new entries
    Data Base Updated
    root@u-s1:~/workspace/ssl/openssl# 



### 步骤四：导出公钥
``` shell
openssl x509 -in ./nginx/nginx-cert.pem -pubkey >> ./nginx/nginx-pub.key
```

### 步骤五：配置nginx

1）编辑/etc/nginx/nginx.conf

            ##
            # SSL Settings
            ##
    #       ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    #       ssl_prefer_server_ciphers on;
            ssl on;
            ssl_certificate /root/workspace/ssl/openssl/nginx/nginx-cert.crt;
            ssl_certificate_key /root/workspace/ssl/openssl/nginx/nginx.key;

2) 编辑/etc/nginx/sites-enabled/default
            listen 80 default_server;
            listen [::]:80 default_server;

            # SSL configuration
            #
             listen 443 ssl default_server;
             listen [::]:443 ssl default_server;

3） 重启nginx
	
        systemctl restart nginx

### 步骤六：IE设置

导入/root/workspace/ssl/openssl/demoCA/cacert.crt到IE的“受信任的根证书颁发机构”

### 步骤七：测试
IE通过 https://192.168.178.137,页面正常显示，bingo！

或者通过curl命令
``` shell
curl --cacert ./demoCA/cacert.crt https://192.168.178.137
```
    root@u-s1:~/workspace/ssl/openssl# curl --cacert ./demoCA/cacert.crt https://192.168.178.137
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>

``` shell
curl  https://192.168.178.137
```
    root@u-s1:~/workspace/ssl/openssl# curl  https://192.168.178.137
    curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
    More details here: http://curl.haxx.se/docs/sslcerts.html

    curl performs SSL certificate verification by default, using a "bundle"
     of Certificate Authority (CA) public keys (CA certs). If the default
     bundle file isn't adequate, you can specify an alternate file
     using the --cacert option.
    If this HTTPS server uses a certificate signed by a CA represented in
     the bundle, the certificate verification probably failed due to a
     problem with the certificate (it might be expired, or the name might
     not match the domain name in the URL).
    If you'd like to turn off curl's verification of the certificate, use
     the -k (or --insecure) option.

``` shell
curl -k  https://192.168.178.137
```
    root@u-s1:~/workspace/ssl/openssl# curl -k  https://192.168.178.137
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>

``` shell
curl -I -v --cacert ./demoCA/cacert.crt https://192.168.178.137
```
    root@u-s1:~/workspace/ssl/openssl# curl -I -v --cacert ./demoCA/cacert.crt https://192.168.178.137
    * Rebuilt URL to: https://192.168.178.137/
    *   Trying 192.168.178.137...
    * Connected to 192.168.178.137 (192.168.178.137) port 443 (#0)
    * found 1 certificates in ./demoCA/cacert.crt
    * found 692 certificates in /etc/ssl/certs
    * ALPN, offering http/1.1
    * SSL connection using TLS1.2 / ECDHE_RSA_AES_128_GCM_SHA256
    * 	 server certificate verification OK
    * 	 server certificate status verification SKIPPED
    * 	 common name: 192.168.178.137 (matched)
    * 	 server certificate expiration date OK
    * 	 server certificate activation date OK
    * 	 certificate public key: RSA
    * 	 certificate version: #3
    * 	 subject: C=CN,ST=Beijing,O=ZOrg,OU=ZOrgUnit,CN=192.168.178.137
    * 	 start date: Tue, 22 May 2018 08:29:21 GMT
    * 	 expire date: Wed, 22 May 2019 08:29:21 GMT
    * 	 issuer: C=CN,ST=Beijing,L=Beijing,O=CAOrg,OU=CAOrgUnit,CN=CACN,EMAIL=caadmin@sina.com
    * 	 compression: NULL
    * ALPN, server accepted to use http/1.1
    > HEAD / HTTP/1.1
    > Host: 192.168.178.137
    > User-Agent: curl/7.47.0
    > Accept: */*
    > 
    < HTTP/1.1 200 OK
    HTTP/1.1 200 OK
    < Server: nginx/1.10.3 (Ubuntu)
    Server: nginx/1.10.3 (Ubuntu)
    < Date: Tue, 22 May 2018 08:54:22 GMT
    Date: Tue, 22 May 2018 08:54:22 GMT
    < Content-Type: text/html
    Content-Type: text/html
    < Content-Length: 612
    Content-Length: 612
    < Last-Modified: Tue, 22 May 2018 07:10:27 GMT
    Last-Modified: Tue, 22 May 2018 07:10:27 GMT
    < Connection: keep-alive
    Connection: keep-alive
    < ETag: "5b03c263-264"
    ETag: "5b03c263-264"
    < Accept-Ranges: bytes
    Accept-Ranges: bytes

    < 
    * Connection #0 to host 192.168.178.137 left intact


# 二级ca中心的建立
	目的，鉴于CA中心的安全性，不建议直接通过根CA进行证书颁发，而通过根CA认证的二级CA颁发证书。

## 创建二级CA私钥[必须]
``` shell
openssl genrsa -out ./2ndca/ca.key 1024/2048
```
## 创建二级CA公钥(也可以通过证书导出)[可选]
``` shell
openssl rsa -in ./2ndca/ca.key -pubout -out ./2ndca/ca-pub.key
```
## 创建二级CA根证书[必须且是rootca签发]
1）创建csr
```
openssl req -new -key ./2ndca/ca.key -out ./2rdca/cacert.csr
```
    root@u-s1:~/workspace/ssl/openssl# openssl req -new -key ./2ndca/ca.key -out ./2ndca/cacert.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:CN
    State or Province Name (full name) [Some-State]:Beijing
    Locality Name (eg, city) []:Beijing
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:CA2Org
    Organizational Unit Name (eg, section) []:CA2OrgUnit
    Common Name (e.g. server FQDN or YOUR name) []:CA2CN
    Email Address []:cn2cnadmin@sina.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

2)创建证书

```
openssl ca -extensions v3_ca -in ./2ndca/cacert.csr -days 365 -out ./2ndca/cacert.crt -cert demoCA/cacert.crt -keyfile demoCA/private/ca.key
```

    root@u-s1:~/workspace/ssl/openssl# openssl ca -extensions v3_ca -in ./2ndca/cacert.csr -days 365 -out ./2ndca/cacert.crt -cert demoCA/cacert.crt -keyfile demoCA/private/ca.key
    Using configuration from /usr/lib/ssl/openssl.cnf
    Check that the request matches the signature
    Signature ok
    Certificate Details:
            Serial Number: 3 (0x3)
            Validity
                Not Before: May 22 12:40:03 2018 GMT
                Not After : May 22 12:40:03 2019 GMT
            Subject:
                countryName               = CN
                stateOrProvinceName       = Beijing
                organizationName          = CA2Org
                organizationalUnitName    = CA2OrgUnit
                commonName                = CA2CN
                emailAddress              = cn2cnadmin@sina.com
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    84:52:8B:BA:F0:D9:0C:F3:CC:8E:B3:2C:05:4C:79:F0:1E:DF:84:EE
                X509v3 Authority Key Identifier: 
                    keyid:B0:A1:EE:B6:79:A6:28:CE:3E:37:E2:B5:29:F9:13:75:7B:C3:B0:4E

                X509v3 Basic Constraints: 
                    CA:TRUE
    Certificate is to be certified until May 22 12:40:03 2019 GMT (365 days)
    Sign the certificate? [y/n]:y


    1 out of 1 certificate requests certified, commit? [y/n]y
    Write out database with 1 new entries
    Data Base Updated
    root@u-s1:~/workspace/ssl/openssl# 

## 通过证书导出二级CA公钥[可选]
``` shell
openssl x509 -in ./2ndca/cacert.crt -pubkey >> ./2ndca/ca-pub.key
```
## 测试二级CA签发证书-给nginx颁发证书
``` shell
openssl genrsa -out ./2ndca/nginx/nginx.key 1024/2048

openssl req -new -key ./2ndca/nginx/nginx.key -out ./2ndca/nginx/nginx-cert.csr

```
    root@u-s1:~/workspace/ssl/openssl# openssl req -new -key ./2rdca/nginx/nginx.key -out ./2rdca/nginx/nginx-cert.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:CN
    State or Province Name (full name) [Some-State]:Beijing
    Locality Name (eg, city) []:Beijing
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Z2Org
    Organizational Unit Name (eg, section) []:Z2OrgUnit
    Common Name (e.g. server FQDN or YOUR name) []:192.168.178.137
    Email Address []:

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
    root@u-s1:~/workspace/ssl/openssl# 

``` shell
openssl ca -extensions v3_ca -in ./2ndca/nginx/nginx-cert.csr -days 365 -out ./2ndca/nginx/nginx-cert.crt -cert ./2ndca/cacert.crt -keyfile ./2ndca/ca.key
````
    root@u-s1:~/workspace/ssl/openssl# openssl ca -extensions v3_ca -in ./2rdca/nginx/nginx-cert.csr -days 365 -out ./2rdca/nginx/nginx-cert.crt -cert ./2rdca/cacert.crt -keyfile ./2rdca/ca.key
    Using configuration from /usr/lib/ssl/openssl.cnf
    Check that the request matches the signature
    Signature ok
    Certificate Details:
            Serial Number: 4 (0x4)
            Validity
                Not Before: May 22 12:51:47 2018 GMT
                Not After : May 22 12:51:47 2019 GMT
            Subject:
                countryName               = CN
                stateOrProvinceName       = Beijing
                organizationName          = Z2Org
                organizationalUnitName    = Z2OrgUnit
                commonName                = 192.168.178.137
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    56:2E:F9:E3:31:55:1D:4E:93:0F:26:C0:0A:D9:C6:E4:31:5B:5B:A3
                X509v3 Authority Key Identifier: 
                    keyid:84:52:8B:BA:F0:D9:0C:F3:CC:8E:B3:2C:05:4C:79:F0:1E:DF:84:EE

                X509v3 Basic Constraints: 
                    CA:TRUE
    Certificate is to be certified until May 22 12:51:47 2019 GMT (365 days)
    Sign the certificate? [y/n]:y


    1 out of 1 certificate requests certified, commit? [y/n]y
    Write out database with 1 new entries
    Data Base Updated
    root@u-s1:~/workspace/ssl/openssl# 

``` shell
openssl x509 -in ./2ndca/nginx/nginx-cert.crt -pubkey >> ./2ndca/nginx/nginx-pub.key
```
修改nginx配置
``` 
/etc/nginx/nginx.conf
        ssl_certificate /root/workspace/ssl/openssl/2ndca/nginx/nginx-cert.crt;
        ssl_certificate_key /root/workspace/ssl/openssl/2ndca/nginx/nginx.key;
```

OK:
curl -k -I https://192.168.178.137
curl -I --cacert ./2ndca/cacert.crt https://192.168.178.137

NG:
使用rootca不行
curl -I --cacert ./demoCA/cacert.crt https://192.168.178.137
curl -I https://192.168.178.137

IE测试结果是将democa(demoCA/cacert.crt)导入受信任的根证书颁发机构，将2ndca证书(./2ndca/cacert.crt)
导入到中级证书颁发机构后，访问成功，但是少其一均不可，即证书链不能断。



# 三级ca中心的建立
    目的，测试3级CA的使用场景
## [创建三级级CA私钥](#必须)
```
openssl genrsa -out ./3rdca/ca.key 1024/2048
```
## [创建三级级CA公钥\(也可以通过证书导出\)](#可选)
```
openssl rsa -in ./3rdca/ca.key -pubout -out ./3rdca/ca-pub.key

```
## [创建三级CA根证书](#必须且是rootca签发)
1)创建CSR
``` shell
openssl req -new -key ./3rdca/ca.key -out ./3rdca/cacert.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=CA3Org/OU=CA3OrgUnit/CN=CA3CN/emailAddress=cn3cnadmin@sina.com"
```
2)创建证书
``` shell
openssl ca -extensions v3_ca -in ./3rdca/cacert.csr -days 365 -out ./3rdca/cacert.crt -cert ./2ndca/cacert.crt -keyfile ./2ndca/ca.key
```
## [通过证书导出三级CA公钥](#可选)

## 测试三级CA签发证书-给nginx颁发证书

```
openssl genrsa -out ./3rdca/nginx/nginx.key 1024/2048
```
```
openssl req -new -key ./3rdca/nginx/nginx.key -out ./3rdca/nginx/nginx-cert.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=Z3Org/OU=Z3OrgUnit/CN=192.168.178.137"
```
```
openssl ca -extensions v3_ca -in ./3rdca/nginx/nginx-cert.csr -days 365 -out ./3rdca/nginx/nginx-cert.crt -cert ./3rdca/cacert.crt -keyfile ./3rdca/ca.key
```
```
openssl x509 -in ./3rdca/nginx/nginx-cert.crt -pubkey >> ./3rdca/nginx/nginx-pub.key
```

```
/etc/nginx/nginx.conf
        ssl_certificate /root/workspace/ssl/openssl/3rdca/nginx/nginx-cert.crt;
        ssl_certificate_key /root/workspace/ssl/openssl/3rdca/nginx/nginx.key;
```

    root@u-s1:~/workspace/ssl/openssl# curl -I --cacert ./demoCA/cacert.crt https://192.168.178.137
    curl: (60) server certificate verification failed. CAfile: ./demoCA/cacert.crt CRLfile: none
    More details here: http://curl.haxx.se/docs/sslcerts.html

    curl performs SSL certificate verification by default, using a "bundle"
     of Certificate Authority (CA) public keys (CA certs). If the default
     bundle file isn't adequate, you can specify an alternate file
     using the --cacert option.
    If this HTTPS server uses a certificate signed by a CA represented in
     the bundle, the certificate verification probably failed due to a
     problem with the certificate (it might be expired, or the name might
     not match the domain name in the URL).
    If you'd like to turn off curl's verification of the certificate, use
     the -k (or --insecure) option.
    root@u-s1:~/workspace/ssl/openssl# curl -I --cacert ./2ndca/cacert.crt https://192.168.178.137
    curl: (60) server certificate verification failed. CAfile: ./2ndca/cacert.crt CRLfile: none
    More details here: http://curl.haxx.se/docs/sslcerts.html

    curl performs SSL certificate verification by default, using a "bundle"
     of Certificate Authority (CA) public keys (CA certs). If the default
     bundle file isn't adequate, you can specify an alternate file
     using the --cacert option.
    If this HTTPS server uses a certificate signed by a CA represented in
     the bundle, the certificate verification probably failed due to a
     problem with the certificate (it might be expired, or the name might
     not match the domain name in the URL).
    If you'd like to turn off curl's verification of the certificate, use
     the -k (or --insecure) option.
    root@u-s1:~/workspace/ssl/openssl# curl -I --cacert ./3rdca/cacert.crt https://192.168.178.137
    HTTP/1.1 200 OK
    Server: nginx/1.10.3 (Ubuntu)
    Date: Tue, 22 May 2018 14:27:18 GMT
    Content-Type: text/html
    Content-Length: 612
    Last-Modified: Tue, 22 May 2018 07:10:27 GMT
    Connection: keep-alive
    ETag: "5b03c263-264"
    Accept-Ranges: bytes

    root@u-s1:~/workspace/ssl/openssl# 


# 总结
```
democa crt--》 nginx crt  
          --》 2ndca crt --》 nginx crt
                        --》 3rdca crt -->nginx crt
```
<br>
IE表现：
    如果要认证第一个nginx，需要导入demoCA cert
    如果要认证第二个nginx，需要导入demoCA cert和2ndca cert
    如果要认证第三个nginx，需要导入demoCA cert和2ndca cert和3rdca cert
<br>
curl的表现：

    在curl --cacert 制定nginx证书颁发机构的ca cert即可，不需要到rootca（即根ca证书）

但是无论哪种情况下，也不是只拿着rootca的证书就能完成客户端ca的校验的。


# 扩展测试nginx的双向ssl认证

使用二级CA（2ndca）的私钥和证书作为本测试的客户端证书，2ndca的cert和key作为nginx客户的cert和key，ca中心cert使用 democa的cert，服务端使用democa颁发给nginx的证书（第一个nginx的证书），配置如下：

```
/etc/nginx/nginx.conf
        ssl on;
        ssl_certificate /root/workspace/ssl/openssl/nginx/nginx-cert.crt;
        ssl_certificate_key /root/workspace/ssl/openssl/nginx/nginx.key;
        ssl_client_certificate /root/workspace/ssl/openssl/demoCA/cacert.crt;
        ssl_verify_client on;
其中 ssl_client_certificate /root/workspace/ssl/openssl/demoCA/cacert.crt;; 的意思是使用 CA 证书来验证请求带的客户端证书是否是该 CA 签发的。

```
```
root@u-s1:~/workspace/ssl/openssl# curl -I --cert ./2ndca/cacert.crt --key ./2ndca/ca.key  --cacert ./demoCA/cacert.crt https://192.168.178.137
HTTP/1.1 200 OK
Server: nginx/1.10.3 (Ubuntu)
Date: Tue, 22 May 2018 14:52:43 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 22 May 2018 07:10:27 GMT
Connection: keep-alive
ETag: "5b03c263-264"
Accept-Ranges: bytes

root@u-s1:~/workspace/ssl/openssl# 

root@u-s1:~/workspace/ssl/openssl# curl  -v --cert ./2ndca/cacert.crt --key ./2ndca/ca.key  --cacert ./demoCA/cacert.crt https://192.168.178.137
* Rebuilt URL to: https://192.168.178.137/
*   Trying 192.168.178.137...
* Connected to 192.168.178.137 (192.168.178.137) port 443 (#0)
* found 1 certificates in ./demoCA/cacert.crt
* found 692 certificates in /etc/ssl/certs
* ALPN, offering http/1.1
* SSL connection using TLS1.2 / ECDHE_RSA_AES_128_GCM_SHA256
*    server certificate verification OK
*    server certificate status verification SKIPPED
*    common name: 192.168.178.137 (matched)
*    server certificate expiration date OK
*    server certificate activation date OK
*    certificate public key: RSA
*    certificate version: #3
*    subject: C=CN,ST=Beijing,O=ZOrg,OU=ZOrgUnit,CN=192.168.178.137
*    start date: Tue, 22 May 2018 08:29:21 GMT
*    expire date: Wed, 22 May 2019 08:29:21 GMT
*    issuer: C=CN,ST=Beijing,L=Beijing,O=CAOrg,OU=CAOrgUnit,CN=CACN,EMAIL=caadmin@sina.com
*    compression: NULL
* ALPN, server accepted to use http/1.1
> GET / HTTP/1.1
> Host: 192.168.178.137
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.10.3 (Ubuntu)
< Date: Tue, 22 May 2018 15:05:23 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 22 May 2018 07:10:27 GMT
< Connection: keep-alive
< ETag: "5b03c263-264"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host 192.168.178.137 left intact
root@u-s1:~/workspace/ssl/openssl# 
root@u-s1:~/workspace/ssl/openssl# curl  -v --cacert ./demoCA/cacert.crt https://192.168.178.137
* Rebuilt URL to: https://192.168.178.137/
*   Trying 192.168.178.137...
* Connected to 192.168.178.137 (192.168.178.137) port 443 (#0)
* found 1 certificates in ./demoCA/cacert.crt
* found 692 certificates in /etc/ssl/certs
* ALPN, offering http/1.1
* SSL connection using TLS1.2 / ECDHE_RSA_AES_128_GCM_SHA256
*    server certificate verification OK
*    server certificate status verification SKIPPED
*    common name: 192.168.178.137 (matched)
*    server certificate expiration date OK
*    server certificate activation date OK
*    certificate public key: RSA
*    certificate version: #3
*    subject: C=CN,ST=Beijing,O=ZOrg,OU=ZOrgUnit,CN=192.168.178.137
*    start date: Tue, 22 May 2018 08:29:21 GMT
*    expire date: Wed, 22 May 2019 08:29:21 GMT
*    issuer: C=CN,ST=Beijing,L=Beijing,O=CAOrg,OU=CAOrgUnit,CN=CACN,EMAIL=caadmin@sina.com
*    compression: NULL
* ALPN, server accepted to use http/1.1
> GET / HTTP/1.1
> Host: 192.168.178.137
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 400 Bad Request
< Server: nginx/1.10.3 (Ubuntu)
< Date: Tue, 22 May 2018 15:11:50 GMT
< Content-Type: text/html
< Content-Length: 262
< Connection: close
< 
<html>
<head><title>400 No required SSL certificate was sent</title></head>
<body bgcolor="white">
<center><h1>400 Bad Request</h1></center>
<center>No required SSL certificate was sent</center>
<hr><center>nginx/1.10.3 (Ubuntu)</center>
</body>
</html>
* Closing connection 0
root@u-s1:~/workspace/ssl/openssl# 

```
```
原理：
    通过democa颁发两对公钥私钥对，其中公钥通过证书方式提供。

        client                          server

请求    client cert      ---》   

                        《---            server cert
       客户端通过democa判断server cert合法性
                                        server通过demeca校验client cert合法性

以上验证通过以后
       客户端从server cert中拿到 server的公钥
                                        server从client cert中能到 client的公钥

选择任何一对公钥私钥将对称加密的密码通知client和server，密码协调工作

然后对称加密传输开始，数据通信开始

备注：以上逻辑简化了https的协议，此处只是示意。



```

其中如何使用IE测试双向认证的方法：
将客户端的证书和key转换成IE认识的格式
　　　　openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.pfx
并且导入到IE，在IE访问https://192.169.178.137的时候提示选择提交的client证书，本测试是CA2CN证书，即二级CA的证书。



                                      
# 问题

## 问题一
	The organizationName field needed to be the same in the
	CA certificate (CAOrg) and the request (ZOrg)

	修改openssl.cnf
``` conf
	# For the CA policy
[ policy_match ]
#countryName            = match
#stateOrProvinceName    = match
#organizationName       = match
countryName             = supplied
stateOrProvinceName     = supplied
organizationName        = supplied
```

## 问题二
如果IE导入证书，如果导入的位置不对，会发现导入后，通过
(IE--》Internet Options--》Contents--》Certificates)找不到了，没法再进行维护。

方法：CMD--》MMC--》添加删除管理单元--》证书 可以看到，可以维护。


## 问题三
如果客户端是IE，客户端key和cert怎么提供给IE

方法：
openssl pkcs12 -export -clcerts -in ./2ndca/cacert.crt -inkey ./2ndca/ca.key -out ie-client.pfx

# 工具

create_cert.sh

``` shell
#!/bin/bash
#
# 通过demoCA创建harbor证书的命令，其中harbor证书的CN是IP：
#  ./create_cert.sh harbor 192.168.178.167 demoCA
# 通过二级CA创建harbor的证书的命令，其中harbor的证书CN是IP：
#  ./create_cert.sh harbor2 192.168.178.167 2ndca
#  
# author: forcl@sina.com
#

if [ "$#" -ne "3" ]; then
    echo "usage: $0 <certname> <cnname> <caname>"
    exit 1
fi

CERT_NM=$1
CN_NM=$2
CA_NM=$3

if [ "x$CERT_NM" == "x" ];then
   CERT_NM=test
fi
if [ "x$CN_NM" == "x" ];then
   CN_NM=192.168.178.137
fi
if [ "x$CA_NM" == "x" ];then
    CA_NM=demoCA
fi
echo "creating dir $CERT_NM."
mkdir $CERT_NM

echo "create $CERT_NM private key."
openssl genrsa -out ./$CERT_NM/$CERT_NM.key 1024/2048

echo "create cert request."
openssl req -new -key ./$CERT_NM/$CERT_NM.key -out ./$CERT_NM/$CERT_NM.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=ZOrg/OU=ZOrgUnit/CN=$CN_NM"

echo "create cert." 
if [ "x$CA_NM" == "xdemoCA" ];then
    openssl ca -extensions v3_ca -batch -in ./$CERT_NM/$CERT_NM.csr -days 365 -out ./$CERT_NM/$CERT_NM.crt -cert ./$CA_NM/cacert.crt -keyfile ./$CA_NM/private/ca.key
else
    openssl ca -extensions v3_ca -batch -in ./$CERT_NM/$CERT_NM.csr -days 365 -out ./$CERT_NM/$CERT_NM.crt -cert ./$CA_NM/cacert.crt -keyfile ./$CA_NM/ca.key
fi

echo "create pub key."
openssl x509 -in ./$CERT_NM/$CERT_NM.crt -pubkey >> ./$CERT_NM/$CERT_NM-pub.key


#openssl genrsa -out ./harbor/harbor.key 1024/2048
#openssl req -new -key ./harbor/harbor.key -out ./harbor/harbor.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=ZOrg/OU=ZOrgUnit/CN=197.168.178.167"
#openssl ca -extensions v3_ca -batch  -in ./harbor/harbor.csr -days 365 -out ./harbor/harbor.crt -cert ./demoCA/cacert.crt -keyfile ./demoCA/private/ca.key
#openssl x509 -in ./harbor/harbor.crt -pubkey >> ./harbor/harbor-pub.key

```

# 感谢

http://blog.chinaunix.net/uid-20553497-id-3163297.html
https://www.cnblogs.com/vincentfu/p/5475248.html


