 # 配置環境
Debian 9 && Ubuntu 16~18
# 配置內容
- 升級系統並安裝 Docker
```
apt-get update && apt-get install -y wget vim
wget -qO- get.docker.com | bash
```
- 安裝shadowsocks
```
docker pull teddysun/shadowsocks-libev
docker pull containrrr/watchtower
mkdir -p /etc/shadowsocks-libev
```
- 寫入配置
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":443,
    "password":"password0", #修改密碼
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open":false,
    "nameserver":"1.1.1.1",
    "mode":"tcp_and_udp",
    "plugin":"v2ray-plugin",
    "plugin_opts":"server"
}
EOF
```
- 啟動服務
```
docker run --network host --name ss-libev  -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 開啟 BBR 加速：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 升級軟體
使用這種配置方式後，**watchtower**會自動監測並更新軟體，你無需手動更新

# 客戶端
- 安卓系統 ： [Shadowsocks下載](https://github.com/shadowsocks/shadowsocks-android/releases) | [obfs插件下載](https://github.com/shadowsocks/v2ray-plugin-android/releases)    
- Windows系統 ：[點擊下載](https://github.com/shadowsocks/shadowsocks-windows/releases) | [obfs插件下載](https://github.com/shadowsocks/v2ray-plugin/releases)    
Windows系統 配置如下：  

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/ss.jpg)