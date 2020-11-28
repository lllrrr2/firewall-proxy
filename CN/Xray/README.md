# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**   
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统      

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容
- 安装基础工具  
```bash
apt update && apt install -y socat wget    
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
mkdir -p /etc/nginx/conf.d /etc/xray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/xray/server.pem --key-file /etc/xray/server.key --ecc
```
- 安装 Docker && Nginx && V2ray 
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull ghcr.io/charlieethan/xray
docker pull containrrr/watchtower
```
- 编辑 Xray 配置 
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
            "id": "b831381d-6324-4d53-ad4f-8cda48b30866",  // 更改id
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
          "path": "/your_path",  // 更改路径
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
          "serverName": "your_domain.com",  // 改为你的域名
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
            "id": "b831381d-6324-4d53-ad4f-8cda48b30866",    // 更改id
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
        "path": "/your_path"   // 更改路径
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
- 修改 Nginx 配置 
```bash
nano /etc/nginx/conf.d/default.conf
```
- 复制配置
```bash
server {
    listen 127.0.0.1:80;
    server_name your_domain.com;  # 改为你的域名
    location / {
        proxy_pass https://proxy.com;  # 改为你想伪装的网址
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
```
- 启动服务 
```bash
docker run --network host --name xray -v /etc/xray:/etc/xray --restart=always -d ghcr.io/charlieethan/xray
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
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

# 说明
在使用这种配置方式后，同一个 UUID 可以同时使用 TCP+TLS 和 Websocket 模式下的 VLess 协议。这意味着如果你的 IP 未被墙，你可以在客户端使用 TCP+TLS 的模式最大程度跑满服务器的带宽；而如果服务器 IP 被墙，你不用更换任何服务器端的配置文件，只需要开启 CDN 并在客户端直接使用 Websocket 模式连接即可，十分的方便快捷

# 客户端
Android系统: [点击下载](https://github.com/2dust/v2rayNG/releases)    

Windows && Linux && MacOS : [Qv2ray 下载](https://github.com/Qv2ray/Qv2ray/releases)   

Qv2ray 用法 : [文档](https://qv2ray.net/getting-started/step2.html) 