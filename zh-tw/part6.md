# 6. http2 通訊協定

過往帶到現今的背景、歷史和用意就此打住。現在來探討一下這個通訊協定的細節：創建 http2 的位元和概念。

## 6.1. 二進位

http2 是個二進位（binary）通訊協定。

先齊來深思一下。若你先前曾接觸過互聯網通訊協定，你現在很可能直覺地會抗拒這個選擇，並娓娓道出理據，說文字/ascii 型的通訊協定怎樣較勝一籌，因為人們能透過 telnet 自製要求云云。

http2 二進位使架構（framing）容易許多。在 HTTP 1.1，或普遍在文字型通訊協定（text based protocols）中，整理出框架（frames）的開端和結尾是其中一項最為複雜的事情。透過遠離選用的空白字元（white spaces）和不同編寫相同訊息的方式，實作就變得更簡易。

再者，這樣更容易把實際各個通訊協定部件（protocol parts）與架構（framing）區分——這在 HTTP1 中是混作一團，讓人困惑的。

由於通訊協定備有壓縮功能，並通常在 TLS 上運行，文字的價值就大大減少——反正線路上根本看不見文字。我們必須習慣使用 Wireshark 之類的偵測器（inspector），徹底了解在通訊層面上 http2 究竟在發生甚麼。

要對此通訊協定進行偵錯（Debugging）大抵會需要使用 curl 等工具，或使用 Wireshark 的 http2 剖析器（dissector）之類來分析該網路串流（network stream）。

## 6.2. 二進位格式

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/frame-layout.png" />

http2 傳送的是二進位框架（binary frames）。有不同類型的框架可以送出，全部的設置都相同：

長度（Length）、型別（Type）、旗標（Flags）、串流識別碼（Stream Identifier）和框架裝載（frame payload）。

http2 規格中定義了十種不同的框格（frames），而對應 HTTP 1.1 功能的最基本兩種大抵是 DATA 和 HEADERS。之後會再深入探討。

## 6.3. 多工串流（Multiplexed streams）

上一節描述二進位框架格式（binary frame format）的時候所提及的串流識別碼（Stream Identifier），會把每個以 http2 傳送的框架（frame）關聯至一個「串流」。一個串流是一個邏輯關聯（logical association）。一個客戶端與伺服器在一個 http2 連線內交換的獨立、雙向框架序列（sequence）。

單一 http2 連線能包含多個同時開啟的串流（concurrently open streams），兩個端點（endpoint）皆能從多重串流（multiple streams）交錯框架（interleaving frames）。串流能單向建立並使用，或是由客戶端或伺服器共享，而且兩個端點皆能關閉串流。在一個串流之內傳送的框架順序是重要的。接收端（Recipients）會按照其接收順序來處理框架。

串流的多工處理（Multiplexing），是指不同串流的包裝（packages）會在同一連線上混合（mixed）。兩列（或更多）獨立列車的資料會變成一條，再在另一端重新分開。齊來看看這兩列列車：

![one train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-justin.jpg)
![another train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-ikea.jpg)

就這樣在同一連線上多工（multiplexed）:

![multiplexed train](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-multiplexed.jpg)

## 6.4. 優先順序（Priorities）和相依性（dependencies）

每一串流同時亦有一個優先順序（priority）［亦稱為「權數」（“weight”）］，用來告訴對等（peer）該把哪一些串流視為最重要，以備因資源限制時，迫使伺服器選擇先要傳送哪一些串流。

使用 PRIORITY 框格，客戶端亦能向伺服器指定串流相依於哪一條別的串流（stream）。這允許客戶端建置一個優先順序樹狀結構（priority “tree”），當中數個「子串流」（“child streams”）能依存在「父串流」（“parent streams”）的完成要求。

優先順序權數（priority weights）和相依性（dependencies）可以在執行階段（run-time）動態改變，這樣該可讓瀏覽器確定當使用者向下捲動滿是影像的頁面時，它能指定哪些影像是最重要的；或在切換分頁時，它能指定優先處理（prioritize）突然取得焦點的一組新的串流。

## 6.5. 標頭壓縮（Header compression）

HTTP 是個無狀態的通訊協定（state-less protocol）。簡而言之，每一個要求都需要附帶足夠的詳細資料，足以讓伺服器處理該要求，而毋需伺服器儲起先前各個要求的一眾資訊和中繼資料。由於 http2 並不改變任何這些範例（paradigms），因此亦需跟隨。

這使 HTTP 相當重複。當客戶端從同一伺服器要求許多資源時，比如一個網頁上有許多影像，就會有一系列大量的要求，看起來都大致相同。幾乎一式一樣的系列似乎可讓壓縮一展所長。

隨著每一頁面的物件數目上升，如較早前所述，Cookies 的使用以至要求的大小亦隨時間一直增加。Cookies 亦需要在所有要求中包含，而在許多要求中卻大都是相同的。

HTTP 1.1 要求大小隨時間增加，有時甚至大於 TCP 窗口初值（initial TCP window），使這些要求傳送得很慢，因為需要待伺服器傳回一個 ACK 才能送出整個要求，導致需要一整個來回行程（full round-trip）。壓縮的理據又添一筆。

### 6.5.1. 壓縮亦考究

HTTPS 及 SPDY 壓縮被發現易受 [BREACH](http://en.wikipedia.org/wiki/BREACH_%28security_exploit%29) 和 [CRIME](http://en.wikipedia.org/wiki/CRIME) 攻擊。透過插入已知的文字到串流（stream）並觀察輸出的變化，攻擊者可以得悉送出的內容。

為一個通訊協定中的動態內容進行壓縮，而毋須日後變得易受上述一些攻擊，需要思量和小心考慮。這是 HTTPbis 團隊嘗試去做的事。

就此登場：[HPACK](http://www.rfc-editor.org/rfc/rfc7541.txt)：Header Compression for HTTP/2——顧名思義，是一個為 http2 標頭而特製的壓縮格式，嚴格來說是由另一個互聯網草稿制定。這個新格式，連同其他對策——例如有一位元會要求中繼（intermediaries）不要壓縮一個指定的標頭、框架的選用填補（optional padding）——該可讓入侵此壓縮格式變得更困難。

引述 Roberto Peon（音譯：潘樂途）（HPACK 創立人之一）的話：

> HPACK 的設計在讓一個合格（conforming）的實作難以洩露資訊、
> 讓編碼與解碼非常快速/不耗費資源、讓接收端（receiver）能以控制
> 壓縮內容大小（compression context size）、允許代理伺服器（proxy）
> 重新編製索引（re-indexing）［即在代理伺服器內前端（frontend）與後端（backend）的共用狀態（shared state）］，
> 以及快速比較 Huffman 編碼的字串（huffman-encoded strings）。

## 6.6. 重設——再想法

HTTP 1.1 其中一個弊處是，當一個 HTTP 訊息以一個 Content-Length 特定大小送出時，要停止並非如說般容易。當然你通常能（並非一定）中斷該 TCP 連線，但之後需要重新交涉（negotiate）一個新的 TCP 交握（TCP handshake），就是一個代價。

較好的方案，就是只終止訊息，重新開始。http2 裡可使用 RST_STREAM 框架，有助防止浪費頻寬，避免需要中斷（tear down）任何連線。

## 6.7. Server push（伺服器推入）

這項功能又名為「快取推入」（“cache push”）。背後的意念是，當客戶端要求資源甲時，伺服器可以知道客戶端之後很可能亦想要資源丙，故未待客戶端要求時就一併送出。這有助客戶端把丙放進快取中，需要用到時便能信手拈來。

Server push（伺服器推入）必須由客戶端明確允許伺服器這樣做；即使如此，客戶端亦可以自行決定以 RST_STREAM 迅速中止一個推入串流（pushed stream），倘若它不願接收特定這個。

## 6.8. 流量控制（Flow Control）

每條 http2 的個別串流都有其自己公告（advertised）的流量窗口（flow window），允許另一端傳送多少資料。若你湊巧知道 SSH 怎樣運作，兩者的形式與理念都十分相似。

就每一串流，兩端須要告訴對等自己有更多空間接收傳入的資料，而另一端僅允許傳送該數量的資料，直到窗口得到延伸（extended）為止。只有 DATA 框架會受到流量控制。

