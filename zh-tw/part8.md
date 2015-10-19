# 8. http2 新世界

當世界採用 http2 時，會是一番怎樣的新景象？會否廣泛採用？

## 8.1. http2 與我何干？

現在，http2 尚未獲廣泛部署或使用。他朝如何，無人能預知。君可見 SPDY 怎樣獲採用，加上過去與現在的實驗，從而可猜猜算算。

http2 減少了所需的網路來回行程（network round-trips）數目，並以多工（multiplexing）及快速捨棄不要的串流（streams）迴避了 head of line blocking（龍頭阻滯）的吊詭。

它允許大量平行串流（parallel streams），比起當今大部份域名分割（sharding）的網站，有過之而無不及。

當好好使用串流上的優先順序（priorities）時，客戶端大可能實實在在優先得到重要資料，才到較次要的資料。總而言之，我認為這很大機會能提升網頁載入速度，讓網站回應更靈敏。一句作結：這會是更佳的網站體驗。

且拭目以待到底快了多少、能有多少提升，我想暫未能定論。首先，這個科技仍屬非常早期，甚至仍未見客戶端和伺服器度身而做的實作，好把握所有這個新的通訊協定所帶來的性能。

## 8.2. http2 怎樣影響網路開發？

歷年來，網路開發者（web developers）及網路開發環境（web development environments）搜羅了一整個工具箱的小技巧與工具，以緩解（work around）使用 HTTP 1.1 時的問題——還記得在本文件開端時我勾劃了其中好一些，作為支持 http2 的理由（justification）。

許多這些對策（workarounds）今時今日各個工具及開發者已不假思索應用，然而卻很可能有損 http2 的效能，至少並不能完全體現 http2 帶來新的超能力。Spriting（圖像織結）及 inlining（內述法）大抵不該在 http2 中使用。而 Sharding（域名分割）將不利於 http2 使用較少連線時所帶來的好處。

其中一個問題當然是，網站與網路開發者需要開發與部署在一個世界，至少短期內同時會有 HTTP1.1 和 http2 客戶端的使用者；要為所有使用者提供最大效能，又毋需提供兩個不同的前端（front-ends），將相當具挑戰。

僅此上述理由，我認為 http2 能發揮所有潛能，還需待一段時間。

## 8.3. http2 實作

在一份這樣的文件中嘗試記載（document）特定的實作，無疑只會失敗告終、徒勞無功，不消一會兒就會顯得過時。因此，我將會以較廣的層面敘述大體情況，讀者可自行參閱 http2 網站上的[實作清單](https://github.com/http2/http2-spec/wiki/Implementations)。

一開始已有一定數量的實作，隨著 http2 工作的進行，數目隨時間上升。執筆之時名單上已有超過 40 個實作，大部份實作了 http2 的終定版本。

### 8.3.1 瀏覽器

Firefox 瀏覽器一直緊貼最新的草稿，Twitter 亦跟上步伐以 http2 提供服務。Google 在 2014 年四月其間在數個服務提供的測試伺服器上提供 http2 支援，並自 2014 年五月起在 Chrome 的開發版本中提供 http2 支援。Microsoft 在其下一版本的 Internet Explorer 技術預覽版（tech preview）中展示了 http2 支援。Safari（隨附於 iOS 9 及 Mac OS X El Capitan）及 Opera 都表示將提供 http2 支援。

### 8.3.2 伺服器

現在已有許多 http2 伺服器的實作。

熱門的 Nginx 伺服器提供 http2 支援，初見於
[1.9.5](https://www.nginx.com/blog/nginx-1-9-5/) 版本——已在 2015 年九月二十二日釋出（取代 SPDY 模組，因此無法在同一伺服器報行個體並行操作）。

Apache 的 httpd 伺服器備有一個 http2 模組，仍在開發階段，名為
[mod_http2](https://httpd.apache.org/docs/2.4/mod/mod_http2.html)，預期會隨著未來的 2.4.17 版本釋出。

[H2O](https://h2o.examp1e.net/)、[Apache Traffic
Server](http://trafficserver.apache.org/)、[nghttp2](https://nghttp2.org/)、[Caddy](http://caddyserver.com/) 及
[LiteSpeed](https://www.litespeedtech.com/products/litespeed-web-server/overview)
各有發行具備 http2 的伺服器。

### 8.3.3 其他

curl 及 libcurl 支援不安全（insecure）http2 ，以及使用多個不同 TLS 程式庫（libraries）之一所建立 TLS 為基礎的 http2。

Wireshark 支援 http2。絕佳的工具來分析 http2 網路流量（network traffic）。

## 8.4. http2 紛紜

在此通訊協定開發其間，一直已有反覆討論。當然，總有人相信這通訊協定結果完全是個錯誤。在此，我想提及一些較常見的控訴及反駁：

### 8.4.1. 「這通訊協定是由 Google 設計或製造的」

類似論調言下之意，是指世界將因此變得依賴或受 Google 控制。這並不真確。此通訊協定是在 IETF 中開發，形式與過去三十年各個通訊協定的開發無異。不過，我們都讚賞並認可（recognize and acknowledge）Google 在 SPDY 的傑作，不只證明了這是部署新通訊協定的可行方法，亦提供了數字闡述得到的提升。

Google 已公開[宣布](http://blog.chromium.org/2015/02/hello-http2-goodbye-spdy-http-is_9.html)將會在 2016 年移除 Chrome 對 SPDY 和 NPN 的支援，並促請伺服器遷移到 HTTP/2。

### 8.4.2. 「這通訊協定僅對瀏覽器有用」

某程度上是正確的。http2 開發背後其中一個主要的推動力，是修正 HTTP pipelining（流水線）。若你的使用案例（use case）本來就不需要 pipelining（流水線），很可能 http2 對你也沒有很大得益。當然這不是通訊協定中惟一的改善，但確是相當大的一個。

當各個服務開始發現在單一連線上多工串流（multiplexed streams）所帶來的完整能力和威力，我想將會見到更多 http2 的應用程式使用（application use）。

小型 REST API 及較簡單的 HTTP 1.x 程式編寫使用（programmatic uses）或許看不見改用 http2 有很大得益。但同樣，對大部份使用者而言 http2 也不見得有太大壞處。

### 8.4.3. 「這通訊協定對大型網型才有用」

此言差矣。多工能力（multiplexing capabilities）將有助大大提升高延遲連線（high latency connections）上的體驗，對許多較小型的網站沒有廣大的地理分佈（wide geographical distributions）非常有用。大型網站通常已是較快、分佈得較廣、提供較短的來回時間（round-trip times）予使用者。

### 8.4.4. 「使用 TLS 拖慢速度」

某程度上或許沒錯。TLS 交握（handshake）的確增加了一點額外的工夫，但現在已見現有及持續努力更進一步減少 TLS 所需要的來回行程（necessary round-trips）。在線路（wire）上進行 TLS 而非純文字的額外負荷（overload）並不顯著，或清晰可見相同流量模式（same traffic pattern）較在不安全通訊協定（non-secure protocol）上花費高出多少 CPU 及功能（power）。數字及實際影響因意見（opinions）及度量（measurements）各有不同。作為例子，可參閱 [istlsfastyet.com](https://istlsfastyet.com/) 作為其中一個資訊來源。

電訊及其他網路營辦商——例如 ATIS Open Web
Alliance 中的成員——表示[需要未加密的流量（unencrypted
traffic）](http://www.atis.org/openweballiance/docs/OWAKickoffSlides051414.pdf)以提供快取（caching）、壓縮（compression）及其他技巧，好透過衛星、航空等提供快速的網路體驗。http2 並不強制使用 TLS，因此不該混為一談。

許多互聯網使用者都表示偏好 TLS 更廣泛使用，我們該協助保障使用者的隱私。

實驗亦顯示，使用 TLS 較在連接埠 80 上實作新的純文字通訊協定（plain-text protocols）較為成功，因為外面實在有太多中間裝置（middle boxes）將連接埠 80 誤作 HTTP 1.1（有時亦頗相似 HTTP）而作出干擾。

最後，幸虧 http2 單一連線的多工串流（multiplexed streams over a single connection），正常瀏覽器使用案例（normal browser use cases）終歸仍能進行遠遠較少的 TLS 交握（handshakes）並表現得較快，相對於仍在使用 HTTP 1.1 時的 HTTPS。

### 8.4.5. 「非 ASCII 者拉倒」

Yes, we like being able to see protocols in the clear since it makes debugging and tracing easier. But text based protocols are also more error prone and open up for much more parsing and parsing problems.

If you really can't take a binary protocol, then you couldn't handle TLS and compression in HTTP 1.x either and its been there and used for a very long time.

### 8.4.6. 「比 HTTP/1.1 也不是快許多」

This is of course subject to debate and discussions on how to measure what faster means, but already in the SPDY days many tests were made that proved faster browser page loads (like ["How Speedy is SPDY?"](https://www.usenix.org/system/files/conference/nsdi14/nsdi14-paper-wang_xiao_sophia.pdf) by people at University of Washington and ["Evaluating the Performance of SPDY-enabled Web Servers"](http://www.neotys.com/blog/performance-of-spdy-enabled-web-servers) by Hervé Servy) and such experiments have been repeated with http2 as well. I'm looking forward to seeing more such tests and experiments getting published. A [basic first test made by httpwatch.com](http://blog.httpwatch.com/2015/01/16/a-simple-performance-comparison-of-https-spdy-and-http2) might imply that HTTP/2 holds its promises.

### 8.4.7. “It has layering violations”

Seriously, that's your argument? Layers are not holy untouchable pillars of a global religion and if we've crossed into a few gray areas when making http2 it has been in the interest of making a good and effective protocol within the given constraints.

### 8.4.8. “It doesn't fix several HTTP/1.1 shortcomings”

That's true. With the specific goal of maintaining HTTP/1.1 paradigms there were several old HTTP features that had to remain. Such as the common headers that also include the often dreaded cookies, authorization headers and more. But by the upside of maintaining these paradigms is that we got a protocol that is possible to deploy without an inconceivable amount of upgrade work that requires fundamental parts to be completely replaced or rewritten. Http2 is basically just a new framing layer.

## 8.5. Will http2 become widely deployed?

It is too early to tell for sure, but I can still guess and estimate and that's what I'll do here.

The naysayers will say “look at how good IPv6 has done” as an example of a new protocol that's taken decades to just start to get widely deployed. http2 is not an IPv6 though. This is a protocol on top of TCP using the ordinary HTTP update mechanisms and port numbers and TLS etc. It will not require most routers or firewalls to change at all.

Google proved to the world with their SPDY work that a new protocol like this can be deployed and used by browsers and services with multiple implementations in a fairly short amount of time. While the amount of servers on the Internet that offer SPDY today is in the 1% range, the amount of data those servers deal with is much larger. Some of the absolutely most popular web sites today offer SPDY.

http2, based on the same basic paradigms as SPDY, I would say is likely to be deployed even more since it is an IETF protocol. SPDY deployment was always held back a bit by the “it is a Google protocol” stigma.

There are several big browsers behind the roll-out. Representatives from Firefox, Chrome, Safari, Internet Explorer and Opera have expressed they will ship http2 capable browsers and they have shown working implementations.

There are several big server operators that are likely to offer http2 soon, including Google, Twitter and Facebook and we hope to see http2 support soon get added to popular server implementations such as the Apache HTTP Server and nginx. H2o is a new blazingly fast HTTP server with http2 support that shows potential.

Some of the biggest proxy vendors, including HAProxy, Squid and Varnish have expressed their intentions to support http2.

All through-out 2015, the amount of traffic that is http2 has been increasing. In early September, Firefox 40 usage was at 13% out of all HTTP traffic and 27% out of all HTTPS traffic, while Google see roughly 18% incoming HTTP/2. It should be noted that Google runs other new protocol experiments as well (see QUIC in 12.1) which makes the http2 usage levels lower than it could otherwise be.

