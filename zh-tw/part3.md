# 3. 延遲搭救法

一如以往，問題當前，人們便會聚首尋求對策。某一些緩解措施是神來之筆，有一些卻只是臨時充撐的爛木橋。

## 3.1 Spriting
<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/spriting.jpg" />

Spriting （圖像織結）通常是指將許多小圖像湊合成單一的大影像。然後，就可以利用 Javascript 或 CSS 從大影像中「剪出」所需的碎片，以顯示較小的個別圖像。

網站運用此技巧以求速度。以 HTTP 1.1 提取單一的大影像，比起個別 100 張較小的圖像來得快許多。

當然，比如網站上一些頁面只想顯示一兩張小圖片，就會帶來短處。同時，當快取逾期時，所有圖片亦會同時消失，即使一些最常用的亦不能倖免。

## 3.2 Inlining

Inlining （內述法）是另一種避免傳送許多個別圖像的技巧，方法是使用內嵌在 CSS 檔案內的 data: URL。好處與壞處與 spriting 類同。

    .icon1 {
        background: url(data:image/png;base64,<data>) no-repeat;
    }

    .icon2 {
        background: url(data:image/png;base64,<data>) no-repeat;
    }


## 3.3 Concatenation

一個大型的網站或需許多不同的 javascript 檔案。一些前端工具會幫助開發者將每一個合併成單一的大匯集，讓瀏覽器提取單一的大檔案，毋須逐個提取。當實際上需要很少的時候，傳送的資料就會太多。當需要作出一個修改的時候，需要重新載入的資料亦會太多。

這方法當然最不便的大多是參與的開發者。

## 3.4 Sharding

最後一個我想提及的效能提升技巧，常被稱為 “sharding”（域名分割）。簡單來說，就是儘可能以許多不同的主機域名處理網頁服務的不同部份。乍看之下或許很奇怪，但背後卻言之成理。

起始 HTTP 1.1 規格述明一個客戶端就每個主機允許使用最多兩個 TCP 連線。因此，為免有違規格，聰明的網站就創立出新的主機名稱。突然間，你的網站就能有更多的連線，減少了頁面載入時間。

隨著時間過去，該限制被移除，今日客戶端輕易就使用六至八個連線至每個主機名稱，但始終有個 they still have a limit so sites continue to use this technique to bump the number of connections. As the number of objects are ever increasing – as I showed before – the large number of connections are then used just to make sure HTTP performs well and makes your site fast. It is not unusual for sites to use well over 50 or even up to and beyond 100 connections now for a single site using this technique. Recent stats from httparchive.org show that the top 300K URLs in the world need on average 40(!) TCP connections to display the site, and the trend says this is still increasing slowly over time.

Another reason is also to put images or similar resources on a separate host name that doesn't use any cookies, as the size of cookies these days can be quite significant. By using cookie-free image hosts you can sometimes increase performance simply by allowing much smaller HTTP requests!

The picture below shows how a packet trace looks like when browsing one of Sweden's top web sites and how requests are distributed over several host names.

![image sharding at expressen.se](https://raw.githubusercontent.com/bagder/http2-explained/master/images/expressen-sharding.jpg)
