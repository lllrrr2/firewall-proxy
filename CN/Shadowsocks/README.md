# 配置环境
纯净的 Debian 9 && Ubuntu 16~18 系统
# 配置内容
- 升级系统并安装 Docker
```
apt-get update && apt-get install -y wget vim
wget -qO- get.docker.com | bash
```
- 安装shadowsocks
```
docker pull teddysun/shadowsocks-libev
docker pull containrrr/watchtower
mkdir -p /etc/shadowsocks-libev
```
- 写入配置
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":443,
    "password":"password0", #修改密码
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
- 启动服务
```
docker run --network host --name ss-libev  -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 开启 BBR 加速：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 升级软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

# 客户端
- 安卓系统 ： [Shadowsocks下载](https://github.com/shadowsocks/shadowsocks-android/releases) | [obfs插件下载](https://github.com/shadowsocks/v2ray-plugin-android/releases)    
- Windows系统 ：[点击下载](https://github.com/shadowsocks/shadowsocks-windows/releases) | [obfs插件下载](https://github.com/shadowsocks/v2ray-plugin/releases)    
Windows系统 配置如下：  

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/ss.jpg)