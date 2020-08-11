# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**    
# 搭建環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統			

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 內容
- 安裝基礎工具  
```bash
apt update && apt install -y socat wget git vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安裝證書生成腳本  
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 安裝證書  (**your_domain.com** 改為你的功能變數名稱）
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir -p /etc/nginx /etc/nginx/conf.d
acme.sh --installcert -d your_domain.com --fullchain-file /etc/nginx/conf.d/server.pem --key-file /etc/nginx/conf.d/server.key --ecc
```
- 安裝 Docker && Nginx && Brook  
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/brook
docker pull containrrr/watchtower
```
- 修改 Nginx 配置 
```bash
vim /etc/nginx/conf.d/default.conf
```
- 複製配置  
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/nginx/conf.d/server.pem;  
    ssl_certificate_key   /etc/nginx/conf.d/server.key;
    ssl_protocols         TLSv1.2 TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
   
    server_name  your_domain.com;    #改為你的功能變數名稱
    location / {
        proxy_pass https://proxy.com;     #改為你想偽裝的網址
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;
     }

    location /your_path {       #改為你在上面修改的路徑
        proxy_redirect off;
        proxy_pass http://127.0.0.1:1000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;    #改為你伺服器的 IP 地址
    return 301 https://your_domain.com$request_uri;    #改為你的功能變數名稱
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
  }
```
- 啟動服務  
```bash 
docker run --network host --name nginx -v /etc/nginx:/etc/nginx/conf.d --restart=always -d nginx
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
**修改 `/your_path` 為你的路徑,`your_password` 為你的密碼**
```bash 
docker run --network host --name brook -e "ARGS=wsserver --path /your_path -l :1000 -p your_password" --restart=always -d teddysun/brook
```
- 開啟 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新軟體
使用這種配置方式後，**watchtower**會自動監測並更新軟體，你無需手動更新

# 客戶端
- Android 6.0+ && Windows 7+: https://github.com/txthinking/brook/releases

配置示例：		

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/3.jpg)