# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**   
# 配置環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統      

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 配置內容
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
mkdir /etc/caddy
acme.sh --installcert -d your_domain.com --fullchain-file /etc/caddy/server.pem --key-file /etc/caddy/server.key --ecc
```
- 安裝 Docker && V2ray && Caddy
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/v2ray
docker pull containrrr/watchtower
wget https://github.com/charlieethan/firewall-proxy/releases/download/1.0.5/caddy
chmod +x caddy && mv caddy /usr/local/bin
```
- 編輯 v2ray 配置 
```bash
mkdir /etc/v2ray
cp /etc/caddy/server.pem /etc/v2ray
cp /etc/caddy/server.key /etc/v2ray
vim /etc/v2ray/config.json
```
- 複製配置  
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
          "path": "/your_path",  #更改路徑
          "host": [
            "your_domain.com"    #改為你的功能變數名稱
          ]
        },
        "tlsSettings": {
          "serverName": "your_domain.com",  #改為你的功能變數名稱
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
- 複製配置  
```bash
http://your_domain.com {  #改為你的功能變數名稱
    redir https://your_domain.com{url}  #改為你的功能變數名稱
}

https://your_domain.com {  #改為你的功能變數名稱
    log stdout
    errors stderr
    tls /etc/caddy/server.pem /etc/caddy/server.key
    proxy / https://proxy.com   #改為你想偽裝的網址
    proxy /mypath https://localhost:2000 {
    insecure_skip_verify
    header_upstream Host "your_domain.com"  #改為你的功能變數名稱
    header_upstream X-Forwarded-Proto "https"
}
```
- 啟動服務  
```bash 
nohup /usr/local/bin/caddy -conf /etc/caddy/Caddyfile >caddy.log 2<&1 &
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
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

![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)

**yourdomain**填你的功能變數名稱 ，**id**和**alterId**填你上面設置的  
**Path**填上面設置的路徑 ，其餘部分照抄即可
# 客戶端
Windows系統: [點擊下載](https://github.com/2dust/v2rayN/releases)

Android系統: [點擊下載](https://github.com/2dust/v2rayNG/releases) 
# Q && A
- Q : 為什麼不用 Nginx 或 Apache？     
A : 因為 Nginx 不支持反向代理 HTTP2 流量, Apache 安裝配置很複雜，暫時不予考慮      
- Q : 為什麼不用 Caddy2？     
A : 因為 Caddy2 目前仍在開發階段，配置複雜且官方文檔很模糊，等待官方文檔豐富之後再切換使用
