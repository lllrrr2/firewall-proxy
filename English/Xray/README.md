# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open **CDN service** at first
- **Please pay attention to the marks on each line of the config files, and modify them as requested**
# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
# Content
- install basic tools
```bash
apt update && apt install -y socat wget   
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
mkdir -p /etc/nginx/conf.d /etc/xray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/xray/server.pem --key-file /etc/xray/server.key --ecc
```
- Install Docker && Nginx && V2ray
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull ghcr.io/charlieethan/xray
docker pull containrrr/watchtower
```
- modify config file 
```bash
cat > /etc/xray/config.json <<EOF
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30866",  // modify UUID,you can generate one from https://www.uuidgenerator.net/
            "flow": "xtls-rprx-direct",
            "level": 0
          }
        ],
        "decryption": "none",
        "fallbacks":[
        {
          "dest": 80
        },
        {
          "path": "/your_path",  // modify path
          "dest": 1000,
          "xver": 1
        }
       ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "xtls",
        "tcpSettings": {
        "type": "none"
        },
        "xtlsSettings": {
          "serverName": "your_domain.com",  // modify "your_domain.com" to your domain
          "allowInsecure": false,
          "alpn": [
          "http/1.1"
          ],
          "certificates": [
            {
              "certificateFile": "/etc/xray/server.pem",
              "keyFile": "/etc/xray/server.key"
            }
          ]
       }
     }
   },
   {
      "port": 1000,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30866",    // modify UUID,keep it as same as above
            "level": 0
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
        "acceptProxyProtocol": true,
        "path": "/your_path"   // modify path
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
EOF
```
- modify config files of Nginx 
```bash
cat > /etc/nginx/conf.d/default.conf <<EOF
server {
    listen 127.0.0.1:80;
    server_name your_domain.com;  // modify "your_domain.com" to your domain
    location / {
        proxy_pass https://proxy.com;  // modify to any website URL you want to disguise
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
EOF
```
- Start Service  
```bash 
docker run --network host --name xray -v /etc/xray:/etc/xray --restart=always -d ghcr.io/charlieethan/xray
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
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
Android 6.0+: [Download](https://github.com/2dust/v2rayNG/releases) 

Windows && Linux && MacOS : [Qxray Download](https://github.com/Qv2ray/Qv2ray/releases)    

The Usage of Qxray (Chinese Only) : [Usage](https://qv2ray.net/getting-started/step2.html)  