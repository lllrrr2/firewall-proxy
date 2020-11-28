# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open **CDN service** at first
- **Please pay attention to the marks on each line of the config files, and modify them as requested**
# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
# Content
- install basic tools
```bash
apt update && apt install -y wget unzip   
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- Install Docker && V2ray && tls-shunt-proxy
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/v2ray
docker pull charlieethan/tls-shunt-proxy
docker pull containrrr/watchtower
```
- get HTML Tamplates    
**I prepared 50 templates to use,this is an example to download one of them. You can modify `1.zip` to `2~50.zip`**   
```bash
mkdir -p /var/www/html /etc/v2ray /etc/tls-shunt-proxy
wget -P /var/www/html https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip && unzip /var/www/html/1.zip -d /var/www/html
```
- modify config file 
```bash
cat > /etc/v2ray/config.json <<EOF
{
  "inbounds": [
    {
      "port": "2000",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "09c948f9-044d-4956-e056-89d39cb3db9d64", // modify UUID,you can generate one from https://www.uuidgenerator.net/
            "alterId": 0
          }
        ]
      },
      "streamSettings": {
        "network": "h2",
        "security": "none",
        "httpSettings": {
          "path": "/your_path",  // modify path
          "host": [
            "your_domain.com"  // modify "your_domain.com" to your domain
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
EOF
```
- modify config files of tls-shunt-proxy
```bash
nano /etc/tls-shunt-proxy/config.yaml
```
- copy configuration files
```bash
listen: 0.0.0.0:443
redirecthttps: 0.0.0.0:80
inboundbuffersize: 4
outboundbuffersize: 32

vhosts:
  - name: your_domain.com  
    # modify "your_domain.com" to your domain
    tlsoffloading: true
    managedcert: true
    keytype: p256
    alpn: h2,http/1.1
    protocols: tls12,tls13

    http:
      paths:
        - path: "*"
          handler: proxyPass
          args: 127.0.0.1:2000

    default:
      handler: fileServer
      args: /var/www/html
```
- Start Service  
```bash 
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --network host --name tls-shunt-proxy -v /etc/tls-shunt-proxy:/etc/tls-shunt-proxy -v /var/www/html:/var/www/html --restart=always -d charlieethan/tls-shunt-proxy
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

Windows && Linux && MacOS : [Qv2ray Download](https://github.com/Qv2ray/Qv2ray/releases)    

The Usage of Qv2ray (Chinese Only) : [Usage](https://qv2ray.net/getting-started/step2.html) 