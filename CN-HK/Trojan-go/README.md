# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**    
# 配置環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統			

軟體 : Debian 9/10 && Ubuntu 16/18/20
## 配置內容 
- 升級並安裝必要軟體   
```bash
apt update && apt -y install socat wget vim
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安裝腳本 
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申請證書 （請將 **your_domain.com** 改為你的功能變數名稱）  
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/trojan-go
acme.sh --installcert -d your_domain.com --fullchain-file /etc/trojan-go/server.pem --key-file /etc/trojan-go/server.key --ecc
```
- 安裝 Docker && Nginx && Trojan     
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan-go
docker pull containrrr/watchtower
```
- 修改 Trojan-go 配置
```bash
vim /etc/trojan-go/config.json
```
<details>
<summary>配置1 ：不使用CDN</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改為你設定的密碼
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改為你的功能變數名稱
        "fallback_port": 3000 
    }
}
```
</details>

<details>
<summary>配置2 ：使用CDN，且你信任你的CDN提供商</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改為你設定的密碼
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改為你的功能變數名稱
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改為你設定的路徑
    "host": "your_domain.com"   #修改為你的功能變數名稱
    }
}
```
</details>  

<details>
<summary>配置3 ：使用CDN，但你不信任你的CDN提供商（例如國內廠商的CDN）</summary>

```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改為你設定的密碼
    ],
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "/etc/trojan-go/server.pem",
        "key": "/etc/trojan-go/server.key",
	"sni": "your_domain.com",    #修改為你的功能變數名稱
        "fallback_port": 3000 
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path",  #修改為你設定的路徑
    "host": "your_domain.com"   #修改為你的功能變數名稱
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1"   #修改為另一個密碼，請勿與上方密碼一致
  }
}
```
</details>

- 修改 Nginx 配置  
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
**複製下列配置**  
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name your_domain.com;   #修改為你的功能變數名稱
    location / {
        proxy_pass https://your_proxy.com;   #修改為你想偽裝的網站功能變數名稱，例如 https://unsplash.com/  
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k;  
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;  #修改為你伺服器的 IP地址
    return 301 https://your_domain.com$request_uri;   #修改為你的功能變數名稱
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
server {
	listen 127.0.0.1:3000;
	server_name _;
	return 400;
}
```
- 啟動服務  
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan-go -v /etc/trojan-go:/etc/trojan-go --restart=always -d teddysun/trojan-go
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 開啟 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 更新軟體
使用這種配置方式後，**watchtower**會自動監測並更新軟體，你無需手動更新

## 客戶端的使用 
PC平臺 ：[點擊下載](https://github.com/charlieethan/firewall-proxy/releases/tag/1.4.0)   		
安卓平臺 ：[點擊下載](https://github.com/charlieethan/firewall-proxy/releases/download/V0.7.7/Igniter-Go-v0.7.7.apk)			

**移動版推薦配置如下 ：**		
<details>
<summary>對應配置1</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    }
}
```
</details>

<details>
<summary>對應配置2</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    }
}
```
</details>

<details>
<summary>對應配置3</summary>

```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
	"verify_hostname": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "firefox"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
    "enabled": true,
    "path": "/your_path", 
    "hostname": "your_domain.com"  
    },
    "shadowsocks": {
    "enabled": true,
    "method": "AES-128-GCM",
    "password": "password1" 
  }
}
```
</details>
