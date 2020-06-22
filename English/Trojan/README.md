# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first        
- **Please pay attention to the marks on each line of the config files, and modify them as requested**      
# Build Environment 
Debian 9/10 && Ubuntu 16/18/20        
# Content
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
- request SSL certificate (modify **yourdomain.com** to your domain）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/server.crt --key-file /etc/trojan/server.key --ecc
```
- install Docker && Nginx && Trojan
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan
docker pull containrrr/watchtower
```
- modify config files of trojan
```bash
vim /etc/trojan/config.json
```
Paste config files below
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #modify to your password
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/trojan/server.crt",
        "key": "/etc/trojan/server.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```
- modify config files of nginx
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
- paste config below       
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;    #modify "your_domain.com" to your domain
    location / {
        proxy_pass proxy.com;         #modify to any website URL you want to disguise
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k; 
    }

}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;      #modify to your server IP address
    return 301 https://yourdomain.com$request_uri;   #modify "your_domain.com" to your domain
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- Start Service     
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan -v /etc/trojan:/etc/trojan --restart=always -d teddysun/trojan
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
# For new version Update:
Using this config method,**watchtower** can auto detect and update your software,You don't need to update manually any more

# Client
Android 6.0+ ：[Download](https://github.com/trojan-gfw/igniter/releases)                  

Windows 7.0+ ：[Download](https://github.com/Trojan-Qt5/Trojan-Qt5/releases)   
