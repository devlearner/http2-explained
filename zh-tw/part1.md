# 1. 從此說起

本文件從技術與通訊協定層面描述 http2 。一切源起於 2014 年四月 Daniel 在斯德哥爾摩演講的一份投影片簡報。
拙作此後經改編增訂，便成為這一份羅列細節與愜意闡釋的好一份文件。

RFC 7540 是 http2 規格定案的正式名稱，並在 2015 年五月十五日發布： http://www.rfc-editor.org/rfc/rfc7540.txt

本文件如有錯漏之處，乃自身不足所犯之故。還盼不吝賜教，將在更新版本勘正。

在本文件中，我將儘可能使用 "http2" 來描述這個新的通訊協定，儘管從純技術用語而言，正當的名稱是 HTTP/2。
此舉旨在促進行文流暢，並使文字更可觀可讀。

## 1.1 著者

我是 Daniel Stenberg （音譯：施格爾），在 Mozilla 工作。過去逾二十年，我在許多專案中參與開放源始碼及網路工作。
也許我最為人熟悉的，就是擔任 curl 及 libcurl 的主要開放者。
多年來，我一直參與 IETF HTTPbis 工作小組的工作，其間一直與時並進，了解 HTTP 1.1 更新的最新發展，並參與制定 http2 標準化工作。

  電郵： daniel@haxx.se

  Twitter： [@bagder](https://twitter.com/bagder)

  網址： [daniel.haxx.se](http://daniel.haxx.se/)

  部落格： [daniel.haxx.se/blog](http://daniel.haxx.se/blog/)

## 1.2 協助

若文中發現疏漏謬誤，請將有關段落的更正本發送給我，以作修正版。襄助的各方，我將給予適當鳴謝！希望本文件能隨時間更臻完善。

本文件可在 [http://daniel.haxx.se/http2](http://daniel.haxx.se/http2) 找到

## 1.3 版權

<img style="float: right;" src="https://raw.githubusercontent.com/bagder/http2-explained/master/images/creative-commons.png" />

This document is licensed under the Creative Commons Attribution 4.0 license: http://creativecommons.org/licenses/by/4.0/

## 1.4 修改

本文件初版在 2014 年四月二十五日發布。下列最近版本所作過的最主要變動。

### 版本 1.13

- Converted the master version of this document to Markdown syntax
- 13: Mention more resources, updated links and descriptions 
- 12: Updated the QUIC description with reference to draft 
- 8.5: Refreshed with current numbers 
- 3.4: The average is now 40 TCP connections 
- 6.4: Updated to reflect what the spec says 

### 版本 1.12

- 1.1: HTTP/2 is now in an official RFC 
- 6.5.1: Link to the HPACK RFC 
- 9.1: Mention the Firefox 36+ config switch for http2 
- 12.1: Added section about QUIC 

### 版本 1.11

- Lots of language improvements mostly pointed out by friendly contributors 
- 8.3.1: Mention nginx and Apache httpd specific acitivities 

### 版本 1.10

- 1: The protocol has been “okayed” 
- 4.1: Refreshed the wording since 2014 is last year 
- Front: Added image and call it “http2 explained” there, fixed link 
- 1.4: Added document history section 
- Many spelling and grammar mistakes corrected 
- 14: Added thanks to bug reporters 
- 2.4: Better labels for the HTTP growth graph 
- 6.3: Corrected the wagon order in the multiplexed train 
- 6.5.1: HPACK draft-12 

### 版本 1.9

- Updated to HTTP/2 draft-17 and HPACK draft-11  
- Added section "10. http2 in Chromium" (== one page longer now)  
- Lots of spell fixes  
- At 30 implementations now  
- 8.5: Added some current usage numbers  
- 8.3: Mention internet explorer too  
- 8.3.1 Added "missing implementations"  
- 8.4.3: Mention that TLS also increases success rate
