# 7. 延伸標準

通訊協定管理要求（mandates）一個接收端（receiver）必須讀取並忽略所有使用未知框架類型（unknown frame types）的未知框架。雙方因此能以躍點逐跳為基礎（hop-by-hop basis）來交涉使用新的框架類型，新類型的框架不允許改變狀態，亦不會受到流量限制（flow controlled）。

http2 根本應否允許延伸（extensions）這個主題過往在通訊協定開發時一直議論紛紛，意見一直反覆。到 draft-12 之後再來一次鐘擺，最後一槌定音——延伸再獲允許。

延伸未有成為實際通訊協定的一部份，而是會在核心通訊協定規格（core protocol spec）以外再作記載（documented）。在此時，已有兩種框格類型曾獲討論是否包含到通訊協定中，很大可能會成為首先以延伸傳送的框格。有見它們之前的熱門程度，並曾成為「原生」框架（“native” frames）的大熱，我會僅此聊作描述：

## 7.1. Alternative Services

隨著 http2 開始獲採用，有理由相信 TCP 連線將會更長久、保線連線（kept alive）的時間將會遠長於過往 HTTP 1.x 的連線。客戶端該更能隨心所欲，以單一連線至每一主機/網站，單一連線因此可能會開啟一段頗長的時間。

這將會影響 HTTP 負載平衡器（load balancers）的運作方式，某些情況下，一個網站或想公告（advertise）並建議客戶端連線到另一個主機。可能是出於效能的原故，或是網站需要下線以進行維護等工作。

伺服器就會送出 [Alt-Svc: 標頭](http://tools.ietf.org/html/draft-ietf-httpbis-alt-svc-07)（或 http2 的 ALTSVC 框架），告訴客戶端可使用替代服務（alternative service）。使用另一服務（service）、主機（host）及連接埠編號（port number）連線到相同內容的另一路由（route）。

客戶端應以非同步方式（asynchronously）嘗試連線到該服務，若成功的話，之後就僅使用該替代服務。

### 7.1.1. 隨機 TLS（Opportunistic TLS）

Alt-Svc 標頭允許以 http:// 提供內容的伺服器通知客戶端，相同的內容亦可用於 TLS 連線。

這項功能相當受爭議。此等連線將進行未驗證的 TLS（unauthenticated TLS），並不會在何處公告為「安全」（“secure”）、不會在介面中出現掛鎖，事實上亦沒有任何方式告訴使用者這並非普通的 HTTP，但這依然是隨機 TLS（opportunistic TLS），而一些人非常抗拒這個概念。

## 7.2. Blocked（被阻斷）

此類型的框架原意是由 http2 一方在資料仍有待傳送，但流量控制（flow control）又禁止傳送任何資料時送出一次。意念是，當實作收到此框架時，就會得悉實作中處理有欠妥善，又或傳送速度（transfer speeds）因而有欠理想。

引述 draft-12 移出此類框架成為延伸前的描述：

> BLOCKED 框架包含在此草槁版本中以促進實驗（experimentation）。倘若實驗意見反應（feedback）欠佳，可被移除。

