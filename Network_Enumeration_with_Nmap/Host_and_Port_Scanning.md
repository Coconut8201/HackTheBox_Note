# Host and Port Scanning  主機和埠掃描
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
