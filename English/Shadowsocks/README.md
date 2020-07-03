# Build Environment
Debian 9 && Ubuntu 16~18
# Config
- update and install Docker
```
apt-get update && apt-get install -y wget vim
wget -qO- get.docker.com | bash
```
- install Shadowsocks
```
docker pull teddysun/shadowsocks-libev
docker pull containrrr/watchtower
mkdir -p /etc/shadowsocks-libev
```
- Paste config files
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":443,
    "password":"password0", # modify password
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
- Start Service
```
docker run --network host --name ss-libev  -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# For new version Update:
Using this config method,**watchtower** can auto detect and update your software,You don't need to update manually any more
# Client
- Android 6.0+： [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-android/releases) | [Plugin Download](https://github.com/teddysun/v2ray-plugin-android/releases/tag/v1.3.3)    
- Windows 7+: [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-windows/releases)      
[Plugin Download](https://github.com/teddysun/v2ray-plugin-android/releases/tag/v1.3.3)    
The config on Windows:

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/ss.jpg)
