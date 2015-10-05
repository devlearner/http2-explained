# 2. 時至今日 HTTP

HTTP 1.1 已演變成一種互聯網上的一切幾乎都用上的通訊協定。能好好利用這一點的通訊協定以至基礎設施都下了巨大投資。程度之巨，今天要在 HTTP 上運行，總要比另起爐灶來得容易。

## 2.1 HTTP 1.1 很龐大

想當年創立 HTTP 並應用到世界的時候，人人都視之為一種相當簡單直截的通訊協定。但時間令人大跌眼鏡。HTTP 1.0 是 1996 年發布的 RFC 1945 中的規格，長達 60 頁。僅三年後，1999 年發布的 RFC 2616 所描述的 HTTP 1.1 已大大增幅到 176 頁。然而，當我們在 IETF 參與該規格的更新時，將其分開並轉成六份文件，總頁數大大增加（形成 RFC 7230 及其子系）。怎樣說，HTTP 1.1 很大，並包含一眾詳細、細微，以至許多選用的部份。

## 2.2 選項萬象

HTTP 1.1 的特性有許多微小的細節及可用的選項，以便日後延伸。這形成一種軟件生態：幾乎從沒有一個實作，真正實作了一切。而且「一切」究竟是甚麼，也難以說清。這導致了一種情況：過往一直少用的功能，很少被實作；而落實了這些功能的實作，卻又見得很少用途。

然後，當用戶端和伺服器開始多用這些功能時，這又產生了互通性問題。這特性的一個要例就是 HTTP Pipelining。

## 2.3 TCP 善用不足

HTTP 1.1 要真正善用所有 TCP 所帶來的功效及表現，一直顯得相當吃力。HTTP 用戶端和伺服器要富有創意，才能找到方法減少頁面載入時間。

與此同時，多年來不懈的嘗試，亦確認了 TCP 確實相當不容易被取締。因此，我們不斷努力改善 TCP 以及建基於上的通訊協定。

簡單來說，TCP 可以更好善用，以避免暫停，或那些本該能傳送或接收更多資料的時刻。以下各節標注其中一些短處。

## 2.4 Transfer sizes and number of objects

When looking at the trend for some of the most popular sites on the web today and what it takes to download their front pages, a clear pattern emerges. Over the years the amount of data that needs to be retrieved has gradually risen up to and above 1.9MB . What is more important in this context is that on average over a hundred individual resources are required to display each page.

As the graph below shows, the trend has been going on for a while and there is little to no indication that it'll change anytime soon. It shows the growth of the total transfer size (in green) and the total number of requests used on average (in red) to serve the most popular web sites in the world, and how they have changed over the last four years.

![transfer size growth](https://raw.githubusercontent.com/bagder/http2-explained/master/images/transfer-size-growth.png)

## 2.5 Latency kills

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/page-load-time-rtt-decreases.png" />

HTTP 1.1 is very latency sensitive, partly because HTTP Pipelining is still riddled with enough problems to remain switched off to a large percentage of users.

While we've seen a great increase in available bandwidth to people over the last few years, we have not seen the same level of improvements in reducing latency. High latency links, like many of the current mobile technologies, make it really hard to get a good and fast web experience even if you have a really high bandwidth connection.

Another use case that really needs low latency is certain kinds of video, like video conferencing, gaming and similar where there's not just a pre-generated stream to send out.

## 2.6. Head of line blocking

HTTP Pipelining is a way to send another request while waiting for the response to a previous request. It is very similar to queuing at a counter at the bank or in a super market. You just don't know if the person in front of you is a quick customer or that annoying one that will take forever before he/she is done: head of line blocking.

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/head-of-line-blocking.jpg" />

Sure you can be careful about line picking so that you pick the one you really believe is the correct one, and at times you can even start a new line of your own but in the end you can't avoid making a decision and once it is made you cannot switch lines.

Creating a new line is also associated with a performance and resource penalty so that's not scalable beyond a smaller number of lines. There's just no perfect solution to this.

Even today, 2015, most desktop web browsers ship with HTTP pipelining disabled by default.

Additional reading on this subject can be found for example in the Firefox [bugzilla entry 264354](https://bugzilla.mozilla.org/show_bug.cgi?id=264354).
