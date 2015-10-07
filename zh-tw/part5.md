# http2 概念

那麼，到底 http2 要達成甚麼？HTTPbis 小組制定出要實施的界限又是甚麼？

其實相當嚴格，同時亦有所限制團隊天馬行空的能力。

- 需要維持 HTTP 範例。HTTP 仍是客戶端透過 TCP 傳送要求到伺服器的通訊協定。

- http:// 和 https:// URL 不能改變。不能為此作出新的配置（Scheme）。正使用此等 URL 的內容實在太多，難以預期會改變。

- HTTP1 伺服器和客戶端在打下數十年仍會存在，需要能夠將它們代理（proxy）到 http2 伺服器。

- 其次，代理伺服器（Proxy）必須能以一比一對應 http2 功能到 HTTP 1.1 客戶端。

- 從通訊協定移除或減少選用部份。這倒不是一項真正的要求，卻是從 SPDY 及其 Google 團隊悟出來的一條真理。碓定了一切是強制的時候，就沒法子出現今天不實作、為他朝埋伏的情況。

- 不再有次要版本（minor version）。決定了客戶端和伺服器一是與 http2 相容，一是不相容。他朝若有需要延伸通訊協定或作修改，就會誕生 http3。http2 不再有次要版本。

## 5.1. 現有 URI 配置下的 http2

如前述，現有的 URI 配置（schemes）不得修改，因此 http2 需要沿用。有見今天 HTTP 1.x 已在使用這些配置，我們明顯需要一個方法 將通訊協定升級到 http2，或是要求伺服器使用 http2 而非較舊的通訊協定。

HTTP 1.1 已定義方法實施，就是 Upgrade: 標頭。它允許在舊有的通訊協定上提取要求時，讓伺服器以新的通訊協定傳回回應。代價是一來回行程（round-trip）。

該來回行程的負面影響（round-trip penalty）是不為 SPDY 團隊接受的，他們亦只實作了 TLS 上的 SPDY，因此他們開發了一種新的 TLS 延伸（TLS extension）作為大大縮減交涉（negotiation）的捷徑。應用這個名為 Next Protocol Negotiation 或 NPN 的延伸，伺服器告訴客戶端其了解的通訊協定，然後客戶端能繼而使用伺服器所偏好的通訊協定。

## 5.2. http2 中的 https://

http2 許多焦點集中於怎樣在 TLS 上正確運行。SPDY 只能在 TLS 上使用，因此有意見力爭讓 TLS 強制用於 http2，但未能達成具識，因此 http2 確立 TLS 為選用。不過，兩個舉足輕重的實作清楚表明只會實作 TLS 上的 http2：Mozilla Firefox 和 Google Chrome——當今兩個領先的網路瀏覽器。

選擇僅限 TLS 的原因包括尊重使用者的隱私，以及早期的度量顯示以 TLS 落實新的通訊協定有較高成功率。因為連接埠 80 上的普遍會假定為 HTTP 1.1，讓一些中間裝置干擾並破壞了其他通訊協定通訊時的流量。

強制 TLS 這個主題在郵寄清單和會面中都激起了許多人耍手搖頭，並惹起許多憤怒聲音——是福氣，還是禍水？這個主題相當易傳播——當你面對面向一位 HTTPbis 參與者發問這條問題時，務必小心！

同樣，一直激烈持續討論的是，http2 究竟應否制定一個使用 TLS 時強制的加密方式（ciphers）清單，還是該有個黑名單，還是在 TLS 「層面」上根本不該有任何要求，而留待 TLS 工作小組討論。規格結果指定 TLS 須至少為版本 1.2 並有加密套件（cipher suite）限制。

## 5.3. http2 在 TLS 上的交涉

Next Protocol Negotiation (NPN) 是用來與 TLS 伺服器交涉 SPDY 的通訊協定。袖於並不是當. As it wasn't a proper standard, it was taken through the IETF and ALPN came out of that: Application Layer Protocol Negotiation. ALPN is what is being promoted to be used for http2, while SPDY clients and servers still use NPN.

The fact that NPN existed first and ALPN has taken a while to go through standardization has lead to many early http2 clients and http2 servers implementing and using both these extensions when negotiating http2. Also, as NPN is what's used for SPDY and many servers offer both SPDY and http2 so supporting both NPN and ALPN on those servers make perfect sense.

ALPN primarily differs from NPN in who decides what protocol to speak. With ALPN the client tells the server a list of protocols in its order of preference and the server picks the one it wants, while with NPN the client makes that final choice.

## 5.4. http2 for http://

As mentioned briefly previously, for plain-text HTTP 1.1 the way to negotiate
http2 is by asking the server with an Upgrade: header. If the server speaks
http2 it responds with a “101 Switching” status and from then on it speaks
http2 on that connection. You of course realize that this upgrade procedure
costs a full network round-trip, but the upside is that an http2 connection
should be possible to keep alive and re-use to a larger extent than HTTP1
connections generally are.

While some browsers' spokespersons have stated they will not implement this means of speaking http2, the Internet Explorer team has expressed that they will, and curl already supports this.
