# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first        
- **Please pay attention to the marks on each line of the config files, and modify them as requested**      
# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
# Config
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
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/nginx/conf.d/server.pem --key-file /etc/nginx/conf.d/server.key --ecc
```
- install Docker && Nginx && Shadowsocks
```
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/shadowsocks-libev
docker pull containrrr/watchtower
```
- Paste config files
```
mkdir /etc/shadowsocks-libev
cat > /etc/shadowsocks-libev/config.json <<EOF
{
"server":"127.0.0.1",
"server_port":2000,
"password":"your_password",   #modify to your password
"timeout":300,
"method":"aes-256-gcm",
"fast_open":false,
"nameserver":"1.0.0.1",
"mode":"tcp_and_udp",
"plugin":"v2ray-plugin",
"plugin_opts":"server;path=/your_path"   #modify to your path
}
EOF
```
- modify config files of Nginx 
```bash
vim /etc/nginx/conf.d/default.conf
```
- copy your config  
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/nginx/conf.d/server.pem;  
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
        proxy_pass http://127.0.0.1:2000;
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
```
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name ss -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# For new version Update:
Using this config method,**watchtower** can auto detect and update your software,You don't need to update manually any more
# Client
- Android 6.0+： [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-android/releases) | [Plugin Download](https://github.com/teddysun/v2ray-plugin-android/releases)    
- Windows 7+: [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-windows/releases)      
[Plugin Download](https://github.com/teddysun/v2ray-plugin/releases)    
**You need to put Shadowsocks and v2ray plugin into one folder**		
The config on Windows:

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/4.jpg)

`Plugin Program : v2ray-plugin`		
`Plugin Options : tls;host=your_domain.com;path=/your_path`		