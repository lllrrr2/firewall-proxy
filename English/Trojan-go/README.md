## Prepare 
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first	   
- **Please pay attention to the marks on each line of the config files, and modify them as requested**    	
## Build Environment	
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
## Content 
- install basic tools   
```bash
apt update && apt -y install socat wget vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- install script	 
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- request SSL certificate (modify **your_domain.com** to your domain）  
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/trojan-go
acme.sh --installcert -d your_domain.com --fullchain-file /etc/trojan-go/server.pem --key-file /etc/trojan-go/server.key --ecc
```
- install Docker && Nginx && Trojan    
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan-go
docker pull containrrr/watchtower
```
- modify config files
```bash
vim /etc/trojan-go/config.json
```

<details>
<summary>Configuration 1 : Do NOT need to use CDN</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #modify to your password
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #modify to your domain
        "fallback_port": 3000 
    }
}
```
</details>

<details>
<summary>Configuration 2 : Need to use CDN,and you trust your CDN supplier</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #modify to your password
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #modify to your domain
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #modify to your path
    "host": "your_domain.com"   #modify to your domain
    }
}
```
</details>  

<details>
<summary>Configuration 3 : Need to use CDN,and you DO NOT trust your CDN supplier</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #modify to your password
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #modify to your domain
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #modify to your path
    "host": "your_domain.com"   #modify to your domain
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1"   #modify to another password
  }
}
```
</details>

- modify config files
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
**paste the config file below**  
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name your_domain.com;   #modify to your domain
    location / {
        proxy_pass https://your_proxy.com;   #modify to any website URL you want to disguise  
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;  #modify to your server IP address
    return 301 https://your_domain.com$request_uri;   #modify "your_domain.com" to your domain
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
- Start service  
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ： 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## For new version Update:
Using this config method,**watchtower** can auto detect and update your software,You don't need to update manually any more

## Client 
Android 6.0+ ：[Download](https://github.com/charlieethan/firewall-proxy/releases/download/V0.7.7/Igniter-Go-v0.7.7.apk)			

Windows && Linux && MacOS : [Qv2ray Download](https://github.com/Qv2ray/Qv2ray/releases)	

Plugin to support Trojan-Go in Qv2ray : [Plugin Download](https://github.com/Qv2ray/QvPlugin-Trojan-Go/releases) 		

The Usage of Qv2ray (Chinese Only) : [Usage](https://qv2ray.net/plugins/usage.html)		

**Recommend config on Mobile ：**		
<details>
<summary>For Configuration 1</summary>

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
<summary>For Configuration 2</summary>

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
<summary>For Configuration 3</summary>

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
