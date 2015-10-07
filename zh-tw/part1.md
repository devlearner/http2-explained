# 1. 從此說起

本文件從技術與通訊協定層面描述 http2 。一切源起於 2014 年四月 Daniel 在斯德哥爾摩演講的一份投影片簡報。此後經改編增訂，便成為這一份羅列細節與愜意闡釋的好一份文件。

RFC 7540 是 http2 規格定案的正式名稱，並在 2015 年五月十五日發布： http://www.rfc-editor.org/rfc/rfc7540.txt

本文件如有錯漏之處，乃自身不足所犯之故。還盼不吝賜教，將在更新版本勘正。

在本文件中，我將儘可能使用 "http2" 來描述這個新的通訊協定，儘管從純技術用語而言，正當的名稱該是 HTTP/2。此舉旨在促進行文流暢，並使文字更可觀可讀。

## 1.1 著者

我是 Daniel Stenberg （音譯：施格爾），在 Mozilla 工作。過去逾二十年，我在許多專案中參與開放源始碼與網路工作。也許我最為人熟悉的，就是擔任 curl 及 libcurl 的主要開放者。多年來，我一直參與 IETF HTTPbis 工作小組的工作，其間一直與時並進，了解 HTTP 1.1 更新的最新發展，並參與制定 http2 標準化工作。

  電郵： daniel@haxx.se

  Twitter： [@bagder](https://twitter.com/bagder)

  網址： [daniel.haxx.se](http://daniel.haxx.se/)

  部落格： [daniel.haxx.se/blog](http://daniel.haxx.se/blog/)

## 1.2 協助

若文中發現疏漏謬誤，請將有關段落的更正本發送給我，以作修正版。襄助的各方，將給予適當鳴謝！希望本文件能隨時間更臻完善。

本文件可在 [http://daniel.haxx.se/http2](http://daniel.haxx.se/http2) 找到

## 1.3 版權

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/creative-commons.png" />

This document is licensed under the Creative Commons Attribution 4.0 license: http://creativecommons.org/licenses/by/4.0/

## 1.4 修改

本文件初版在 2014 年四月二十五日發布。以下列出最近版本所作過的最主要變動。

### 版本 1.13

- 將此文件的主版本轉換成 Markdown 語法
- 13: 提及更多資源，更新了多個連結和描述
- 12: 按其草稿內容更新了 QUIC 描述 
- 8.5: 更新至目前數字
- 3.4: 現在平均是 40 個 TCP 連線
- 6.4: 更新以反映規格所述

### 版本 1.12

- 1.1: HTTP/2 現已是正式 RFC 
- 6.5.1: 連結至 HPACK RFC 
- 9.1: 提及 Firefox 36+ 的 http2 設置參數
- 12.1: 加入篇幅描述 QUIC 

### 版本 1.11

- 許多行文修繕，蒙各貢獻者不吝指正
- 8.3.1: 提及 nginx 和 Apache httpd 特定活動

### 版本 1.10

- 1: 通訊協定已獲「首肯」
- 4.1: 更新用詞：2014 已是去年 
- 首頁: 加入了影像並稱之為 “http2 explained”（http2 解說），修正了連結
- 1.4: 加入文件「修改」一節
- 修正許多拼字及文法錯誤
- 14: 加入鳴謝 bug 報告者
- 2.4: HTTP 增長圖表更好的文字標籤
- 6.3: 更正了多工列車上的車卡順序
- 6.5.1: HPACK draft-12 

### 版本 1.9

- 更新至 HTTP/2 draft-17 及 HPACK draft-11  
- 加入章節 "10. http2 in Chromium" (== 多一頁了)  
- 許多拼字修正
- 現在有 30 個實作
- 8.5: 加入一些目前應用的數字
- 8.3: 一併提及 internet explorer
- 8.3.1 加入「欠奉的實作」
- 8.4.3: 提及 TLS 亦增加成功率
