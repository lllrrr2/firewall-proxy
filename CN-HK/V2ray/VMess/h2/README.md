# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**   
# 配置環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統      

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 配置內容
- 安裝基礎工具  
```bash
apt update && apt install -y wget unzip vim    
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安裝 tls-shunt-proxy 
```bash
mkdir -p /etc/tsp /etc/v2ray /var/www/html
wget -P /usr/local/bin https://github.com/charlieethan/firewall-proxy/releases/download/0.6.1/tsp && chmod +x /usr/local/bin/tsp
```
- 安裝 Docker && V2ray
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/v2ray
docker pull containrrr/watchtower
```
- 下載網站範本    
**我準備了50個偽裝網站範本，這裏只是一個示例，你可以將 `1.zip` 改為 `2~50.zip`**   
```bash
wget -P /var/www/html https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip && unzip /var/www/html/1.zip -d /var/www/html
```
- 編輯 v2ray 配置 
```bash
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
            "id": "09c948f9-044d-4956-e056-89d39cb3db9d64", #更改id
            "alterId": 0  #請不要修改，以啓用 VMess AEAD，抵抗主動檢測
          }
        ]
      },
      "streamSettings": {
        "network": "h2",
        "security": "none",
        "httpSettings": {
          "path": "/your_path",  #更改路徑
          "host": [
            "your_domain.com"  #改為你的功能變數名稱
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
- 修改 tls-shunt-proxy 配置
```bash
vim /etc/tsp/config.yaml
```
- 複製配置 
```bash
listen: 0.0.0.0:443
redirecthttps: 0.0.0.0:80
inboundbuffersize: 4
outboundbuffersize: 32

vhosts:
  - name: your_domain.com  #改為你的功能變數名稱
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
- 啟動服務  
```bash 
nohup tsp -config /etc/tsp/config.yaml >/etc/tsp/tsp.log 2<&1 &
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
Android系統: [點擊下載](https://github.com/2dust/v2rayNG/releases)    

Windows && Linux && MacOS : [Qv2ray 下載](https://github.com/Qv2ray/Qv2ray/releases)   

Qv2ray 用法 : [文檔](https://qv2ray.net/getting-started/step2.html) 