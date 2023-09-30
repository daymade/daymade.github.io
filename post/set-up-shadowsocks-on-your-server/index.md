
<!--more-->

## 购买VPS

### digitalocean ($5/month size)

 * 注册
 * 充值 信用卡或者paypal

https://cloud.digitalocean.com/droplets

### github edu package

[Learn How to Code Using the Student Developer Pack - GitHub](https://education.github.com/pack)

## 安装系统

### ubuntu 14.04

### 添加sshkey

* 创建sshkey 参考github教程
* 将 id_rsa.pub 文件内容

### 登录

```bash
$ ssh root@{yourserverip}
```

## 安装python 和 pip

```bash
$ apt-get update
$ apt-get install python-pip
```

## 安装shadowsocks

* pip install shadowsocks

## 配置开启

* ssserver -k password -m aes-256-cfb

    ```
    cd /etc
    touch shadowsocks.json
    vim shadowsocks.json
    cat shadowsocks.json

    {                                 
        "server":"{yourserverip}",    
        "server_port":8388,           
        "local_address":"127.0.0.1",  
        "local_port":1080,            
        "password":"{yourpasswd}",    
        "timeout":300,                
        "method":"aes-256-cfb",       
        "fast_open":false             
    }       
    ```
* ssserver -c /etc/shadowsocks.json -d start
* less /var/log/shadowsocks.log
* ssserver -c /etc/shadowsocks.json -d stop
