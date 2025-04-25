# Host and Port Scanning  主機和埠掃描
## 快速整理筆記
<details>
  <summary>展開內容</summary>
  <h3>🧠 掃描目標的目的:</h3>

  在確認主機存活後，可進一步獲取以下資訊：
  - 開放的連接埠（TCP/UDP）
  - 服務名稱與版本（Service/Version）
  - 作業系統(OS)
  - 服務所提供的資訊（如 Samba Workgroup）

  <h3>🔍 埠狀態說明:</h3>

  |  狀態 | 說明 |
  |---|---|
  |open 打開 | 連接已建立，服務可回應|
  |closed 閉 | 目標回傳 RST，表示埠關閉|
  |filtered 過濾 | 未收到回應或 ICMP 錯誤，狀態不明|
  |unfiltered 未過濾 | 僅出現在 ACK 掃描中，表示可達但狀態不明|
  |open 打開 | filtered過濾|
  |closed 閉 | filtered過濾|

  <h3>🚀 常見掃描方法與範例</h3>
  🔧 基礎用法: <br/>
  <code>sudo nmap <目標IP> [選項(option)]</code>

  🔥nmap option:
  | 選項 | 用途 |
  |---|---|
  | -sS | SYN 掃描（半開掃描）[預設，需 root]，掃描前 1000 個 TCP 連接埠|
  | -sT | TCP Connect 掃描(完整 TCP 握手) |
  | -sU | UDP 掃描 |
  | -sV | 探測服務版本 |
  | -O | 辨識作業系統 |
  | -Pn | 跳過主機存活檢查（不使用 ping）|
  | -p | 指定連接埠（如 -p 22,80 或 -p 1-65535 |
  | --top-ports=N | 掃描前 N 個熱門連接埠 |
  | -F | 快速掃描（前 100 個連接埠） |
  | --packet-trace | 顯示發送與接收封包 |
  | --reason | 顯示為何被判定為該狀態 |
  | -n | 不進行 DNS 解析 |
  | --disable-arp-ping | 停用 ARP 掃描（適用於本地網路 |

  💡 TCP 掃描案例
  掃描前 10 熱門 TCP 連接埠:
  ```bash
  sudo nmap 10.129.2.28 --top-ports=10
  ```

  SYN 掃描指定埠位:
  ```bash
  sudo nmap -sS -p 80,443 -Pn -n --disable-arp-ping 10.129.2.28
  ```

  跟蹤封包 + 原因顯示:
  ```bash
  sudo nmap -sT -p 443 -Pn -n --disable-arp-ping --packet-trace --reason 10.129.2.28
  ```

  <h3>🌐 UDP 掃描案例</h3>
  掃描指定 UDP 埠:
  
  ```bash
  sudo nmap -sU -p 137 -Pn -n --disable-arp-ping --packet-trace --reason 10.129.2.28
  ```

  常見狀況說明:
  - UDP 無回應 => |filtered
  - ICMP type3/code3 => 代表該UDP port closed
  - 服務僅在應用層主動回應時才有資料返回

  <h3>🔍 版本掃描範例</h3>
  檢測開啟埠的服務與版本
  ```bash
  sudo nmap -sV -p 445 -Pn -n --disable-arp-ping --packet-trace --reason 10.129.2.28
  ```

  <h3>📌 結語</h3>

  | 狀況 | 處理建議 |
  |---|---|
  | filtered 狀態 | 多數為防火牆丟棄封包，需多次測試或換掃描方式 |
  | 無回應的 UDP | 改用 -sU --reason 並觀察 ICMP 回應類型 |
  | 需求高準確率時 | 使用 -sT Connect 掃描 |
  | 需隱密偵測 | 使用 -sS SYN 掃描 |

</details>

<hr>
<hr>

## 詳細筆記
在確認目標主機是存活（alive）之後，我們會希望獲得該系統更準確、更深入的資訊。

- 開放的連接埠(port)與其服務
- 服務版本 services version
- Information that the services provided 服務提供的資訊
- Operating system  操作系統

掃描port 共有6種狀態:
|state|描述|
|---|---|
|open|表示已建立與掃描埠的連接。這些連接可以是 TCP 連接 、UDP 數據報以及 SCTP 關聯 |
|closed|當埠顯示為 closed 時，TCP 協定指示我們收到的數據包包含 RST 標誌。這種掃描方法還可用於確定我們的目標是否還活著|
|filtered|Nmap 無法正確識別掃描的埠是打開還是關閉，因為目標沒有返回對該埠的回應，或者我們從目標收到錯誤代碼|
|unfiltered|埠的此狀態僅在 TCP-ACK 掃描期間發生，這意味著該埠可訪問，但無法確定它是打開的還是關閉的|
|open\|filtered|如果沒有得到一個特定埠的回應，Nmap 會把它設置為那個狀態。這表明防火牆或數據包篩檢程式可以保護埠|
|closed\|filtered|此狀態僅出現在 IP ID 空閒掃描中，並指示無法確定掃描的埠是關閉還是被防火牆過濾|

## Discovering Open TCP Ports 發現打開的 TCP 連接埠
預設情況下，Nmap 使用 SYN 掃描（-sS）來掃描前 1000 個 TCP 連接埠。但這個 SYN 掃描僅在我們以 root 權限執行 Nmap 時才會自動啟用，因為它需要建立原始 TCP 封包（raw packet）的 socket 權限。若不是以 root 執行，Nmap 就會改用 TCP connect 掃描（-sT）作為預設方式。

|方式|範例|說明|
|---|---|---|
|逐個列出| -p 22, 25, 80, 139, 445|掃描這幾個特定的連接埠|
|連續範圍| -p 22-445|掃描從22到445 的所有連接埠|
|熱門連接埠|--top-ports=10|掃描nmap資料庫中的常見的10個連接埠|
|掃描所有連接埠| -p- |掃描所有的65535個TCP 連接埠|
|快速掃描| -F |掃描前100 個最熱門的連接埠，速度比較快|

### Scanning Top 10 TCP Ports
```bash
[!bash!]$ sudo nmap 10.129.2.28 --top-ports=10 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:36 CEST
Nmap scan report for 10.129.2.28
Host is up (0.021s latency).

PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
23/tcp   closed   telnet
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 1.44 seconds
```
|掃描選項|描述|
|---|---|
|10.129.2.28|掃描特定的目標|
|--top-ports=10|掃描top 10 熱門的連接埠| 

我們可以看到，在這次的掃描中，我們僅針對目標的前 10 個 TCP 連接埠進行了掃描，Nmap 根據每個連接埠的狀態進行了回報。

如果我們追蹤 Nmap 發送的封包，會發現當 Nmap 對目標的 TCP 21 埠發送 SYN 封包時，目標主機回傳了一個帶有 RST（Reset）旗標的封包，這代表：
**目標主機上的該連接埠是「關閉（closed）」的。**

為了能清楚觀察 SYN 掃描的行為，我們會關閉以下功能：
- **-Pn**：停用 ICMP echo request，讓 Nmap 不透過 ping 判斷主機是否上線。

- **-n**：停用 DNS 解析，加快掃描速度，並避免名稱干擾。

- **--disable-arp-ping**：停用 ARP ping 掃描，防止本地網段的 ARP 回應取代 ICMP 回應。

# Nmap - Trace the Packets Nmap - 跟蹤數據包
```bash
[!bash!]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

|掃描選項|描述|
|---|---|
|10.129.2.28|掃描特定的目標|
| -p 21| 僅掃描指定值|
| --packet-trace|顯示發送和接收的所有數據包| 
|-n|禁用dns 解析|
|--disable-arp-ping|禁用ARP ping|

我們可以從 SENT 那一行看出，我們（10.10.14.2）向目標（10.129.2.28）發送了一個帶有 SYN 旗標（S）的 TCP 封包。接著在 RCVD 那一行中，我們可以看到目標回傳了一個包含 RST 和 ACK 旗標（RA）的 TCP 封包。RST 和 ACK 旗標分別用來確認收到 TCP 封包（ACK）以及結束 TCP 連線（RST）。

### Request 請求
|message|描述|
|---|---|
|SENT (0.0429s)|表示 Nmap 的 SENT 操作，即向目標發送封包的動作。|
|TCP|顯示用來與目標連接埠互動的協定。|
|10.10.14.2:63090 >|代表我們的 IPv4 位址和源埠，Nmap 將使用它來發送數據包|
|10.129.2.28：21|顯示目標 IPv4 位址和目標埠|
|S|發送的 TCP 資料包的 SYN 標誌|
|ttl=56 id=57322 iplen=44 seq=1699105818 win=1024 mss 1460|其他 TCP 標頭參數。|

### Response 回應
|message|描述|
|---|---|
|RCVD (0.0573s)|指示從目標接收的數據包|
|TCP|正在使用的協定|
|10.129.2.28:21 >|表示目標 IPv4 位址和源埠，將用於回復|
|10.10.14.2:63090|顯示我們的 IPv4 位址和將要回復的埠|
|RA|發送的 TCP 資料包的 RST 和 ACK 標誌|
|ttl=64 id=0 iplen=40 seq=0 win=0|其他 TCP 標頭參數|

### Connect Scan 連接掃描
Nmap 的 TCP 連線掃描（-sT）使用 TCP 三向交握來判斷目標主機上的特定連接埠是開啟還是關閉。掃描時會向目標連接埠發送一個 SYN 封包並等待回應。
如果目標回應 <u>**SYN-ACK 封包，則該連接埠被視為「開啟」；如果回應的是 RST 封包，則該連接埠被視為「關閉」</u>。**

**Connect 掃描**（也稱為完整 TCP 連接掃描）非常準確，因為他完成了三次的TCP 握手，所以我們可以確定埠的確切狀態(open, closed, filtered)。事實上，Connect 掃描是最不隱密的技術之一，因為它會建立完整連線，這在多數系統上都會產生記錄，也很容易被現代的入侵偵測/防禦系統（IDS/IPS）偵測到。

在某些情況下，當準確性是首要考量，且目的是在不對服務造成重大干擾的情況下進行網路映射時，Connect 掃描仍然相當有用。由於這種掃描會完整建立 TCP 連線，因此與服務互動更為乾淨，與其他更具侵入性的掃描方式相比，更不容易導致服務錯誤或不穩定。儘管它不是最隱密的掃描方法，但有時被認為是一種較為「有禮貌」的掃描，因為它的行為就像是一般用戶端的正常連線，因此對目標系統的影響較小。

當目標主機啟用個人防火牆，該防火牆會封鎖（丟棄）所有傳入封包但允許傳出封包時，Connect 掃描也很有用。在這種情況下，Connect 掃描可以繞過防火牆，並準確地判斷目標埠口的狀態。

不過，需要注意的是，Connect 掃描的速度比其他掃描方式慢，因為它需要在每次送出封包後等待目標的回應。如果目標系統正忙碌或反應遲緩，這個等待時間可能會延長。

像 SYN 掃描（也稱為半開掃描）這樣的掃描通常被認為更隱蔽，因為它們不會完成完全握手，在發送初始 SYN 數據包後會導致連接不完整。這最大限度地減少了觸發連接日誌的可能性，同時仍能收集埠狀態資訊。然而，先進的 IDS/IPS 系統已經適應了檢測這些更微妙的技術。

Connect Scan on TCP Port 443
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:26 CET
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```

## Filtered Ports  過濾埠
當一個連接埠顯示為「filtered（已過濾）」時，可能有多種原因。在大多數情況下，是因為防火牆設定了某些規則來處理特定的連線。這些封包可能會被「丟棄（dropped）」或「拒絕（rejected）」。如果封包被丟棄，Nmap 就不會從目標那裡收到任何回應，而根據預設，Nmap 的重試次數（--max-retries）設定為 10。這表示 Nmap 會重新傳送請求到目標的連接埠，以判斷先前的封包是否只是意外地未被處理。

讓我們來看一個例子：防火牆丟棄了我們在執行連接埠掃描時所送出的 TCP 封包。因此，我們掃描了 TCP 連接埠 139，而它早已顯示為「filtered（已過濾）」。為了追蹤我們發送的封包是如何被處理的，我們再次停用了 ICMP echo 請求（-Pn）、DNS 解析（-n），以及 ARP ping 掃描（--disable-arp-ping）。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

|掃描選項|描述|
|---|---|
|10.129.2.28|掃描特定目標|
|-p 139|緊掃描指定的port|
|--packet-trace|顯示發送和接收的所有數據包|
|-n|禁用DNS解析|
|--disable-arp-ping|禁用ARP ping|
|-Pn|禁用ICMP Echo 請求|

我們可以從最後一次掃描中看到，Nmap 發送了兩個帶有 SYN 標誌的 TCP 封包。從掃描所花費的時間（2.06 秒）來看，我們可以判斷出這次明顯比先前的掃描（約 0.05 秒）耗時得多。如果防火牆是「拒絕（reject）」封包的情況就不同了。為此，我們觀察 TCP 連接埠 445，這個連接埠就是被防火牆透過相應規則來處理的。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```
|掃描選項|描述|
|---|---|
|10.129.2.28|掃描特定目標|
|-p 445|緊掃描指定的port|
|--packet-trace|顯示發送和接收的所有數據包|
|-n|禁用DNS解析|
|--disable-arp-ping|禁用ARP ping|
|-Pn|禁用ICMP Echo 請求|

作為回應，我們收到了一個 ICMP 封包，類型為 3、錯誤代碼為 3，這表示目標連接埠無法到達（port unreachable）。儘管如此，如果我們已知該主機是存活的，那就可以合理推斷該連接埠上的防火牆正在「拒絕（reject）」這些封包，因此我們之後需要更詳細地檢查這個連接埠。

## UDP Port Scan  UDP 埠掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -F -sU

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds
```

|掃描選項|描述|
|---|---|
|10.129.2.28|掃描指定的目標|
|-F|掃描錢100 個port|
|-sU|執行UDP掃描|

這種情況的另一個缺點是，<u>**我們經常收不到任何回應**</u>，因為 Nmap 在掃描 UDP 連接埠時會發送空的資料包（datagrams），而我們可能無法收到任何回應。因此，我們無法確定該 UDP 封包是否有成功送達。若該 UDP 連接埠是開啟的，只有在應用程式有設定回應封包的情況下，我們才會收到回覆。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:15 CEST
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.0031s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```
|描述選項|描述|
|---|---|
|10.129.2.28|掃描指令的目標|
|-sU|執行UDP 掃描|
|-Pn|禁用ICMP Echo 請求|
|-n|禁用DNS解析|
|--disable-arp-ping|禁用ARP ping|
|--packet-trace|顯示和接收所有數據包|
|-p 137|緊掃描特定port|
|--reason|顯示port 於特定狀態的原因|

如果我們收到一個 ICMP 回應，其錯誤代碼為 3（port unreachable），那我們就可以確定該連接埠確實是關閉的。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:25 CEST
SENT (0.0445s) UDP 10.10.14.2:63825 > 10.129.2.28:100 ttl=57 id=29925 iplen=28
RCVD (0.1498s) ICMP [10.129.2.28 > 10.10.14.2 Port unreachable (type=3/code=3) ] IP [ttl=64 id=11903 iplen=56 ]
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.11s latency).

PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in  0.15 seconds
```

對於所有其他類型的 ICMP 回應，Nmap 會將被掃描的連接埠標記為「open|filtered（開啟或已過濾）」，表示無法確定該連接埠是開啟還是被防火牆過濾。


```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:32 CEST
SENT (0.0380s) UDP 10.10.14.2:52341 > 10.129.2.28:138 ttl=50 id=65159 iplen=28
SENT (1.0392s) UDP 10.10.14.2:52342 > 10.129.2.28:138 ttl=40 id=24444 iplen=28
Nmap scan report for 10.129.2.28
Host is up, received user-set.

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

另一個實用的連接埠掃描方法是使用 -sV 選項，它可以從開啟的連接埠中獲取更多可用的資訊。這種方法能夠辨識出版本號、服務名稱，以及關於目標的更多細節資訊。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-04 11:10 GMT
SENT (0.3426s) TCP 10.10.14.2:44641 > 10.129.2.28:445 S ttl=55 id=43401 iplen=44  seq=3589068008 win=1024 <mss 1460>
RCVD (0.3556s) TCP 10.129.2.28:445 > 10.10.14.2:44641 SA ttl=63 id=0 iplen=44  seq=2881527699 win=29200 <mss 1337>
NSOCK INFO [0.4980s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.4980s] nsock_connect_tcp(): TCP connection requested to 10.129.2.28:445 (IOD #1) EID 8
NSOCK INFO [0.5130s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.2.28:445]
Service scan sending probe NULL to 10.129.2.28:445 (tcp)
NSOCK INFO [0.5130s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.2.28:445]
Service scan sending probe SMBProgNeg to 10.129.2.28:445 (tcp)
NSOCK INFO [6.5190s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [10.129.2.28:445]
NSOCK INFO [6.5190s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.2.28:445]
NSOCK INFO [6.5320s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.2.28:445] (135 bytes)
Service scan match (Probe SMBProgNeg matched with SMBProgNeg line 13836): 10.129.2.28:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.5320s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds
```

|描述選項|描述|
|---|---|
|-sV|執行服務掃描|