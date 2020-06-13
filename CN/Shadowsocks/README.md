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
mkdir -p /etc/shadowsocks-libev
```
- 写入配置
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":9000,     #修改端口
    "password":"password0", #修改密码
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open":false,
    "nameserver":"8.8.8.8",
    "mode":"tcp_and_udp",
    "plugin":"obfs-server",
    "plugin_opts":"obfs=tls"
}
EOF
```
- 启动服务 （请将`9000`改为你自己设置的端口）
```
docker run -d -p 9000:9000 -p 9000:9000/udp --name ss-libev --restart=always -v /etc/shadowsocks-libev:/etc/shadowsocks-libev teddysun/shadowsocks-libev
```
- 开启 BBR 加速：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 升级 Shadowsocks 内核
```bash
docker stop ss-libev
docker rm ss-libev
docker rmi teddysun/shadowsocks-libev
docker pull teddysun/shadowsocks-libev
docker run --network host --name ss-libev -v /etc/shadowsocks-libev:/etc/shadowsocks-libev --restart=always -d teddysun/shadowsocks-libev
```
# 客户端
- 安卓系统 ： [Shadowsocks下载](https://github.com/shadowsocks/shadowsocks-android/releases) | [obfs插件下载](https://github.com/shadowsocks/simple-obfs-android/releases)    
- Windows系统 ：[点击下载](https://github.com/shadowsocks/shadowsocks-windows/releases) | [obfs插件下载](https://github.com/shadowsocks/simple-obfs/releases)    
Windows系统 配置如下：  

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)
```
obfs-local
obfs=tls;www.github.com
```