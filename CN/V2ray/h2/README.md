# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**   
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统      

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容
- 安装基础工具  
```bash
apt update && apt install -y socat wget git vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装证书生成脚本  
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 安装证书  (**your_domain.com** 改为你的域名）
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/caddy
acme.sh --installcert -d your_domain.com --fullchain-file /etc/caddy/server.pem --key-file /etc/caddy/server.key --ecc
```
- 安装 Docker && V2ray && Caddy
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/v2ray
docker pull containrrr/watchtower
wget https://github.com/charlieethan/firewall-proxy/releases/download/1.0.5/caddy
chmod +x caddy && mv caddy /usr/local/bin
```
- 编辑 v2ray 配置 
```bash
mkdir /etc/v2ray
cp /etc/caddy/server.pem /etc/v2ray
cp /etc/caddy/server.key /etc/v2ray
vim /etc/v2ray/config.json
```
- 复制配置  
```bash
{
  "inbounds": [
    {
      "port": "2000",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "09c948f9-044d-4956-e056-89d39cb3db9d64",  #更改id
            "alterId": 64  #更改alterID
          }
        ]
      },
      "streamSettings": {
        "network": "h2",
        "security": "tls",
        "httpSettings": {
          "path": "/your_path",  #更改路径
          "host": [
            "your_domain.com"    #改为你的域名
          ]
        },
        "tlsSettings": {
          "serverName": "your_domain.com",  #改为你的域名
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
- 修改 Caddy 配置 
```bash
vim /etc/caddy/Caddyfile
```
- 复制配置  
```bash
http://your_domain.com {  #改为你的域名
    redir https://your_domain.com{url}  #改为你的域名
}

https://your_domain.com {  #改为你的域名
    log stdout
    errors stderr
    tls /etc/caddy/server.pem /etc/caddy/server.key
    proxy / https://proxy.com   #改为你想伪装的网址
    proxy /mypath https://localhost:2000 {
    insecure_skip_verify
    header_upstream Host "your_domain.com"  #改为你的域名
    header_upstream X-Forwarded-Proto "https"
}
```
- 启动服务  
```bash 
nohup /usr/local/bin/caddy -conf /etc/caddy/Caddyfile >caddy.log 2<&1 &
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

# 客户端

![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)

**yourdomain**填你的域名 ，**id**和**alterId**填你上面设置的  
**Path**填上面设置的路径 ，其余部分照抄即可
# 客户端
Windows系统: [点击下载](https://github.com/2dust/v2rayN/releases)

Android系统: [点击下载](https://github.com/2dust/v2rayNG/releases) 
# Q && A
- Q : 为什么不用 Nginx 或 Apache？     
A : 因为 Nginx 不支持反向代理 HTTP2 流量, Apache 安装配置很复杂，暂时不予考虑      
- Q : 为什么不用 Caddy2？     
A : 因为 Caddy2 目前仍在开发阶段，配置复杂且官方文档很模糊，等待官方文档丰富之后再切换使用
