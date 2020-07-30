# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**    
# 搭建環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統     

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 內容
- 安裝基礎工具  
```bash
apt update && apt -y install libnss3 wget unzip
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安裝 Caddy && Naiveproxy  
```bash
wget https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1/caddy
chmod +x caddy && setcap cap_net_bind_service=+ep ./caddy
```
- 下載網站範本    
**我準備了10個偽裝網站範本，這裏只是一個示例，你可以將 `1.zip` 改為 `2~10.zip`**   
```bash
mkdir -p /var/www/html && cd /var/www/html
wget https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip && unzip 1.zip 
```
- 修改配置
```bash
cd && cat > caddy.json <<EOF
{
  "apps": {
    "http": {
      "servers": {
        "srv0": {
          "listen": [":443"],
          "routes": [{
            "handle": [{
              "handler": "forward_proxy",
              "hide_ip": true,
              "hide_via": true,
              "auth_user": "user_name",     #改為你的用戶名
              "auth_pass": "your_password",     #改為你的密碼
              "probe_resistance": {"domain": "unsplash.com:443"}
            }]
          }, {
            "match": [{"host": ["your_domain.com"]}],    #改為你的功能變數名稱
            "handle": [{
              "handler": "file_server",
              "root": "/var/www/html"
            }],
            "terminal": true
          }],
          "tls_connection_policies": [{
            "match": {"sni": ["your_domain.com"]}   #改為你的功能變數名稱
          }]
        }
      }
    },
    "tls": {
      "automation": {
        "policies": [{
          "subjects": ["your_domain.com"],   #改為你的功能變數名稱
          "issuer": {
            "module": "acme",
            "email": "your@email.com"  #改為你的郵箱地址
          }
        }]
      }
    }
  }
}
EOF
```
- 啟動服務  
```bash
nohup ./caddy run --config caddy.json >caddy.log 2<&1 &
```
- 開啟 BBR 加速
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 客戶端
**目前沒有移動客戶端支持，你只能在電腦上使用！**      

Windows && Linux && MacOS : [Qv2ray 下載](https://github.com/Qv2ray/Qv2ray/releases)       

支持 Naiveproxy 的插件 : [插件下載](https://github.com/Qv2ray/QvPlugin-NaiveProxy/releases)    

插件的用法 : [文檔](https://qv2ray.net/plugins/usage.html) 
