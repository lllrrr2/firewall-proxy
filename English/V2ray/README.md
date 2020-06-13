# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open **CDN service** at first
- **Please pay attention to the marks on each line of the config files, and modify them as requested**
# Build Environment
Debian 9/10 && Ubuntu 16/18/20
# Content
- install basic tools
```bash
apt update && apt install -y socat wget git vim     
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
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
acme.sh --installcert -d your_domain.com --fullchain-file /etc/nginx/conf.d/server.crt --key-file /etc/nginx/conf.d/server.key --ecc
```
- Install Docker && && Nginx V2ray
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/v2ray
```
- modify config file 
```bash
mkdir /etc/v2ray
vim /etc/v2ray/config.json
```
- copy your config  
```bash
{
  "inbounds": [
    {
      "port": 10000,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",    # modify UUID,you can generate one from https://www.uuidgenerator.net/
            "alterId": 60     #modify alterID,please keep the number between 0~300
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/your_path"   #modify path
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
- modify config files of Nginx 
```bash
vim /etc/nginx/conf.d/default.conf
```
- copy your config  
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/nginx/conf.d/server.crt;  
    ssl_certificate_key   /etc/nginx/conf.d/server.key;
    ssl_protocols         TLSv1.2 TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
    
    server_name  your_domain.com;    #modify "your_domain.com" to your domain
    location / {
        proxy_pass https://proxy.com;     #modify to any website URL you want to disguise
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;
     }

    location /your_path {       ##modify the path you modified above 
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;    #modify to your server IP address
    return 301 https://your_domain.com$request_uri;    #modify "your_domain.com" to your domain
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
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# For new version Update:
- Update v2ray    
```bash
docker stop v2ray
docker rm v2ray
docker rmi teddysun/v2ray
docker pull teddysun/v2ray
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
```
- Update nginx   
```bash
docker stop nginx
docker rm nginx
docker rmi nginx
docker pull nginx
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
```
# Client
Windows 7+: [Download](https://github.com/2dust/v2rayN/releases)    
Configuraton is like below:   


![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/1.jpg)


Android 6.0+: [Download](https://github.com/2dust/v2rayNG/releases) 
