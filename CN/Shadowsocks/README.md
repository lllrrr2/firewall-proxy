# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**    
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统			

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容
- 安装基本工具
```
apt update && apt -y install socat wget vim     
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装脚本
```
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申请证书 （请将 **yourdomain.com** 改为你的域名）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/nginx/conf.d/server.pem --key-file /etc/nginx/conf.d/server.key --ecc
```
- 安装 Docker && Nginx && Shadowsocks
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/shadowsocks-libev
docker pull containrrr/watchtower
```
- 将以下内容粘贴 
```bash
mkdir /etc/shadowsocks-libev
cat > /etc/shadowsocks-libev/config.json <<EOF
{
"server":"127.0.0.1",
"server_port":2000,
"password":"your_password",   #修改密码
"timeout":300,
"method":"aes-256-gcm",
"fast_open":false,
"nameserver":"1.0.0.1",
"mode":"tcp_and_udp",
"plugin":"v2ray-plugin",
"plugin_opts":"server;path=/your_path"   #修改路径
}
EOF
```
- 修改 Nginx 配置
```bash
vim /etc/nginx/conf.d/default.conf
```
- 将以下内容粘贴
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/nginx/conf.d/server.pem;  
    ssl_certificate_key   /etc/nginx/conf.d/server.key;
    ssl_protocols         TLSv1.2 TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
    
    server_name  your_domain.com;    #改为你的域名
    location / {
        proxy_pass https://proxy.com;     #改为你想伪装的网址
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;
     }

    location /your_path {       #改为你在上面修改的路径
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
    server_name ip.ip.ip.ip;    #改为你服务器的 IP 地址
    return 301 https://your_domain.com$request_uri;    #改为你的域名
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
docker run --network host --name ss -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
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
- 安卓系統 ： [Shadowsocks下载](https://github.com/shadowsocks/shadowsocks-android/releases) | [V2ray Plugin 插件下载](https://github.com/teddysun/v2ray-plugin-android/releases)    
- Windows系統 ：[点击下载](https://github.com/shadowsocks/shadowsocks-windows/releases) | [V2ray Plugin 插件下载](https://github.com/teddysun/v2ray-plugin/releases)    
Windows系統 配置如下：  

`Plugin Program : v2ray-plugin`		
`Plugin Options : tls;host=your_domain.com;path=/your_path`		