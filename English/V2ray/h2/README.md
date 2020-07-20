# Prepare
- You need correctly appoint your Domain to your Server IP, and this method Do Not support **CDN service**    
- **Please pay attention to the marks on each line of the config files, and modify them as requested**
# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
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
mkdir /etc/caddy
acme.sh --installcert -d your_domain.com --fullchain-file /etc/caddy/server.pem --key-file /etc/caddy/server.key --ecc
```
- Install Docker && V2ray && Caddy
```bash
wget -qO- get.docker.com | bash
wget -qO- https://getcaddy.com | bash -s personal
docker pull teddysun/v2ray
docker pull containrrr/watchtower
```
- modify config file 
```bash
mkdir /etc/v2ray
cp /etc/caddy/server.pem /etc/v2ray
cp /etc/caddy/server.key /etc/v2ray
vim /etc/v2ray/config.json
```
- copy your config  
```bash
{
  "inbounds": [
    {
      "port": "2000",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "09c948f9-044d-4956-e056-89d39cb3db9d64",  # modify UUID,you can generate one from https://www.uuidgenerator.net/
            "alterId": 64  #modify alterID,please keep the number between 0~300
          }
        ]
      },
      "streamSettings": {
        "network": "h2",
        "security": "tls",
        "httpSettings": {
          "path": "/your_path",  #modify path
          "host": [
            "your_domain.com"    #modify "your_domain.com" to your domain
          ]
        },
        "tlsSettings": {
          "serverName": "your_domain.com",  #modify "your_domain.com" to your domain
          "certificates": [
            {
              "certificateFile": "/etc/v2ray/server.pem",
              "keyFile": "/etc/v2ray/server.key"
            }
          ]
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
- modify config files of Caddy
```bash
vim /etc/caddy/Caddyfile
```
- copy your config  
```bash
http://your_domain.com {  #modify "your_domain.com" to your domain
    redir https://your_domain.com{url}  #modify "your_domain.com" to your domain
}

https://your_domain.com {  #modify "your_domain.com" to your domain
    log stdout
    errors stderr
    tls /etc/caddy/server.pem /etc/caddy/server.key
    proxy / https://proxy.com   #modify to any website URL you want to disguise
    proxy /mypath https://localhost:2000 {
    insecure_skip_verify
    header_upstream Host "your_domain.com"  #modify "your_domain.com" to your domain
    header_upstream X-Forwarded-Proto "https"
}
```
- Start Service  
```bash 
nohup /usr/local/bin/caddy -conf /etc/caddy/Caddyfile >caddy.log 2<&1 &
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# For new version Update:
Using this config method,**watchtower** can auto detect and update your software,You don't need to update manually any more

# Client
Windows 7+: [Download](https://github.com/2dust/v2rayN/releases)    
Configuraton is like below:   


![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)


Android 6.0+: [Download](https://github.com/2dust/v2rayNG/releases) 
# Q && A
- Q : Why not use Nginx or Apache?      
A : Because Nginx don't support reverse proxy for HTTP2, and Apache is too large to use it only for proxying your traffic    
- Q : Why not use Caddy2?     
A : Because Caddy2 is too complex to use, many things changed and less docs than Caddy1, I still need to wait until the official provide more detailed docs   