---
title : cipher 筆記
tags : 學習
---
[toc]
    
    
## 事件緣由

對方的Windows server 2012 無法連線上我們的網站
但是其他的網站都可以

比對DNS解析 （O）
比對PING （O）
比對IP （O）
比對加解密（O）
請對方開我們的網站（X）
但是對方的開發電腦 可以連我方的網站（O）


經過一段時間的驗證
可能與"根憑證"有關（ 事後發現沒問題）

對方檢查錯誤訊息 從handshake就發生問題（ssl）
於是檢查"加密套件"的問題
於是開始了這一條路

## 1. 我方的server認可的加密套件
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305

## 2. 對方有支援的
![](https://i.imgur.com/ECxUjEl.jpg)

查詢方法
[Windows 官方](https://docs.microsoft.com/zh-tw/windows/win32/secauthn/tls-cipher-suites-in-windows-8-1)
gpedit.msc<< WINDOWS 開啟調整指令
![](https://i.imgur.com/KpbWAws.png)
![](https://i.imgur.com/96Q9Bu0.png)


[英文說明板](https://www.howtogeek.com/221080/how-to-update-your-windows-server-cipher-suite-for-better-security/)

可以設定的（2022/06/27） <<可能隨著電腦的強化 演算法也會GG
https://www.grc.com/miscfiles/SChannel_Cipher_Suites.txt
```
   - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    - TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    - TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
    - TLS_RSA_WITH_AES_128_GCM_SHA256
    - TLS_RSA_WITH_AES_256_GCM_SHA384
    - TLS_RSA_WITH_AES_128_CBC_SHA256
```

## 3. 但是對方還是無法跟我方handshake
可能原因
Q 對方server太久2012 
A 請對方補齊相對應的windows server update KB

Q 檢查後發現雖然我們有六個 但是實際上只有三個
A 因為我方SSL簽證是RSA 因此ECDHE_ECDSA都無法用
![](https://i.imgur.com/KXbdT03.png)
```
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
```
因此只能用此三個 但是這三個 跟對方的server差異太高
對方無法更新上去

## 解決方法

經比對shopee 東森 PCHOME 等平台比對後
發現她們等級都是B
https://www.ssllabs.com/ssltest/analyze.html?d=www.pchome.com&latest
甚至是AMAZON
https://www.ssllabs.com/ssltest/analyze.html?d=https://www.amazon.com

因此我們決定加入一個有點危險weak的演算法
但是此演算RSA +256
雖然Weak 但是還是有A等的保障
https://www.ssllabs.com/ssltest/analyze.html?d=scm.oneec.ai
:::info 
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA 
:::




## （Optional）各種演算法大概介紹
https://zh.wikipedia.org/zh-tw/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A

## （Optional）TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA 這個是什麼意思呢

https://www.toutiao.com/article/6796609560049091085/?wid=1656311511834

重點是:

那麼Cipher的這一長串名字是什麼含義呢？其實，每種Cipher的名字裡包含了四部分信息，分別是

密鑰交換算法，用於決定客戶端與服務器之間在握手的過程中如何認證，用到的算法包括RSA，Diffie-Hellman，ECDH，PSK等
加密算法，用於加密消息流，該名稱後通常會帶有兩個數字，分別表示密鑰的長度和初始向量的長度，比如DES 56/56, RC2 56/128, RC4 128/128, AES 128/128, AES 256/256
報文認證信息碼（MAC）算法，用於創建報文摘要，確保消息的完整性（沒有被篡改），算法包括MD5，SHA等。
PRF（偽隨機數函數），用於生成“master secret”。
　　完全搞懂上面的內容似乎還需要一本書的介紹（我已經力不從心了）。不過大致了解一下，有助於理解Cipher的名字，比如前面服務器發回給客戶端的Cipher，

　　
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA

　　從其名字可知，它是

1. 基於TLS協議的；
2. 使用ECDHE、RSA作為密鑰交換算法；
3. 加密算法是AES（密鑰和初始向量的長度都是256）；
4. MAC算法（這裡就是哈希算法）是SHA。





## （Optional）程式上面要怎麼設定HTTPS走哪一個加密演算呢

Java:
https://developer.51cto.com/article/633932.html
否則就是server上有什麼就按照順序TRY什麼


C# :
https://docs.microsoft.com/zh-tw/dotnet/api/system.net.security.ciphersuitespolicy.-ctor?view=net-6.0

NET 5 在設定上就有所差異
https://docs.microsoft.com/zh-tw/dotnet/core/compatibility/cryptography/5.0/default-cipher-suites-for-tls-on-linux

比較多可以設定的都在net core上
framwork 目前看只有可以設定TLS1.2等


## （Optional） 可參考資源

黑大解釋: https://blog.darkthread.net/blog/rdp-ssl-cipher-vulnerability/
IIS加密GUI設定工具 :https://www.nartac.com/Products/IISCrypto/Download
分析server用什麼加密:https://www.ssllabs.com/ssltest/analyze.html

handshake做了哪些事情:https://www.cloudflare.com/zh-tw/learning/ssl/what-happens-in-a-tls-handshake/

查詢電腦支援那些cipher:https://docs.microsoft.com/zh-tw/windows/win32/secauthn/tls-cipher-suites-in-windows-8-1

英文的圖文解說板:https://www.howtogeek.com/221080/how-to-update-your-windows-server-cipher-suite-for-better-security/
