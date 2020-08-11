## Prepare 
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first	   
- **Please pay attention to the marks on each line of the config files, and modify them as requested**    	
# Build Environment
Hardware : RAM ≧ 512M ROM ≧ 5G | 64bit OS Required			

Software : Debian 9/10 && Ubuntu 16/18/20	
# Config
- install basic tools 
```bash
apt update && apt -y install libnss3 wget unzip
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- get Caddy && Naiveproxy	
```bash
wget https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1/caddy
chmod +x caddy && setcap cap_net_bind_service=+ep ./caddy
```
- get HTML Tamplates	  
**I prepared 20 templates to use,this is an example to download one of them. You can modify `1.zip` to `2~20.zip`**		
```bash
mkdir -p /var/www/html && cd /var/www/html
wget https://github.com/charlieethan/firewall-proxy/releases/download/2.1.1-t/1.zip && unzip 1.zip 
```
- modify config files
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
              "auth_user": "user_name",     #modify to your username
              "auth_pass": "your_password",     #modify to your password
              "probe_resistance": {"domain": "unsplash.com:443"}
            }]
          }, {
            "match": [{"host": ["your_domain.com"]}],    #modify "your_domain.com" to your domain
            "handle": [{
              "handler": "file_server",
              "root": "/var/www/html"
            }],
            "terminal": true
          }],
          "tls_connection_policies": [{
            "match": {"sni": ["your_domain.com"]}   #modify "your_domain.com" to your domain
          }]
        }
      }
    },
    "tls": {
      "automation": {
        "policies": [{
          "subjects": ["your_domain.com"],   #modify "your_domain.com" to your domain
          "issuer": {
            "module": "acme",
            "email": "your@email.com"  #modify "your@email.com" to your email address
          }
        }]
      }
    }
  }
}
EOF
```
- Start Service  
```bash
nohup ./caddy run --config caddy.json >caddy.log 2<&1 &
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# Client
**There is no Android && iOS support, only PC have software to use!**		  

Windows && Linux && MacOS : [Qv2ray Download](https://github.com/Qv2ray/Qv2ray/releases)	

Plugin to support Naiveproxy in Qv2ray : [Plugin Download](https://github.com/Qv2ray/QvPlugin-NaiveProxy/releases) 		

The Usage of Qv2ray (Chinese Only) : [Usage](https://qv2ray.net/plugins/usage.html)		