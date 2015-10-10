# 8. http2 新世界

當 http2 廣泛採用時，會是一番怎樣的新景象？又或者，會否廣獲採用？

## 8.1. http2 與我何干？

http2 尚未廣泛部署或使用。無人能預知他朝事。君可見 SPDY 怎樣獲採用，加上過去與現在的實驗，從而可猜猜算算。

http2 減少了所需的網路來回行程（network round-trips）數目，並以多工（multiplexing）及快速捨棄不要的串流（stream）迴避了 head of line blocking（龍頭阻滯）的吊詭。

它允許大量平行串流（parallel streams），比起當今大部份域名分割（sharding）網站，有過之而無不及。

當在串流上好好使用優先順序（priorities）時，將能大大提升機會，客戶端能實際優先得到重要資料，才到較次要的資料。總而言之，我認為這很大機會能提升網頁載入速度，讓網站回應更靈敏。一句作結：更佳的網站體驗。

到底快了多少、能有多少提升，且拭目以待，我想暫未能定論。首先，這個科技仍屬非常早期，甚至仍未見客戶端和伺服器度身作出實作，以把握所有這個新的通訊協定所帶來的性能。

## 8.2. http2 怎樣影響網路開發？

Over the years web developers and web development environments have gathered a full toolbox of tricks and tools to work around problems with HTTP 1.1, recall that I outlined  some of them in the beginning of this document as a justification for http2.

Lots of those workarounds that tools and developers now use by default and without thinking, will probably hurt http2 performance or at least not really take advantage of http2's new super powers. Spriting and inlining should most likely not be done with http2. Sharding will probably be detrimental to http2 as it will probably benefit from using less connections.

A problem here is of course that web sites and web developers need to develop and deploy for a world that in the short term at least, will have both HTTP1.1 and http2 clients as users and to get maximum performance for all users can be challenging without having to offer two different front-ends.

For these reasons alone, I suspect there will be some time before we will see the full potential of http2 being reached.

## 8.3. http2 實作

Trying to document specific implementations in a document such as this is of course completely futile and doomed to fail and only feel outdated within a really short period of time. Instead I'll explain the situation in broader terms and refer readers to the [list of implementations](https://github.com/http2/http2-spec/wiki/Implementations) on the http2 web site.

There was a large amount of implementations already early on, and the amount has increased over time during the http2 work. At the time of  writing this there are over 40 implementations listed, and most of them implement the final version.

### 8.3.1 瀏覽器

Firefox has been the browser that's been on top of the bleeding edge drafts,
Twitter has kept up and offered its services over http2. Google started during
April 2014 to offer http2 support on a few test servers running their services
and since May 2014 they offer http2 support in their development versions of
Chrome. Microsoft has shown a tech preview with http2 support for their next
Internet Explorer version. Safari (with iOS 9 and Mac OS X El Capitan) and
Opera have both said they will support http2.

### 8.3.2 伺服器

現在已有許多 http2 伺服器的實作。

熱門的 Nginx 伺服器提供 http2 支援，初見於
[1.9.5](https://www.nginx.com/blog/nginx-1-9-5/) 版本，已在 2015 年九月二十二日釋出（取代 SPDY 模組，因此無法在同一伺服器報行個體並行操作）。

Apache 的 httpd 伺服器備有一個 http2 模組，仍在開發階段，名為
[mod_http2](https://httpd.apache.org/docs/2.4/mod/mod_http2.html)，預期會隨著未來的 2.4.17 發行釋出。

[H2O](https://h2o.examp1e.net/)、[Apache Traffic
Server](http://trafficserver.apache.org/)、[nghttp2](https://nghttp2.org/)、[Caddy](http://caddyserver.com/) 及
[LiteSpeed](https://www.litespeedtech.com/products/litespeed-web-server/overview)
各有發行具備 http2 的伺服器。

### 8.3.3 其他

curl 及 libcurl 支援不安全（insecure）http2 ，以及使用多個不同 TLS 程式庫（libraries）之一建立的 TLS 為基礎的 http2。

Wireshark 支援 http2。絕佳的工具來分析 http2 網路流量（network traffic）。

## 8.4. http2 紛紜

During the development of this protocol the debate has been going back and forth and of course there is a certain amount of people who believe this protocol ended up completely wrong. I wanted to mention a few of the more common complaints and mention the arguments against them:

### 8.4.1. “The protocol is designed or made by Google”

It also has variations implying that the world gets even further dependent or controlled by Google by this. This isn't true. The protocol was developed within the IETF in the same manner that protocols have been developed for over 30 years. However, we all recognize and acknowledge Google's impressive work with SPDY that not only proved that it is possible to deploy a new protocol this way but also provided numbers illustrating what gains could be made.

Google has publicly [announced](http://blog.chromium.org/2015/02/hello-http2-goodbye-spdy-http-is_9.html) that they will remove support for SPDY and NPN in Chrome in 2016 and they urge servers to migrate to HTTP/2 instead.

### 8.4.2. “The protocol is only useful for browsers”

This is sort of true. One of the primary drivers behind the http2 development is the fixing of HTTP pipelining. If your use case originally didn't have any need for pipelining then chances are http2 won't do a lot of good for you. It certainly isn't the only improvement in the protocol but a big one.

As soon as services start realizing the full power and abilities the multiplexed streams over a single connection brings, I suspect we will see more application use of http2.

Small REST APIs and simpler programmatic uses of HTTP 1.x may not find the step to http2 to offer very big benefits. But also, there should be very few downsides with http2 for most users.

### 8.4.3. “The protocol is only useful for big sites”

Not at all. The multiplexing capabilities will greatly help to improve the experience for high latency connections that smaller sites without wide geographical distributions often offer. Large sites are already very often faster and more distributed with shorter round-trip times to users.

### 8.4.4. “Its use of TLS makes it slower”

This can be true to some extent. The TLS handshake does add a little extra,
but there are existing and ongoing efforts on reducing the necessary
round-trips even more for TLS. The overhead for doing TLS over the wire
instead of plain-text is not insignificant and clearly notable so more CPU and
power will be spent on the same traffic pattern as a non-secure protocol. How
much and what impact it will have is a subject of opinions and
measurements. See for example [istlsfastyet.com](https://istlsfastyet.com/)
for one source of info.

Telecom and other network operators, for example in the ATIS Open Web
Alliance, say that they [need unencrypted
traffic](http://www.atis.org/openweballiance/docs/OWAKickoffSlides051414.pdf)
to offer caching, compression and other techniques necessary to provide a fast
web experience over satellite, in airplanes and similar.  http2 does not make
TLS use mandatory so we shouldn't conflate the terms.

Many Internet users have expressed a preference for TLS to be used more widely and we should help to protect users' privacy.

Experiments have also shown that by using TLS, there is a higher degree of success than when implementing new plain-text protocols over port 80 as there are just too many middle boxes out in the world that interfere with what they would think is HTTP 1.1 if it goes over port 80 and might look like HTTP at times.

Finally, thanks to http2's multiplexed streams over a single connection, normal browser use cases still could end up doing substantially fewer TLS handshakes and thus perform faster than HTTPS would when still using HTTP 1.1.

### 8.4.5. “Not being ASCII is a deal-breaker”

Yes, we like being able to see protocols in the clear since it makes debugging and tracing easier. But text based protocols are also more error prone and open up for much more parsing and parsing problems.

If you really can't take a binary protocol, then you couldn't handle TLS and compression in HTTP 1.x either and its been there and used for a very long time.

### 8.4.6. “It isn't any faster than HTTP/1.1”

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

