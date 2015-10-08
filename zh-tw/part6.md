# 6. http2 通訊協定

過往帶到現今的背景、歷史和用意就此打住。現在來探討一下這個通訊協定的細節：創立 http2 的位元和概念。

## 6.1. 二進位

http2 是個二進位（binary）通訊協定。

先齊來深思一下。若你先前曾接觸互聯網通訊協定，你現在很可能直覺地會抗拒這個選擇，並娓娓道出理據，說明基於文字/ascii 的通訊協定怎樣較勝一籌，因為人們能透過 telnet 自製要求等。

http2 二進位能讓架構（framing）容易許多。在 HTTP 1.1，或普遍在文字型通訊協定（text based protocols）中，整理出框架（frames）的開端和結尾是其中一項最為複雜的事情。透過遠離選用的空白字元（white spaces）和編寫相同訊息時的不同方式，實作就變得更簡易。

再者，這樣做就能更易區分架構（framing）與實際各個通訊協定部件（protocol parts）——在 HTTP1 中這是混作一團，讓人困惑的。

由於通訊協定備用壓縮功能，並通常在 TLS 上運行，文字的價值就大大減少——反正線路上根本看不見文字。我們必須習慣使用 Wireshark 之類的偵測器（inspector），徹底了解在通訊層面上的 http2 究竟在發生甚麼。

要對此通訊協定進行偵錯（Debugging）大抵將需要使用 curl 等的工具，或使用 Wireshark 的 http2 剖析器（dissector）之類來分析該網路串流（network stream）。

## 6.2. 二進位格式

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/frame-layout.png" />

http2 傳送二進位框架（binary frames）。框架有不同類型，傳送時格式一致：

長度（Length）、型別（Type）、旗標（Flags）、串流識別碼（Stream Identifier）和框架裝載（frame payload）。

http2 規格中定義了十種不同的框格（frames），對應 HTTP 1.1 功能最基本的兩種大抵是 DATA 和 HEADERS。之後會再深入描述。

## 6.3. 多工串流（Multiplexed streams）

上一節描述二進位框架格式（binary frame format）時提及的串流識別碼（Stream Identifier），將以 http2 傳送的每一個框架（frame）關聯至一個「串流」。所謂串流是一個邏輯關聯（logical association）——在一個 http2 連線內客戶端與伺服器交換的一個獨立雙向的框架序列（sequence）。

一個單一的 http2 連線能包含多個同時開啟的串流（concurrently open streams），兩個端點（endpoint）皆能從多重串流（multiple streams）交錯框架（interleaving frames）。串流（Streams）能單向建立並使用，或由客戶端或伺服器共享，而且兩個端點（endpoint）皆能關閉串流。在一個串流（stream）之內傳送的框架（frames）順序是重要的。接收端（Recipients）會按其接收順序處理框架（frames）。

多工處理（Multiplexing）串流（streams）表示來自許多串流（streams）的包裝（packages）會在同一連線上混合（mixed）。兩列（或以上）獨立的列車資料變成一條，再在另一端重新分開。兩列列車來了：

![one train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-justin.jpg)
![another train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-ikea.jpg)

兩列列車在同一連線上多工（multiplexed）:

![multiplexed train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-multiplexed.jpg)

## 6.4. 優先順序（Priorities）和相依性（dependencies）

每一串流（stream）同時亦有一個優先順序（priority）［亦稱為「權數」（“weight”）］，用來告訴對等（peer）該把哪一些串流（streams）視為最重要，以防因資源限制迫使伺服器需要選擇先傳送哪一些串流（streams）。

使用 PRIORITY 框格（frame），客戶端亦能向伺服器指定此串流（stream）相依於哪一條別的串流（stream）。這允許客戶端建置一個優先順序樹狀結構（priority “tree”），當中數個「子串流」（“child streams”）能依存在「父串流」（“parent streams”）的完成。

優先順序權數（priority weights）和相依性（dependencies）可以在執行階段（run-time）動態改變，這該可讓瀏覽器確定當使用者向下捲動滿是影像的頁面時，它能指定哪些影像是最重要的；或在切換分頁時，它能指定優先處理（prioritize）突然取得集點的一組新的串流。

## 6.5. 標頭壓縮（Header compression）

HTTP 是個無狀態的通訊協定（state-less protocol）。簡而言之，這表示每一個要求都需要附帶足夠詳細資料，足以讓伺服器處理該要求，而毋需伺服器儲起各個先前要求的許多資訊和中繼資料。由於 http2 並不變更任何這些範例（paradigms），它亦需跟隨。

這使 HTTP 相當重複。當客戶端從同一伺服器要求許多資源，比如是一個網頁上有許多影像，就會有一系列大量的要求，看起來都大致相同。幾乎一式一樣的一系列似乎可讓壓縮一展所長。

隨著每一頁面的物件數目上升，如較早前所述，Cookies 的使用以至要求的大小亦隨時間一直增加。Cookies 亦需要在所有要求中包含，而在許多要求中卻大都是相同的。

HTTP 1.1 要求大小隨時間增加，有時甚至大於 TCP 窗口始值（initial TCP window），使這些要求傳送得很慢，因為需要待伺服器傳回一個 ACK 才能送出整個要求，導致需要一整個來回行程（full round-trip）。另一需要壓縮的理據。

### 6.5.1. 壓縮亦考究

HTTPS 及 SPDY 壓縮被發現易受 [BREACH](http://en.wikipedia.org/wiki/BREACH_%28security_exploit%29) 和 [CRIME](http://en.wikipedia.org/wiki/CRIME) 攻擊。透過插入已知的文字到串流（stream）中並觀察輸出的變更，攻擊者可以得悉送出的內容。

為一個通訊協定中的動態內容進行壓縮，而毋須日後變得易受上述其中一些攻擊，需要思量和小心考慮。這是 HTTPbis 團隊嘗試去做的事。

就此登場：[HPACK](http://www.rfc-editor.org/rfc/rfc7541.txt)——Header Compression for HTTP/2——顧名思義，是一個為 http2 標頭而特製的壓縮格式，嚴格來說是由另一個互聯網草稿制定。這個新格式，連同其他對策——例如一位元會要求中繼（intermediaries）不要壓一個指定的標頭，框架（frames）的選用填補（optional padding）——該讓入侵此壓縮變得更困難。

引述 Roberto Peon（音譯：潘樂途）（HPACK 創立人之一）：

> “HPACK was designed to make it difficult for a conforming implementation to
> leak information, to make encoding and decoding very fast/cheap, to provide
> for receiver control over compression context size, to allow for proxy
> re-indexing (i.e. shared state between frontend and backend within a proxy),
> and for quick comparisons of huffman-encoded strings”.

## 6.6. 重設——再想法

HTTP 1.1 其中一個弊處是，當一個 HTTP 訊息以一個 Content-Length 特定大小送出時，要停止並非如說般容易。當然你通常能（並非一定）中斷該 TCP 連線， but that
then comes as the price of having to negotiate a new TCP handshake again.

A better solution would be to just stop the message and start anew. This can be done with http2's RST_STREAM frame which will help in preventing wasted bandwidth and avoid having to tear down any connection.

## 6.7. Server push

This is the feature also known as “cache push”. The idea here is that if the client asks for resource X the server may know that the client then also most likely want resource Z and sends that to the client without it asking for it. It helps the client to put Z into its cache so that it'll be there when it wants it.

Server push is something a client explicitly must allow the server to do and even if a client does that, it can at its own choice swiftly terminate a pushed stream with RST_STREAM should it not want a particular one.

## 6.8. Flow Control

Each individual stream over http2 has its own advertised flow window that the other end is allowed to send data for. If you happen to know how SSH works, this is very similar in style and spirit.

For every stream both ends have to tell the peer that it has more room to fit incoming data in, and the other end is only allowed to send that much data until the window is extended. Only DATA frames are flow controlled.

