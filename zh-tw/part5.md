# http2 概念

那麼，到底 http2 要達成甚麼？ HTTPbis 小組制定出甚麼要實施的界限？

這其實相當嚴格，同時亦有數項限制團隊創新的能力。

- 需要維持 HTTP 範例（paradigm）。HTTP 仍是客戶端透過 TCP 傳送要求到伺服器的通訊協定。

- http:// 和 https:// URL 不能改變。不能為此作出新的配置（Scheme）。正使用此等 URL 的內容實在太多，難以預期它們會改變。

- HTTP1 伺服器和客戶端在打下數十年仍會存在，需要能夠將它們代理（proxy）到 http2 伺服器。

- 其次，代理伺服器（Proxy）必須能把 http2 功能以一比一對應到 HTTP 1.1 客戶端。

- 從通訊協定移除或減少選用部份。這倒不是一項真正的要求，卻是從 SPDY 及其 Google 團隊悟出來的一條真理。碓定了一切是強制的時候，就沒法子出現今天不實作、為他朝埋伏的情況。

- 不再有次要版本（minor version）。決定了客戶端和伺服器一是與 http2 相容，一是不相容。他朝若有需要延伸通訊協定或作修改，就會誕生 http3。http2 中不再有次要版本。

## 5.1. 現有 URI 配置下的 http2

如前述，現有的 URI 配置（schemes）不得修改，http2 需要沿用。有見今天 HTTP 1.x 已在使用這些配置，我們明顯需要一個方法將通訊協定升級到 http2，或是要求伺服器使用 http2 而非較舊的通訊協定。

HTTP 1.1 對實施已有定義的方法，就是 Upgrade: 標頭。它允許在舊有的通訊協定上提取要求時，讓伺服器以新的通訊協定傳回回應。代價是一來回行程（round-trip）。

該來回行程的負面影響（round-trip penalty）是不為 SPDY 團隊接受的，他們亦只實作了 TLS 上的 SPDY，因此他們開發了一種新的 TLS 延伸（TLS extension）作為大大縮減交涉（negotiation）的捷徑。應用這個名為 Next Protocol Negotiation 或 NPN 的延伸，伺服器告訴客戶端了解些甚麼通訊協定，客戶端繼而能使用所偏好的通訊協定。

## 5.2. http2 中的 https://

http2 許多焦點集中於怎樣在 TLS 上正確運行。SPDY 只能在 TLS 上使用，因此有意見力爭讓 TLS 強制用於 http2，但未能達成具識，因此 http2 確立 TLS 為選用。不過，兩個舉足輕重的實作卻清楚表明只會實作 TLS 上的 http2：Mozilla Firefox 和 Google Chrome——當今兩個領先的網路瀏覽器。

選擇僅限 TLS 的原因包括尊重使用者的隱私，以及早期的度量顯示以 TLS 落實新的通訊協定有較高成功率。因為連接埠 80 上通過的普遍會假定為 HTTP 1.1，讓一些中間裝置干擾並破壞了使用其他通訊協定通訊時的流量。

強制 TLS 這個主題在郵寄清單和會面中都激起了許多人耍手搖頭，並惹起許多憤怒聲音——是福氣，還是禍水？這個主題相當易傳播——當你面向一位 HTTPbis 參與者發問這條問題時，務必小心！

同樣，一直激烈持續討論的是，http2 究竟應否制定一個使用 TLS 時強制的加密方式（ciphers）清單，還是該有個黑名單，還是在 TLS 「層面」上根本不該有任何要求，而留待 TLS 工作小組討論。規格結果指定 TLS 須至少為版本 1.2 並有加密套件（cipher suite）限制。

## 5.3. http2 在 TLS 上的交涉

Next Protocol Negotiation (NPN) 是用來與 TLS 伺服器交涉 SPDY 的通訊協定。由於並不是正當標準，拿到 IETF 來，結果變成了 ALPN：Application Layer Protocol Negotiation。推廣用到 http2 上的就是 ALPN，而 SPDY 客戶端和伺服器仍舊使用 NPN。

由於 NPN 率先面世，而 ALPN 經過標準化過程需時，使許多早期的 http2 客戶端和 http2 伺服器開發並使用同時這兩種延伸來交涉 http2。再者，有見 NPN 應用於 SPDY 而許多伺服器同時提供 SPDY 和 http2，在這些伺服器上同時支援 NPN 和 ALPN 亦言之成理。

ALPN 與 NPN 的主要分別在於由哪一方來決定使用甚麼通訊協定來交談。在 ALPN，客戶端告訴伺服器一個按其偏好順序排列的通訊協定清單，讓伺服器從中挑選其一；而在 NPN，客戶端作出最終選擇。

## 5.4. http2 中的 http://

剛才略有提及，在純文字 HTTP 1.1 中交涉 http2 的方法，是以一個 Upgrade: 標頭詢問伺服器。若果伺服器能以 http2 交談，就會回應以 “101 Switching” 狀況，自此處起該連線就會轉為 http2。想當然你會留意到此升級程序會花費一個完整的網路來回行程（network round-trip），但好處是一個 http2 連線比起 HTTP1 連線普遍更能保持連線（keep alive）及重用（re-use）。

當一些瀏覽器的發言人表明不會支援這樣交談 http2 的方式，Internet Explorer 團隊就表示會支援；而 curl 已經支援。
