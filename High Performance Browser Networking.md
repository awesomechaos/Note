# High Performance Browser Networking

## 延迟与带宽

* **提高传输速度，要么减小roundtrip time，要么增加congestion window**
* 延迟组成: Propagation delay(传播时延), Transmission delay(发送时延), Processing delay, Quering delay
* 延迟测试 traceroute on unix, tracert on win
* slow-start与slow-start Restart是造成传输时间长的原因
* Head of line blocking:TCP丢包时，要等待该包重新接收完整，后面的包被阻塞了
* **TCP协议不变的地方：**
	1. TCP的三次握手引入了一个固定的来回延迟时间
	2. TCP slow-start
	3. TCP的阻塞控制管着着吞吐量
	4. TCP吞吐量由current congestion window size决定

### 提速方式
* Upgrade server kernel to latest version (Linux: 3.2+).
* Ensure that cwnd size is set to 10.
* Disable slow-start after idle.
	> $> sysctl net.ipv4.tcp_slow_start_after_idle
	>
	> $> sysctl -w net.ipv4.tcp_slow_start_after_idle=0
* Ensure that window scaling is enabled.
* Eliminate redundant data transfers.
* Compress transferred data.
* Use CDN,Position servers closer to the user to reduce roundtrip times.
* Reuse established TCP connections whenever possible.
* [TCP Fast Open](http://chimera.labs.oreilly.com/books/1230000000545/ch02.html#FAST_OPEN)

### Reserved Private IP Ranges
 IP address range | Number of addresses
 :--------------- | :---------------
 10.0.0.0 - 10.255.255.255 | 16,777,216
 172.16.0.0 - 172.31.255.255 | 1,048,576
 192.168.0.0 - 192.168.255.255 | 65,536

### UDP优化建议 [RFC 5405](https://tools.ietf.org/html/rfc5405)
* Application must tolerate a wide range of Internet path conditions.
* Application should control rate of transmission.
* Application should perform congestion control over all traffic.
* Application should use bandwidth similar to TCP.
* Application should back off retransmission counters following loss.
* Application should not send datagrams that exceed path MTU.
* Application should handle datagram loss, duplication, and reordering.
* Application should be robust to delivery delays up to 2 minutes.
* Application should enable IPv4 UDP checksum, and must enable IPv6 checksum.
* Application may use keepalives when needed (minimum interval 15 seconds).

## Transport Layer Security
* TLS握手方式：
![TLS握手](https://github.com/awesomechaos/Note/raw/master/image/hpbn_tls_4.2.png)
* **Optimize TLS handshaks with [Session Resumption](http://chimera.labs.oreilly.com/books/1230000000545/ch04.html#TLS_RESUME) and [False Start](http://chimera.labs.oreilly.com/books/1230000000545/ch04.html#TLS_FALSE_START)**
* **Google’s servers reduce their OpenSSL buffers down to about 5 KB**
* **用gzip替代TLS压缩，服务器关闭TLS压缩，因为有攻击漏洞和对CPU消耗过大**

### [SSL安全测试网站](https://www.ssllabs.com/ssltest/)
