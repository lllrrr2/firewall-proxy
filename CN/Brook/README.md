# 搭建环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统			

软件 : Debian 9/10 && Ubuntu 16/18/20
# 内容
- 升级系统
```bash
apt update && apt install -y wget vim curl
```
- 安装 Brook
```bash
curl https://raw.githubusercontent.com/txthinking/nami/master/install.sh | bash && sleep 6 && exec -l $SHELL
nami install github.com/txthinking/brook
```
- 启动服务
将 **your_password** 改为你的密码		
```bash
nohup brook server -l 0.0.0.0:443 -p your_password >brook.log 2<&1 &
```
- 开启 BBR 加速
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新软件
```bash
nami upgrade github.com/txthinking/brook
```
# 客户端
- Android 6.0+ && Windows 7+: https://github.com/txthinking/brook/releases

配置示例：		

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/3.jpg)