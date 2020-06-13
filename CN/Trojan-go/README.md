## 前期准备 
- 一台可用的 VPS   
- 一个 **没有被 DNS 污染的域名**    
- **确保使用的域名已经成功解析到你的 VPS服务器，并且 未开启 CDN选项**   
## 搭建环境
Debian 9/10 && Ubuntu 16/18/20
## 配置内容 
- 升级并安装必要软件   
```bash
apt update && apt -y install socat wget vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装脚本 
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申请证书 （请将 **your_domain.com** 改为你的域名）  
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/trojan-go
acme.sh --installcert -d your_domain.com --fullchain-file /etc/trojan-go/server.crt --key-file /etc/trojan-go/server.key --ecc
```
- 安装 Docker && Nginx && Trojan     
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan-go
```
- 修改 Trojan-go 配置
```bash
vim /etc/trojan-go/config.json
```
<details>
<summary>配置1 ：不使用CDN</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    }
}
```
</details>

<details>
<summary>配置2 ：使用CDN，且你信任你的CDN提供商</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改为你设定的路径
    "hostname": "your_domain.com"   #修改为你的域名
    }
}
```
</details>  

<details>
<summary>配置3 ：使用CDN，但你不信任你的CDN提供商（例如国内厂商的CDN）</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改为你设定的路径
    "hostname": "your_domain.com"   #修改为你的域名
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1"   #修改为另一个密码，请勿与上方密码一致
  }
}
```
</details>

- 修改 Nginx 配置  
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
**复制下列配置**  
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name your_domain.com;   #修改为你的域名
    location / {
        proxy_pass https://your_proxy.com;   #修改为你想伪装的网站域名，例如 https://unsplash.com/  
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;  #修改为你服务器的 IP地址
    return 301 https://your_domain.com$request_uri;   #修改为你的域名
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
server {
	listen 127.0.0.1:3000;
	server_name _;
	return 400;
}
```
- 启动服务  
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 更新软件
- 更新 Trojan-go
```bash
docker stop trojan-go
docker rm trojan-go
docker rmi teddysun/trojan-go
docker pull teddysun/trojan-go
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
- 更新 Nginx
```bash
docker stop nginx
docker rm nginx
docker rmi nginx
docker pull nginx
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
```
## 客户端的使用 
PC平台 ：https://github.com/Trojan-Qt5/Trojan-Qt5/releases   
安卓平台 ：[点击下载](https://github.com/charlieethan/firewall-proxy/releases/download/V0.7.0/Igniter-Go-v0.7.0.apk)			

**移动版推荐配置如下 ：**		
<details>
<summary>对应配置1</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    }
}
```
</details>

<details>
<summary>对应配置2</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    }
}
```
</details>

<details>
<summary>对应配置3</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1" 
  }
}
```
</details>
