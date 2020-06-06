## Prepare 
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first	   
- **Please pay attention to the marks on each line of the config files, and modify them as requested**    	
## Build Environment	
Debian 9 && Ubuntu 16~18
## Content 
- install basic tools   
```bash
apt update && apt install -y wget
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
acme.sh --installcert -d your_domain.com --fullchain-file /etc/trojan-go/server.crt --key-file /etc/trojan-go/server.key --ecc
```
- install Docker && Trojan    
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan-go
vim /etc/trojan-go/config.json
```
> config a ：DO NOT Need to use CDN  
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
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #modify to your domain
        "fallback_port": 3000 
    }
}
```
>> config b ：Need to use CDN  
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
        "cert": "/etc/trojan-go/server.crt",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #modify to your domain
        "fallback_port": 3000 
    },
      "websocket": {
        "enabled": true,
        "path": "/your_path",    #modify a path
        "hostname": "your_domain.com",   #modify to your domain
        "obfuscation_password": "password1",   #modify to another password
        "double_tls": false
    }
}
```
- install Nginx  
```bash
apt update && apt install -y nginx
```
- modify config files（please modify **your_domain.com** to your domain）
```bash
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
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
nginx -s reload
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ： 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## For new version Update:
```bash
docker stop trojan-go
docker rm trojan-go
docker rmi teddysun/trojan-go
docker pull teddysun/trojan-go
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
```
## Client 
Windows 7.0+ ：https://github.com/Trojan-Qt5/Trojan-Qt5/releases   
Android 6.0+ ：[Download](https://github.com/charlieethan/firewall-proxy/releases/download/V0.5.1m/Igniter-Go-v0.5.1.apk)			

**Recommend config on Mobile ：**		
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
**If you use config b, please modify client config by yourself**
