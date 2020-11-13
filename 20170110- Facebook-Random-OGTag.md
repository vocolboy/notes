# 起手式
* 一台能連上 GOOGLE 的電腦

* 一雙能看懂 Facebook Graph Api 眼睛

* 一台 Server

* 一顆 Domain

* ( 非必要 )一組帳密 facebook application **app-id** & **app-secret**


# 正題開始

### 先說說大概流程：
> 有一台 Server 可以讓 FB 爬蟲戳戳 <br>
> 定時去打 Graph Debugger URL API 讓他重新 cache<br>
> 全文完 ✔️


### 實作過程

一. 把機器 Run 起來跟 Domain 綁一起有個 endpoint
> 這邊要注意的在 facebook 牆上貼純文字是預設抓 **HTTP** <br>
>
> ex. 留言 `eatgether.com` <br>
> Facebook 會去解析 `http://eatgether.com` <br>

> 所以只想玩玩就不用特地處理 **HTTPS** 了

二. 寫一個小型專案在機器上，並且有個 endpoint 可以進入渲染網頁
> 最簡單的就是 先寫死一組 list
>
```javascript
{
    "item-1": {
        "og:url": "item-1-og-uri",
        "og:title": "item-1-og-title",
        "og:description": "item-1-og-description",
        "og:image":"item-1-og-image-uri"
    },
    "item-2": {
        "og:url": "item-2-og-uri",
        "og:title": "item-2-og-title",
        "og:description": "item-2-og-description",
        "og:image":"item-2-og-image-uri"
    }
}
```

> 然後就是隨機塞進網頁的 og:tag 裡
>
> 照片檔案最好越小越好 | **最小限制尺寸 200 * 200**

> Guide : https://developers.facebook.com/docs/sharing/webmasters#basic
>
> 想知道有沒有寫對可以用：https://developers.facebook.com/tools/debug/

三. 跑個排程打打 Graph Debugger URL API
> GET method "https://graph.facebook.com/v2.9/?scrape=true&id=http://eatgether.com/"
>
> **上述網址請記得換成自己的**
>
> 推薦設定排程時間 5s 差不多
>
> 這邊還有一點，這個 graph 可以不組上 access token 但是會有限制大概一天就被限制流量了 <br>
>
> 建議可以組上玩比較久

> Guide : https://developers.facebook.com/docs/facebook-login/access-tokens#apptokens

四. 加速優化方法
> 真的架設好會發現，奇怪怎麼有時候照片和標題對不上 <br>
>
> 這時候你可以選擇把 網站上的 endpoint 變成兩個 <br>
>
> A endpoint 負責隨機一組 key 再導向 B <br>
>
> B endpoint 接到後組到網址上 <br>
>
> 看起來就會是 `http://eatgether.com`  == 302 導向 ==> `http://eatgether.com/item-1`
>
> 好處是什麼在 ?
>
> Facebook 他的 cache 照片會把
> `http://eatgether.com/item-1 , item-1-og-image-uri` 放一起
>
> `http://eatgether.com` 就只是單純紀錄導向後的網址，這樣顯示照片資源就會相對準確快速
