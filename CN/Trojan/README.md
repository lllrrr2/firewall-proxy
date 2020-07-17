# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**    
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统			
软件 : Debian 9/10 && Ubuntu 16/18/20
# 开始部署
- 安装基本工具
```bash
apt update && apt -y install socat wget vim     
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装脚本
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申请证书 （请将 **yourdomain.com** 改为你的域名）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/server.crt --key-file /etc/trojan/server.key --ecc
```
- 安装 Docker && Nginx && Trojan
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan
docker pull containrrr/watchtower
```
- 修改 Trojan 配置
```bash
vim /etc/trojan/config.json
```
- 将以下内容粘贴 
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #改为你的密码
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
- 修改 Nginx 配置
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
- 将以下内容粘贴
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;    #修改为你的域名
    location / {
        proxy_pass proxy.com;         #修改为你想伪装的网站域名，例如 https://unsplash.com/
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k; 
    }

}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;      #修改为你服务器的 IP地址
    return 301 https://yourdomain.com$request_uri;   #修改为你的域名
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
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan -v /etc/trojan:/etc/trojan --restart=always -d teddysun/trojan
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 启动BBR加速
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
# 更新软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

# 客户端
安卓系统 ：[点击下载](https://github.com/trojan-gfw/igniter/releases)          
> 配置如下： **地址**填你的域名，**端口**填 443 ，**密码**填你刚才设置的密码，其他选项无需更改        

Windows系统 ：[点击下载](https://github.com/Trojan-Qt5/Trojan-Qt5/releases)   
> 项目地址 & 使用说明 ：https://github.com/TheWanderingCoel/Trojan-Qt5
