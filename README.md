由於 AI 產生內容得靠伺服器運算後產生結果，資料多的話得等不少時間，若能產生資料後馬上分段送出，可大大提升使用者感受，要達到這種效果主要靠的是 Server-Send Event(以下簡稱SSE) 伺服器主動推播協定

前端只需寫個事件監聽函式，收到資料就立刻加在聊天訊息上，不需等全部內容產出就能開始觀看，而且資料產生的速度遠比人觀看的速度快，使用者可以不用對著螢幕發呆

Spring AI 的 ChatModel 也實作了這種方法，我們只需改兩個小地方即可達成流式輸出效果
![https://ithelp.ithome.com.tw/upload/images/20240803/201612903V3vt19AqN.png](https://ithelp.ithome.com.tw/upload/images/20240803/201612903V3vt19AqN.png)

1. 將原本的 chatModel.`call`(prompt) 改為 chatModel.`stream`(prompt)
2. 將回傳值由 `String` 改為 Flux`<String>`

這個 Flux 背後正是使用 SSE 技術，不過將細節封裝起來，後端實際上並不用花太多功夫處理

我們先看看 Postman 測試結果
![https://ithelp.ithome.com.tw/upload/images/20240803/20161290uBM3pgWpFo.png](https://ithelp.ithome.com.tw/upload/images/20240803/20161290uBM3pgWpFo.png)

結果跟昨天一樣，看起來能正常運行，不過似乎少了甚麼？我要的是流式輸出阿，你怎麼一次全給我

由於沒指定輸出格式 Postman 還是將結果收集完才送出
我們只需在 API 接口加上 `produces = MediaType.TEXT_EVENT_STREAM_VALUE`
![https://ithelp.ithome.com.tw/upload/images/20240803/20161290e1TleLRiQP.png](https://ithelp.ithome.com.tw/upload/images/20240803/20161290e1TleLRiQP.png)

這樣 Postman 就會知道 API 回傳內容將採用流式輸出
![https://ithelp.ithome.com.tw/upload/images/20240803/20161290vJWHoBSaAB.png](https://ithelp.ithome.com.tw/upload/images/20240803/20161290vJWHoBSaAB.png)

另外若直接呈現在網頁上會有中文亂碼問題，因為流式輸出會將內容採用 Byte 傳送
![https://ithelp.ithome.com.tw/upload/images/20240803/20161290iM2tbCjGAe.png](https://ithelp.ithome.com.tw/upload/images/20240803/20161290iM2tbCjGAe.png)

我們只要在 `application.yml` 增加字元格式，將其設為 `UTF-8` 這樣瀏覽器就知道要用 UTF-8 解碼

```yaml
server:
  servlet:
    encoding:
      charset: UTF-8
      enabled: true
      force: true
```

如此網頁上能正確顯示中文
![https://ithelp.ithome.com.tw/upload/images/20240803/20161290y3Y5lKVGuk.png](https://ithelp.ithome.com.tw/upload/images/20240803/20161290y3Y5lKVGuk.png)

回顧一下今天學到甚麼
- 將 AI 回傳結果改為流式輸出
- 解決網頁執行時中文亂碼問題

第一個章節總算順利完成，所學的也很有限，不過也算是透過 Spring AI 與 AI 溝通的第一步，明天就會開始有點寫程式的樣子了，敬請期待
