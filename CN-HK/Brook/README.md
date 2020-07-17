# 搭建環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統			
軟體 : Debian 9/10 && Ubuntu 16/18/20
# 內容
- 升級系統
```bash
apt update && apt install -y wget vim curl
```
- 安裝 Brook
```bash
curl https://raw.githubusercontent.com/txthinking/nami/master/install.sh | bash && sleep 6 && exec -l $SHELL
nami install github.com/txthinking/brook
```
- 啟動服務
將 **your_password** 改為你的密碼		
```bash
nohup brook server -l 0.0.0.0:443 -p your_password >brook.log 2<&1 &
```
- 開啟 BBR 加速
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新軟體
```bash
nami upgrade github.com/txthinking/brook
```
# 客戶端
- Android 6.0+ && Windows 7+: https://github.com/txthinking/brook/releases

配置示例：		

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/3.jpg)