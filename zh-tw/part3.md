# 3. 延遲搭救法

一如以往，問題當前，人們便會聚首尋求對策。某一些緩解措施是神來之筆，某一些卻只是拼湊充撐的爛木橋。

## 3.1 Spriting
<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/spriting.jpg" />

Spriting （圖像織結）通常是指將許多小圖像湊合成單一的大影像。然後，就可以利用 Javascript 或 CSS 從大影像中「剪出」所需的碎片，以顯示較小的個別圖像。

網站用此技巧求速度。以 HTTP 1.1 提取單一的大影像，比起提取個別 100 張較小的圖像來得快許多。

當然，比如網站上一些頁面只想顯示一兩張小圖片，就會帶來短處。同時，當快取逾期時，所有圖片亦會同時消失，即使一些最常用的小圖像亦不能倖免。

## 3.2 Inlining

Inlining （內述法）是另一種避免傳送許多個別圖像的技巧，方法是使用內嵌在 CSS 檔案內的 data: URL。好處與壞處與 Spriting 類同。

    .icon1 {
        background: url(data:image/png;base64,<data>) no-repeat;
    }

    .icon2 {
        background: url(data:image/png;base64,<data>) no-repeat;
    }


## 3.3 Concatenation

一個大型的網站或需許多不同的 Javascript 檔案。一些前端工具會幫助開發者將每一個合併成單一的大匯集，讓瀏覽器提取單一的大檔案，毋須逐個提取。當實際上需要很少的時候，傳送的資料就會太多。當需要作出一個修改的時候，需要重新載入的資料亦會太多。

這方法當然最不便的大多是參與的開發者。

## 3.4 Sharding

最後一個我想提及的效能提升技巧，常被稱為 “sharding”（域名分割）。簡單來說，就是儘可能以許多不同的主機域名處理網頁服務的不同部份。乍看之下或許很奇怪，但背後卻言之成理。

起始 HTTP 1.1 規格述明一個客戶端就每個主機允許使用最多兩個 TCP 連線。因此，為免有違規格，聰明的網站就創立出新的主機名稱。突然間，你的網站就能有更多的連線，減少了頁面載入時間。

隨著時間過去，該限制被移除，今日客戶端輕易就用上六至八個連線至每個主機名稱，但始終有個上限，因此網站沿用此技巧將連線數目推上。隨著物件數目持續上升－－就如先前所述－－連線數目之多只為了碓保 HTTP 表現得理想、自己的網站載入得快。現在網站用此技巧，單一站台就動軋用上逾 50 甚至超越 100 個連線並不罕見。httparchive.org 的最近統計就顯示，世界上首三十萬個 URL 需要 40(!) 個 TCP 連線以顯示網站，而趨勢顯示這只會隨著時間慢慢上升。

另一個原因，亦是將影像之類的資源放在獨立的主機名稱，而不加使用任何 Cookies，因為今時今日的 Cookies 大小能相當驚人。用上無 Cookies 的影像主機，從而大大減少 HTTP 要求的大小，說不定能提升效能！

以下圖片顯示，瀏覽瑞典一個熱門網站時的封包追蹤，看起來會是甚麼樣子。留意要求怎樣分布到數個主機名稱。

![image sharding at expressen.se](https://raw.githubusercontent.com/bagder/http2-explained/master/images/expressen-sharding.jpg)
