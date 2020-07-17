# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20
# Config
- update system
```bash
apt update && apt install -y wget vim curl
```
- install Brook
```bash
curl https://raw.githubusercontent.com/txthinking/nami/master/install.sh | bash && sleep 6 && exec -l $SHELL
nami install github.com/txthinking/brook
```
- Start Service
modify **your_password** to your password
```bash
nohup brook server -l 0.0.0.0:443 -p your_password >brook.log 2<&1 &
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# For new version Update:
```bash
nami upgrade github.com/txthinking/brook
```
# Client
- Android 6.0+ && Windows 7+: https://github.com/txthinking/brook/releases          

A example of the config:

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/3.jpg)
