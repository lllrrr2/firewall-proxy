## 前期准备 
- 一台可用的 VPS   
- 一个 **没有被 DNS 污染的域名**    
- **确保使用的域名已经成功解析到你的 VPS服务器，并且 未开启 CDN选项**   
- 纯净的 Debian 9 && Ubuntu 16~18 系统 
## 配置内容 
- 升级并安装必要软件   
```bash
apt update && apt install -y wget
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
- 安装 Docker && Trojan     
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan-go
vim /etc/trojan-go/config.json
```
> 配置a ：不需要使用 **CDN** 进行流量中转 （你的服务器 IP地址 未被墙）  
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
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    }
}
```
>> 配置b ：需要使用**CDN** 进行流量中转 （你的服务器 IP地址 已经被墙）  
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
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改为你的域名
        "fallback_port": 3000 
    },
      "websocket": {
        "enabled": true,
        "path": "/your_path",    #修改为你设定的路径
        "hostname": "your_domain.com",   #修改为你的域名
        "obfuscation_password": "password1",   #修改为另一个密码，切勿与上面的密码相同
        "double_tls": false
    }
}
```
**`:wq!`保存并退出** 

- 安装 Nginx  
```bash
apt update && apt install -y nginx
```
- 移除默认（ **your_domain.com 改为你的域名**）
```bash
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
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
nginx -s reload
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 升级 Trojan-Go 内核
```bash
docker stop trojan-go
docker rm trojan-go
docker rmi teddysun/trojan-go
docker pull teddysun/trojan-go
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
## 客户端的使用 
PC平台 ：https://github.com/Trojan-Qt5/Trojan-Qt5/releases   
安卓平台 ：[点击下载](https://github.com/charlieethan/firewall-proxy/releases/download/V0.5.1m/Igniter-Go-v0.5.1.apk)			

**移动版推荐配置如下 ：**		
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
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "auto"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
        "enabled": false,
        "path": "\/path",
        "double_tls": false,
        "obfuscation_password": ""
    }
}
```		
**注：如果开启 websocket ，请自行按照服务器端对本地配置进行修改**
