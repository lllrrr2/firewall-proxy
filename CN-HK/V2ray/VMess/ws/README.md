# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**   
# 配置環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統      

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 配置內容
- 安裝基礎工具  
```bash
apt update && apt install -y socat wget  
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
mkdir -p /etc/nginx/conf.d /etc/v2ray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/nginx/conf.d/server.pem --key-file /etc/nginx/conf.d/server.key --ecc
```
- 安裝 Docker && Nginx && V2ray  
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/v2ray
docker pull containrrr/watchtower
```
- 編輯 v2ray 配置 
```bash
cat > /etc/v2ray/config.json <<EOF
{
  "inbounds": [
    {
      "port": 10000,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",    // 更改id
            "alterId": 0
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
        "path": "/your_path"   // 更改路徑
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
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/nginx/conf.d/server.pem;  
    ssl_certificate_key   /etc/nginx/conf.d/server.key;
    ssl_protocols         TLSv1.2 TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
    
    server_name  your_domain.com;    // 改為你的功能變數名稱
    location / {
        proxy_pass https://proxy.com;     // 改為你想偽裝的網站
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;
     }

    location /your_path {       // 更改路徑
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
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 啟動服務  
```bash 
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
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
Android系統: [點擊下載](https://github.com/2dust/v2rayNG/releases)    

Windows && Linux && MacOS : [Qv2ray 下載](https://github.com/Qv2ray/Qv2ray/releases)   

Qv2ray 用法 : [文檔](https://qv2ray.net/getting-started/step2.html) 
