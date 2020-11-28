# 關於
本項目旨在提供幾個主流的代理軟體搭建教程，讓你獲得自由的上網體驗    
所有搭建方式均採用 [Docker](https://hub.docker.com/) , 方便寫入和升級     
# 內容
- [Xray](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Xray)		
- [Brook](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Brook)  		
- [VLess](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/V2ray/VLess)			
- [VMess](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/V2ray/VMess)		
- [Trojan](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Trojan)      
- [Trojan-GO](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Trojan-go)    	
- [Naiveproxy](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Naiveproxy) 		
- [Shadowsocks](https://github.com/charlieethan/firewall-proxy/tree/master/CN-HK/Shadowsocks)  	

# 注意事項
- 你需要擁有一個功能變數名稱並在搭建服務前將此功能變數名稱解析至你的伺服器IP    
由於互聯網上關於此的教程數以萬計，因此**如果你不會請自行學習**，教程中不再贅述基礎知識		

- 大部分教程（除 VMess+h2 和 Naiveproxy）以外，偽裝站均採用 [反向代理](https://juejin.im/post/6844903782556368910) 的形式，俗稱 “鏡像站”    
所有文章的步驟本人全部測試過，如果你遇見 **所有容器運行正常但偽裝站無法打開的情況** ，唯一的原因是：**反向代理的網站採用了反爬蟲措施**    
解決方法是：**更改偽裝站的網站，直到找見可以反向代理的網站**

# 推薦指數  
⭐⭐⭐⭐⭐⭐⭐ Xray    
⭐⭐⭐⭐⭐⭐ Trojan-GO       
⭐⭐⭐⭐⭐⭐ VLess	    	  
⭐⭐⭐⭐⭐ Trojan         
⭐⭐⭐⭐⭐ Naiveproxy		   	    
⭐⭐⭐⭐ Brook    
⭐⭐⭐⭐ VMess      			  
⭐⭐⭐⭐ Shadowsocks    

**注意：Naiveproxy 目前沒有移動客戶端支持，請自行決定是否使用。**		
# 推薦腳本	
如果你仍覺得麻煩，歡迎使用下麵的一鍵腳本。所有代碼已經經過安全審計，可以放心使用		

1.傳統部署：https://github.com/phlinhng/v2ray-tcp-tls-web		

2.Docker部署：https://github.com/h31105/trojan_v2_docker_onekey			
# 致謝  
<details>
<summary>點擊展開 </summary>

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
