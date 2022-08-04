---
title : kafka 錯誤筆記
tags : 學習
---

[toc]
## 2022/08/04 
### 事起源由
8/4早上塞入了大量rollback的蝦皮queue資料
但是完全沒有消除的跡象，but訂單卻有近來
形成一種queue沒有消除 但是資料確有近來的詭異現象

### 解決方法
把取queue的數量，從一次拿10個，改成一次拿2個

### 當時狀況
#### Kafka
針對整個去打平台的group 都是在"Stable"的狀況

#### Job機
兩個都是綠燈 正常運作，也沒有發出任何錯誤信件出來

**看起來一切良好**

### 深入了解後發現幾個狀況
#### job機有拉 有跑但是沒有commit成功
 
 ```
 2022-08-04 03:10:44.933 ERROR 8 --- [       Thread-2] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-call-api-to-get-data-1, groupId=call-api-to-get-data] Offset commit with offsets {action-to-shopee-56=OffsetAndMetadata{offset=964, leaderEpoch=23, metadata=''}, action-to-shopee-23=OffsetAndMetadata{offset=906, leaderEpoch=22, metadata=''}, action-to-shopee-52=OffsetAndMetadata{offset=929, leaderEpoch=22, metadata=''}, action-to-shopee-19=OffsetAndMetadata{offset=874, leaderEpoch=25, metadata=''}, action-to-shopee-2=OffsetAndMetadata{offset=916, leaderEpoch=22, metadata=''}, action-to-shopee-31=OffsetAndMetadata{offset=853, leaderEpoch=17, metadata=''}, action-to-shopee-60=OffsetAndMetadata{offset=799, leaderEpoch=35, metadata=''}, action-to-shopee-27=OffsetAndMetadata{offset=859, leaderEpoch=19, metadata=''}, action-to-shopee-10=OffsetAndMetadata{offset=902, leaderEpoch=27, metadata=''}, action-to-shopee-39=OffsetAndMetadata{offset=885, leaderEpoch=25, metadata=''}, action-to-shopee-6=OffsetAndMetadata{offset=951, leaderEpoch=22, metadata=''}, action-to-shopee-35=OffsetAndMetadata{offset=879, leaderEpoch=31, metadata=''}, action-to-shopee-18=OffsetAndMetadata{offset=873, leaderEpoch=28, metadata=''}, action-to-shopee-47=OffsetAndMetadata{offset=898, leaderEpoch=19, metadata=''}, action-to-shopee-14=OffsetAndMetadata{offset=922, leaderEpoch=28, metadata=''}, action-to-shopee-43=OffsetAndMetadata{offset=880, leaderEpoch=22, metadata=''}, action-to-shopee-57=OffsetAndMetadata{offset=898, leaderEpoch=27, metadata=''}, action-to-shopee-24=OffsetAndMetadata{offset=890, leaderEpoch=29, metadata=''}, action-to-shopee-53=OffsetAndMetadata{offset=909, leaderEpoch=33, metadata=''}, action-to-shopee-20=OffsetAndMetadata{offset=871, leaderEpoch=36, metadata=''}, action-to-shopee-32=OffsetAndMetadata{offset=899, leaderEpoch=22, metadata=''}, action-to-shopee-61=OffsetAndMetadata{offset=942, leaderEpoch=27, metadata=''}, action-to-shopee-28=OffsetAndMetadata{offset=900, leaderEpoch=27, metadata=''}, action-to-shopee-40=OffsetAndMetadata{offset=894, leaderEpoch=35, metadata=''}, action-to-shopee-7=OffsetAndMetadata{offset=989, leaderEpoch=19, metadata=''}, action-to-shopee-36=OffsetAndMetadata{offset=868, leaderEpoch=23, metadata=''}, action-to-shopee-3=OffsetAndMetadata{offset=862, leaderEpoch=22, metadata=''}, action-to-shopee-48=OffsetAndMetadata{offset=2514, leaderEpoch=27, metadata=''}, action-to-shopee-15=OffsetAndMetadata{offset=952, leaderEpoch=31, metadata=''}, action-to-shopee-44=OffsetAndMetadata{offset=884, leaderEpoch=29, metadata=''}, action-to-shopee-11=OffsetAndMetadata{offset=883, leaderEpoch=17, metadata=''}, action-to-shopee-58=OffsetAndMetadata{offset=870, leaderEpoch=28, metadata=''}, action-to-shopee-25=OffsetAndMetadata{offset=923, leaderEpoch=32, metadata=''}, action-to-shopee-54=OffsetAndMetadata{offset=964, leaderEpoch=28, metadata=''}, action-to-shopee-21=OffsetAndMetadata{offset=827, leaderEpoch=27, metadata=''}, action-to-shopee-33=OffsetAndMetadata{offset=886, leaderEpoch=33, metadata=''}, action-to-shopee-0=OffsetAndMetadata{offset=927, leaderEpoch=35, metadata=''}, action-to-shopee-62=OffsetAndMetadata{offset=870, leaderEpoch=23, metadata=''}, action-to-shopee-29=OffsetAndMetadata{offset=877, leaderEpoch=34, metadata=''}, action-to-shopee-41=OffsetAndMetadata{offset=911, leaderEpoch=27, metadata=''}, action-to-shopee-8=OffsetAndMetadata{offset=951, leaderEpoch=27, metadata=''}, action-to-shopee-37=OffsetAndMetadata{offset=880, leaderEpoch=27, metadata=''}, action-to-shopee-4=OffsetAndMetadata{offset=846, leaderEpoch=30, metadata=''}, action-to-shopee-49=OffsetAndMetadata{offset=917, leaderEpoch=34, metadata=''}, action-to-shopee-16=OffsetAndMetadata{offset=988, leaderEpoch=23, metadata=''}, action-to-shopee-45=OffsetAndMetadata{offset=885, leaderEpoch=33, metadata=''}, action-to-shopee-12=OffsetAndMetadata{offset=851, leaderEpoch=22, metadata=''}, action-to-shopee-26=OffsetAndMetadata{offset=898, leaderEpoch=22, metadata=''}, action-to-shopee-55=OffsetAndMetadata{offset=986, leaderEpoch=31, metadata=''}, action-to-shopee-22=OffsetAndMetadata{offset=849, leaderEpoch=22, metadata=''}, action-to-shopee-51=OffsetAndMetadata{offset=820, leaderEpoch=17, metadata=''}, action-to-shopee-34=OffsetAndMetadata{offset=957, leaderEpoch=28, metadata=''}, action-to-shopee-1=OffsetAndMetadata{offset=803, leaderEpoch=27, metadata=''}, action-to-shopee-63=OffsetAndMetadata{offset=905, leaderEpoch=22, metadata=''}, action-to-shopee-30=OffsetAndMetadata{offset=924, leaderEpoch=27, metadata=''}, action-to-shopee-59=OffsetAndMetadata{offset=923, leaderEpoch=25, metadata=''}, action-to-shopee-42=OffsetAndMetadata{offset=980, leaderEpoch=22, metadata=''}, action-to-shopee-9=OffsetAndMetadata{offset=954, leaderEpoch=35, metadata=''}, action-to-shopee-38=OffsetAndMetadata{offset=925, leaderEpoch=28, metadata=''}, action-to-shopee-5=OffsetAndMetadata{offset=913, leaderEpoch=33, metadata=''}, action-to-shopee-50=OffsetAndMetadata{offset=902, leaderEpoch=27, metadata=''}, action-to-shopee-17=OffsetAndMetadata{offset=885, leaderEpoch=27, metadata=''}, action-to-shopee-46=OffsetAndMetadata{offset=918, leaderEpoch=22, metadata=''}, action-to-shopee-13=OffsetAndMetadata{offset=851, leaderEpoch=33, metadata=''}} failed

org.apache.kafka.clients.consumer.CommitFailedException: Offset commit cannot be completed since the consumer is not part of an active group for auto partition assignment; it is likely that the consumer was kicked out of the group.
 at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.sendOffsetCommitRequest(ConsumerCoordinator.java:1139) ~[kafka-clients-2.8.1.jar!/:na]
 at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.doCommitOffsetsAsync(ConsumerCoordinator.java:957) ~[kafka-clients-2.8.1.jar!/:na]
 at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.commitOffsetsAsync(ConsumerCoordinator.java:924) ~[kafka-clients-2.8.1.jar!/:na]
 at org.apache.kafka.clients.consumer.KafkaConsumer.commitAsync(KafkaConsumer.java:1561) ~[kafka-clients-2.8.1.jar!/:na]
 at org.apache.kafka.clients.consumer.KafkaConsumer.commitAsync(KafkaConsumer.java:1529) ~[kafka-clients-2.8.1.jar!/:na]
 at org.apache.kafka.clients.consumer.KafkaConsumer.commitAsync(KafkaConsumer.java:1506) ~[kafka-clients-2.8.1.jar!/:na]
 at com.systex.oneec.consuming.action.job.init.MainManager.run(MainManager.java:84) ~[classes!/:na]
 at java.base/java.lang.Thread.run(Thread.java:833) ~[na:na]
 ```
 
#### 重新刪除job機後一些狀況
1. kafka 進入 
   1. Preparing Rebalance 狀態
   2. Completing Rebalance 狀態
   3. stable狀態 
   4. Preparing Rebalance 狀態
   5. stable狀態 
2. 同時進入job機看 會出現這一個訊息
```
 [Consumer clientId=consumer-call-api-to-get-data-1, groupId=call-api-to-get-data] Attempt to heartbeat failed since group is rebalancing
```
3. 然後comsumer數量就減少了，但是步驟5的時後又加回來了
**此狀況是有點異常，因為正常的kafka應該進入到3後不應該再回到4，而且走到5**

#### 再次仔細觀察+回想場景
1. code沒變 昨天正常沒道理今天不正常
2. 差異點 今天是吃回滾資料
3. 回滾資料有什麼特殊的
4. 回滾資料會把已完成的訂單完全拉回來+財務資料
5. 然後這樣需要多少時間呢
6. 經驗證 已完成的訂單時間拉10天左右 將近三百張訂單 蝦皮有訂單數量一次請求的上限
7. 外加都要打財務資料
8. 一次queue的總時間將近一分鐘


一次queue所花的時間將近1分鐘<<這個是重點 紀錄一下

然後我們跟kafka的timeout時間吃的是預設(5分鐘) ，並且拉了10個queue來做處理，避免麻煩

問題就在這裡發生了

10個queue 每一個1分鐘 我做完需要10分鐘
但是kafka 5分鐘就認為我死了，把我剔除group
因此我回壓commit的時候無法回壓
取的queue沒有更新 就再次動作

### 總結問題
1. queue的數量*一個queue所需的時間 在某些場景下 太久 造成kafka認定死亡
2. commitasync導致我queue認定正常工作

一句話就是 job機跟kafka註冊 拿了工作，但是做太久，沒有再去拿下一批JOB，kafka就把他刪除了

### 可處理的方法
1. 不可能針對commitasync處理
2. 減低一次取的queue的量
3. 減少queue裡面所需要網外打API的工作(減少時間)
4. 拉高comsumer的timeout時間

### Kafka 狀態表


|  | Column 2  |
| -------- | --------  |
| Stable     | Consumer group has members which have been assigned partitions |
| Completing Rebalance     | Kafka is assigning partitions to group members|
| Preparing Rebalance     | A reassignment of partitions is required, members have been asked to stop consuming|
| Empty     | Consumer group exists, but does not have any members|
| Dead     | Consumer group does not have any members and it's metadata has been removed|
| Unknown     | Group state is not known|


![](https://i.imgur.com/ljYhTaC.png)


### 參考資料
1. https://stackoverflow.com/questions/40162370/heartbeat-failed-for-group-because-its-rebalancing
2. https://stackoverflow.com/questions/39730126/difference-between-session-timeout-ms-and-max-poll-interval-ms-for-kafka-0-10/39759329#39759329
3. 
