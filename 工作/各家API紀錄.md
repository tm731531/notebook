---
title: 打各家API的小紀錄
tags: 電商工作
---

[toc]

## 台灣
### Yahoo
#### 操作紀錄
1. 客戶下單5 :43 但API要5:50以後才打到
   後面又下了一張5:52分的 API要6:02打的才有資料
   因此判斷yahoo 待出貨訂單的時間差為10分鐘一個cache(0 10 20 30 40 50)，跟文件中的大多都是10分鐘一致
   
2. 我方打了無法出貨API，但是還需要廠商去平台點下無法出貨的按鈕，才真正走完流程，這個已經跟yahoo確認證實，我方打API只是會變成一個狀態(再打一次就會異常)，真正的無法出貨要他們UI操作的那一個   

3. 退貨的有三種狀態 (結案 未結案 取消)，查詢時間也有兩種 一個是開始日 一個是結案日 

4. 已出貨訂單查詢的時間要使用訂單成立的日期 
ex: 5/12的訂單 5/15出貨 但是時間用5/15出貨查不到，要用5/12查 

5. 查詢取消的訂單，要使用所有的待出貨訂單編號去詢問Yahoo，平台回應的訂單就是要取消的訂單

6. 只有出貨確認的API，會有物流碼，其他的都沒有

#### 待做
1. 直配目前因為廠商沒有相關的配合功能無法測試
2. 提報相關的API，目前只有根據文件中的查詢庫存來做相關的測試，其他的還沒測試

#### 特別紀錄
1. 店配的出貨結構跟API文件不合
2. 因為設計的原因，他的API就算是Get http 也會需要body ，目前java支援的lib中只有一個有相關功能
3. version文件中有1 sample中有4，但是實際上打不知道4所用何處，目前都是以1為主
4. 文件中的回傳代碼只有在我們打過去通知他的會有相關的代碼必要，其他的還是遵照Http本身的response code就好
5. "商品編號" 網站上有但是不明顯，要去商品頁把字體反白才可以看到
6. 針對待出貨訂單，後續有兩個版本
```
1. Step1: 待出貨查詢：/scmapi/api/ThirdPartyLogistics/ToBeShippedOrders 
   提供 "OrderSubCode"作為分包單號依據 -> RSxxxxx
   假設RM2201100001328訂單下有兩個商品要分包出貨，透過該API取得該筆訂單下的子商品的單號
         RS2201100003299 -> PackageNo:1
         RS2201100003300 -> PackageNo:2
2. Step2: 包裝確認/scmapi/api/ThirdPartyLogistics/ConfirmPacked
         提供 "OrderSubCode"作為分包單號依據 -> RSxxxxx
   完成此步驟，則可產出兩張託運單號
3. Step3: 出貨確認：/scmapi/api/ThirdPartyLogistics/ConfirmShipped
   根據RM訂單編號完成出貨確認
```
7. GetPreparingOrders(不論是三方宅配或是店配)在Header中可以指定api-version為1或2，主要的差別就是在消費者實付金額。
``` 
Version1中顯示的就是賣場原始售價(其中包含了消費者所有的折扣)，所以這個版本會看不出來消費者實付金額。
Version2中就是會多顯示，各種的折扣金額。以您貼圖的案例：
賣場售價 - ProductPrice：315
賣場活動折扣 - ActivityAmount：31
(ProductPrice - ActivityAmount) 284 (消費者實付金額)
當然如果同時間有使用其他的折抵，就會顯示在其他的折抵金額欄位
```
8. 賣場編號跟商品序號是不一樣的
9. 資料回復跟打入都要全體加密

*********
### 東森
#### 操作紀錄
1. 目前已用測試區測試過訂單跟退貨的訂單查詢，以及相關的修改狀態
2. 已通過測試情境，正式區已開通
#### 待作
1. 東森訂單 退貨訂單


#### 特別紀錄
文件: https://b2b.etmall.com/B2BAPIGuide/WorkFlow/ShipManage
1. 修改的時候雖然success的欄位是成功，但是不代表你修改資料是成功的，會有failed count

2. 其他平台的出貨=這個平台的配送
3. 測試的時候按照這樣的測試情境走https://b2b.etmall.com/B2BAPIGuide/Home/TestCase
4. 小白單是沒有的

*********
### MOMO
#### 操作紀錄
https://hackmd.io/@sj890190/HJi7K7VHc

#### 待作

#### 特別紀錄
*********

### 91APP
#### 操作紀錄
1. 自訂出貨分兩種 一種是第三方(可取號的)
2. 訂單狀態跟物流狀態分開
3. V2有TG CODE(購物車代碼)
4. TM CODE為主 TS 為輔 TGV2參考用


#### 待作

#### 特別紀錄
1. V1的訂單對於生難字詞可以查詢到 V2目前無法
2. 宅配有海外
3. 貨到付款跟到店付款不同
4. 宅配單只能給資訊不給下載(有的是7-11店到店)
5. 訂單的狀態與流程大致如此(https://drive.google.com/file/d/1u62mr-X8kALFpzb8foflJPj9oB0c0Gy5/view)



*********
### Cyberbiz
#### 操作紀錄
1. 目前是以測試帳號做訂單資料取得的來源
2. 訂單內含四種狀態，以此轉成OneEC訂單
3. 訂單的配送方式，目前以shipping_vendor.type判斷
4. 訂單的原始物流公司，目前以fulfillment.tracking_company判斷

#### 待做

#### 特別紀錄
1. 每秒至多 5 個 Request
2. 每個請求上限為 2 MB (測試帳號 apidemo 上限為 2 KB)
3. 取得訂單時偶爾會出現unexpected end of stream的錯誤，因此在Header中有加入"Connection", "close"

*********
### Yahoo超級商城
#### 操作紀錄
2. 依不同訂單類型，選擇不同的日期條件

#### 待做
1. 待確認，查詢取消訂單查的狀態意義
-   NEW:未結案: 新申請(?)
-   CANCEL:取消: 出貨前的買家取消(?)
-   SHIPPED:完成出貨: 出貨前的賣家取消(?)
2. 待確認，未付款訂單是否會與未出貨訂單重複
3. 待確認，查詢未付款訂單的狀態意義
-   ALL:全部: 逾時還未付款的取消訂單 + 未付款的訂單(?)
-   NEW:未結案: 未付款的訂單(?)
-   CANCEL:取消: 逾時還未付款的取消訂單(?)
4.  待確認，/v1/Order/GetStatus，是否與查詢訂單+查詢未付款訂單重複
5.  物流運費的訂單資料統一收在otherData(?)
6.  待確認，交易序號及訂單編號哪一個要放OneEC的OrderSn / ItemSn


#### 特別紀錄
1. 同購物車, 若同時買兩個店家的商品，交易序號會依店家拆分，訂單編號會依商品 / 運費拆分
![](https://i.imgur.com/jZMLmPa.png)

#### 待討論
1. 配送方式有三: 宅配、超取、ESD(電子下載)，其中ESD(電子下載)歸類為OneEC的哪種配送方式

*********
## 外國
### 蝦皮

#### 操作紀錄
取得授權的流程
https://hackmd.io/kueSRfmtT3mxf2IRwp1xpA
取Token
```
{"access_token":"426f4c747449475279454e526c786f52","error":"","request_id":"0e49f1223ca7c9d6f6ae7fad7b12e1c2","message":"","expire_in":14376,"refresh_token":"4566616462615173667256557a4b4b79"}

```


過期TOKEN
```
{"url":"https://partner.test-stable.shopeemobile.com/api/v2/order/get_order_list?partner_id=1007089&shop_id=47073&timestamp=1650785194&access_token=7a744b4a706b545a5a56594969436d6a&sign=2617b0c14a7e9dce30ec80aaa258a46eb6a4747d6718bbdd59c8bcfa13d416a3&time_range_field=create_time&page_size=12&response_optional_fields=order_status&time_to=1650785194&time_from=1650698794","code":403,"body":"{\"request_id\":\"d8d76a654366dae3201b40990b7a2edc\",\"error\":\"error_auth\",\"message\":\"Invalid access_token.\"}","message":""}
End

```
重新取得TOKEN
```
{"shop_id":47073,"error":"","access_token":"0d4c837c1bdce4d2cfd3997325e3fd31","expire_in":14400,"message":"","request_id":"2d6a09eabbcfa291bc7ff51c84f9b4a2","partner_id":1007089,"refresh_token":"98a46ac14f34f26c3a170e53e52f6174"}

```
使用實際賣場測試的流程與資料
https://hackmd.io/5W8NcanaQyOryGv4J-2A0Q


#### 待作

#### 特別紀錄
1. 各應用的權限等級
https://hackmd.io/ie5c0hcOTequgct8JrtwzA?both
2. API格式變動的很快，一定要隨時注意信件
3. 有ticket system 但是回得很慢平常還是建議使用(openapi@support.shopee.com)
4. 配送單PDF UI可以多頁一起印 但是API目前尚不支援(2022/9/7)


*********

### 樂天
#### 操作紀錄
1. 訂單模式為 訂單List+detail 打兩次的模式

#### 待作

#### 特別紀錄
1. key 跟url  兩層驗證 Key是 base64字串放在bearer裡面
2. url使用於API的物件中 有點店名的意味
*********

### shopline
#### 操作紀錄

1. Orders就可以取得訂單資料 不用到 order
2. Class眾多 有些會是null 有些是""
3. Update 會包含新訂單 ，還有訂單狀態的更新
4. 訂單時間不要拉得太長比較好
5. 出貨 flow https://hackmd.io/@2GrxaqznSJKLrhmXwCaeug/BJSLn3-Mo
6. 測試資料 https://hackmd.io/@2GrxaqznSJKLrhmXwCaeug/ByRCNaWbs
#### 待作

#### 特別紀錄
1. 密碼: shopline (https://open-api.docs.shoplineapp.com/docs/get-order)
2. 唯一打訂單只看時間不看狀態的
3. 同IP 5分鐘 不能打超過一千次
4. 打太快(連續打進快100次就會被檔) perpage=50的情況下 會timeout
5. 相當遵守restful API規則(有 patch跟put delete等)


*********

### shopify
#### 操作紀錄

#### 待作

#### 特別紀錄
1. API ENDPOINT ( https://shopify.dev/api/admin-rest/2022-07/resources/order)
2. lib (https://shopify.dev/apps/tools/api-libraries#third-party-admin-api-libraries)
3. 

*********
### Amazon(https://spapi.cyou/zh/use-case/aplus-content-api-use-case-guide_2020-11-01.html#%E4%BB%BB%E5%8A%A1-5-%E5%88%9B%E5%BB%BA%E5%86%85%E5%AE%B9%E6%96%87%E6%A1%A3)
#### 操作紀錄

#### 待作

#### 特別紀錄
1. 賣場資料跟數量等等的是分離的
2. https://github.com/amzn/selling-partner-api-docs
3. mws 去年停止支援
4. 