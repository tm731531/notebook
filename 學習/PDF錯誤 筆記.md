---
title : PDF錯誤 筆記
tags : 學習
---
[toc]

## 事件緣由
7/6 晚間七點
使用者端產生了一筆 1082筆資料的"檢貨單"
我們必須要產生相對應的PDF

## 環境
1. Kafka comsumer(max.poll.interval.ms 使用預設)
2. k8s
3. spring boot 2.6.6

## 錯誤

### 1. 五分鐘自動啟動Kafka連線

![](https://i.imgur.com/0C6rsxm.jpg)

![](https://i.imgur.com/xWeawho.jpg)

解法 : 把comsumer拉長到1小時

![](https://i.imgur.com/5GwiM4a.jpg)

![](https://i.imgur.com/oirt4YL.jpg)

但是還是無解

### 2. PDF生產導致CPU強烈飆高

![](https://i.imgur.com/vj90f8f.jpg)

原因: 原本的CPU設定1000m 記憶體1G
資源不夠

解法: 拉大到4000m 記憶體 4G
衍伸問題 >> CPU吃到10 還是無解

### 3. spring boot health checking error(K8S)
```
Destroying Spring FrameworkServlet 'dispatcherServlet'
```
解法
gradle:
```
spring boot version >>2.6.9
jackson  version >> 2.13.3
```

helm :
```
deployment 增加  timeoutSeconds: 3
```

### 4. 第90筆資料中的訂單資料過長

![](https://i.imgur.com/TFsdtwe.jpg)

原因:訂單資料太多筆 ，導致訂單這個欄位太長>>在計算是否要下一頁的時後產生一個無窮迴圈
![](https://i.imgur.com/SqMPScd.jpg)

![](https://i.imgur.com/rKYAPth.jpg)

解法:限制每一個欄位的既定長度跟高度，讓PDF的計算下一頁可以正常WORK


## 結論 

一開始一直朝向工作時間 >>工作資源 >>系統資源擴大的角度思考(X)

後來把真實資料丟給SAMPLE測試 RUN一次就發現問題(O)

下一次先丟資料重現(◎)


## (Option) 參考資料

1. spring boot 2.6.9 修正 health check https://github.com/spring-projects/spring-boot/releases/tag/v2.6.9

issue : https://github.com/spring-projects/spring-boot/issues/31231

2. kafka consumer tuning 
https://strimzi.io/blog/2021/01/07/consumer-tuning/
