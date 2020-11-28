# 关于
本项目旨在提供几个主流的代理软件搭建教程，让你获得自由的上网体验    
所有搭建方式均采用 [Docker](https://hub.docker.com/) , 方便写入和升级     
# 内容
- [Xray](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Xray)		
- [Brook](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Brook)  		
- [VLess](https://github.com/charlieethan/firewall-proxy/tree/master/CN/V2ray/VLess)			
- [VMess](https://github.com/charlieethan/firewall-proxy/tree/master/CN/V2ray/VMess)		
- [Trojan](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Trojan)      
- [Trojan-GO](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Trojan-go)    	
- [Naiveproxy](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Naiveproxy) 		
- [Shadowsocks](https://github.com/charlieethan/firewall-proxy/tree/master/CN/Shadowsocks)  	

# 注意事项
- 你需要拥有一个域名并在搭建服务前将此域名解析至你的服务器IP    
由于互联网上关于此的教程数以万计，因此**如果你不会请自行学习**，教程中不再赘述基础知识		

- 大部分教程（除 VMess+h2 和 Naiveproxy）以外，伪装站均采用 [反向代理](https://juejin.im/post/6844903782556368910) 的形式，俗称 “镜像站”    
所有文章的步骤本人全部测试过，如果你遇见 **所有容器运行正常但伪装站无法打开的情况** ，唯一的原因是：**反向代理的网站采用了反爬虫措施**    
解决方法是：**更改伪装站的网站，直到找见可以反向代理的网站**

# 推荐指数  
⭐⭐⭐⭐⭐⭐⭐ Xray    
⭐⭐⭐⭐⭐⭐ Trojan-GO       
⭐⭐⭐⭐⭐⭐ VLess	    	  
⭐⭐⭐⭐⭐ Trojan         
⭐⭐⭐⭐⭐ Naiveproxy		   	    
⭐⭐⭐⭐ Brook    
⭐⭐⭐⭐ VMess     		  
⭐⭐⭐⭐ Shadowsocks    


**注意：Naiveproxy 目前没有移动客户端支持，请自行决定是否使用。**		
# 推荐脚本	
如果你仍觉得麻烦，欢迎使用下面的一键脚本。所有代码已经经过安全审计，可以放心使用		

1.传统部署：https://github.com/phlinhng/v2ray-tcp-tls-web		

2.Docker部署：https://github.com/h31105/trojan_v2_docker_onekey			
# 致谢  
<details>
<summary>点击展开 </summary>

- [@teddysun](https://hub.docker.com/u/teddysun)    
- [Shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)    
- [Brook](https://github.com/txthinking/brook)				  
- [Naiveproxy](https://github.com/klzgrad/naiveproxy)		
- [V2ray(V2fly)](https://github.com/v2fly/v2ray-core)         
- [Trojan](https://github.com/trojan-gfw/trojan)       
- [Trojan-GO](https://github.com/p4gefau1t/trojan-go)              
- [across](https://github.com/teddysun/across)     
- [Trojan-Qt5](https://github.com/Trojan-Qt5/Trojan-Qt5)     
- [v2rayN](https://github.com/2dust/v2rayN)      
- [v2rayNG](https://github.com/2dust/v2rayNG)     
- [Xray](https://github.com/XTLS/Xray-core)		
- [tls-shunt-proxy](https://github.com/liberal-boy/tls-shunt-proxy)		
- [shadowsocks-android](https://github.com/shadowsocks/shadowsocks-android)     
- [shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows)       
</details>
