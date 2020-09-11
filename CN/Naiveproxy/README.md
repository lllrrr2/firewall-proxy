## 准备工作
你需要拥有一个自己的**域名**，并**已经将域名解析至你的服务器**    
## 搭建环境
硬件 : 内存 ≧ 512M 储存 ≧ 5G | 64位系统			

软件 : Debian 9/10 && Ubuntu 16/18/20
## 内容
- 安装基础工具  
```bash
apt update && apt install -y libnss3 wget unzip
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安装 Caddy && Naiveproxy	
```bash
wget -P /usr/local/bin https://github.com/charlieethan/firewall-proxy/releases/download/2.2.0/caddy
chmod +x /usr/local/bin/caddy && setcap cap_net_bind_service=+ep /usr/local/bin/caddy
```
- 下载网站模板	  
**我准备了50个伪装网站模板，这里只是一个示例，你可以将 `1.zip` 改为 `2~50.zip`**		
```bash
mkdir -p /var/www/html /etc/caddy && wget -P /var/www/html https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip && unzip /var/www/html/1.zip -d /var/www/html
```
- 修改配置
```bash
cat > /etc/caddy/caddy.json <<EOF
{ 
  "admin": {"disabled": true},
  "logging": {
    "sink": {"writer": {"output": "stdout"}},
    "logs": {"default": {"writer": {"output": "stdout"}}}
  },
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
              "auth_user": "user_name",     #改为你的用户名
              "auth_pass": "your_password",     #改为你的密码
              "probe_resistance": {"domain": "unsplash.com:443"}
            }]
          }, {
            "match": [{"host": ["your_domain.com"]}],    #改为你的域名
            "handle": [{
              "handler": "file_server",
              "root": "/var/www/html"
            }],
            "terminal": true
          }],
          "tls_connection_policies": [{
            "match": {"sni": ["your_domain.com"]}   #改为你的域名
          }]
        }
      }
    },
    "tls": {
      "automation": {
        "policies": [{
          "subjects": ["your_domain.com"],   #改为你的域名
          "issuer": {
            "module": "acme",
            "email": "your@email.com"  #改为你的邮箱地址
          }
        }]
      }
    }
  }
}
EOF
```
- 启动服务  
```bash
nohup caddy run --config /etc/caddy/caddy.json >/etc/caddy/caddy.log 2<&1 &
```
- 开启 BBR 加速
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 客户端
**目前没有移动客户端支持，你只能在电脑上使用！**		  

Windows && Linux && MacOS : [Qv2ray 下载](https://github.com/Qv2ray/Qv2ray/releases)	     

支持 Naiveproxy 的插件 : [插件下载](https://github.com/Qv2ray/QvPlugin-NaiveProxy/releases) 		

插件的用法 : [文档](https://qv2ray.net/plugins/usage.html)	
## 扩展阅读
[Naiveproxy+VLess搭建](https://blog.charlieethan.com/index.php/archives/539.html)