# 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**   
# 配置环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统      

软件 : Debian 9/10 && Ubuntu 16/18/20
# 配置内容
- 安装基础工具  
```bash
apt update && apt install -y wget unzip vim    
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装 tls-shunt-proxy 
```bash
mkdir -p /etc/tsp /etc/v2ray /var/www/html
wget https://github.com/charlieethan/firewall-proxy/releases/download/0.6.0/tsp && chmod +x tsp
```
- 安装 Docker && V2ray
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/v2ray
docker pull containrrr/watchtower
```
- 下载网站模板    
**我准备了50个伪装网站模板，这里只是一个示例，你可以将 `1.zip` 改为 `2~50.zip`**   
```bash
cd /var/www/html && wget https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip
unzip 1.zip && rm -rf 1.zip
```
- 编辑 v2ray 配置 
```bash
vim /etc/v2ray/config.json
```
- 复制配置 
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
            "alterId": 0  #请不要修改，以启用 VMess AEAD，抵抗主动检测
          }
        ]
      },
      "streamSettings": {
        "network": "h2",
        "security": "none",
        "httpSettings": {
          "path": "/your_path",  #更改路径
          "host": [
            "your_domain.com"  #改为你的域名
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
- 复制配置 
```bash
listen: 0.0.0.0:443
redirecthttps: 0.0.0.0:80
inboundbuffersize: 4
outboundbuffersize: 32

vhosts:
  - name: your_domain.com  #改为你的域名
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
- 启动服务  
```bash 
cd && nohup ./tsp -config /etc/tsp/config.yaml >tsp.log 2<&1 &
docker run --network host --name v2ray -v /etc/v2ray:/etc/v2ray --restart=always -d teddysun/v2ray
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 更新软件
使用这种配置方式后，**watchtower**会自动监测并更新软件，你无需手动更新

# 客户端
Android系统: [点击下载](https://github.com/2dust/v2rayNG/releases)    

Windows && Linux && MacOS : [Qv2ray 下载](https://github.com/Qv2ray/Qv2ray/releases)   

Qv2ray 用法 : [文档](https://qv2ray.net/getting-started/step2.html) 