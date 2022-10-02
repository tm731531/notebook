---
title : solr 筆記
tags : 學習
---

[![hackmd-github-sync-badge](https://hackmd.io/oGjkVBP7TNe0_P19JwmGsg/badge)](https://hackmd.io/oGjkVBP7TNe0_P19JwmGsg)

[toc]

## solr

### 基本參數介紹
q  查詢的關鍵字，此參數最為重要，例如，q=id:1，默認為q=*:*，

　　fl  指定返回哪些字段，用逗號或空格分隔，注意：字段區分大小寫，例如，fl= id,title,sort

　　start  返回結果的第幾條記錄開始，一般分頁用，默認0開始

　　rows  指定返回結果最多有多少條記錄，默認值為 10，配合start實現分頁

　　sort  排序方式，例如id  desc 表示按照 “id” 降序，多個字段：score desc，price asc

　　wt  (writer type)指定輸出格式，有 xml, json, php等

　　fq  （filter query）過慮查詢，提供一個可選的篩選器查詢。返回在q查詢符合結果中同時符合的fq條件的查詢結果，例如：q=id:1&fq=sort:[1 TO 5]，找關鍵字id為1 的，並且sort是1到5之間的。

　　df   默認的查詢字段，一般默認指定。

　　qt  （query type）指定那個類型來處理查詢請求，一般不用指定，默認是standard。

　　indent   返回的結果是否縮進，默認關閉，用 indent=true|on 開啟，一般調試json,php,phps,ruby輸出才有必要用這個參數。

　　version   查詢語法的版本，建議不使用它，由服務器指定默認值。
  
  
  ### 查詢語法
   “:”  指定字段查指定值，如返回所有值*:*
　　“?”  表示單個任意字符的通配
　　“*”  表示多個任意字符的通配（不能在檢索的項開始使用*或者?符號）
　　“~”  表示模糊檢索，如檢索拼寫類似於”roam”的項這樣寫：roam~將找到形如foam和roams的單詞；roam~0.8，檢索返回相似度在0.8以上的記錄。
　　AND、||  布爾操作符
　　OR、&&  布爾操作符
　　NOT、!、-（排除操作符不能單獨與項使用構成查詢）
　　“+”  存在操作符，要求符號”+”後的項必須在文檔相應的域中存在²
　　( )  用於構成子查詢
　　[]  包含範圍檢索，如檢索某時間段記錄，包含頭尾，date:[201507 TO 201510]
　　{}  不包含範圍檢索，如檢索某時間段記錄，不包含頭尾date:{201507 TO 201510}
  
  
### 

## 參考 
1. https://blog.csdn.net/ystyaoshengting/article/details/50262763

2.
https://iter01.com/554961.html

3. solr 自聚类实现  https://www.cnblogs.com/shaosks/p/8309149.html
4. highlight https://www.cnblogs.com/shaosks/p/8405926.html
5. schema https://www.cnblogs.com/shaosks/p/8409125.html
6. 基本schema 配置 https://www.cnblogs.com/zhoujg/archive/2015/12/17/5053789.html