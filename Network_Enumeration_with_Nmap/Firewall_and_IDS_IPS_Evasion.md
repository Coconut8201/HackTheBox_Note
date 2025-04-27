# Firewall and IDS/IPS Evasion 防火牆和 IDS/IPS 規避
Nmap 提供了許多不同的方法來繞過防火牆規則以及入侵偵測/防禦系統（IDS/IPS）。
這些方法包括將封包進行碎片化、使用誘餌（decoys）等。

## Firewalls  防火牆
防火牆是一種防止來自外部網路未經授權連線嘗試的安全措施。
每個防火牆安全系統都基於一個軟體組件，該組件負責監控防火牆與進入資料連線之間的網路流量，並根據設定好的規則來決定如何處理這些連線。
它會檢查每個網路封包，判斷是允許通過、忽略，還是阻擋。
這套機制旨在防止可能帶來危險的連線進入系統。

## IDS/IPS
與防火牆類似，入侵偵測系統（IDS）與入侵防禦系統（IPS）也是基於軟體的組件。
IDS 會掃描網路中可能的攻擊行為，進行分析，並回報任何偵測到的攻擊。
IPS 則補充了 IDS 的功能，在偵測到潛在攻擊時，會主動採取防禦措施。
這些攻擊的分析通常是基於模式比對（pattern matching）與特徵碼（signatures）。
如果偵測到特定的攻擊模式，例如服務偵測掃描（service detection scan），IPS 可能會阻止即將發生的連線嘗試。

## Determine Firewalls and Their Rules 確定防火牆及其規則
我們已經知道，當一個連接埠顯示為「filtered（已過濾）」時，可能有多種原因。
在大多數情況下，是因為防火牆設定了特定的規則來處理某些連線。
這些封包可能會被丟棄（dropped）或拒絕（rejected）。
被丟棄的封包會被直接忽略，主機不會返回任何回應。

被拒絕（rejected）的封包情況則不同，這些封包會帶有 RST（重設）標誌返回。
這些封包可能包含不同類型的 ICMP 錯誤代碼，或者完全不包含任何內容。

此類錯誤可以是：
- Net Unreachable — 網路無法到達
- Net Prohibited — 禁止存取網路
- Host Unreachable — 主機無法訪問
- Host Prohibited — 主機存取被禁止
- Port Unreachable — 埠無法訪問
- Proto Unreachable — 協議（Protocol）無法訪問


Nmap 的 TCP ACK 掃描（-sA）方式，相較於一般的 SYN 掃描（-sS）或 Connect 掃描（-sT），對防火牆與 IDS/IPS 系統來說更難以過濾。
原因是它只發送一個帶有 ACK 標誌 的 TCP 封包。
當目標連接埠無論是開啟還是關閉時，主機都必須回應一個帶有 RST 標誌 的封包。

與一般傳出的連線不同，來自外部網路的所有主動連線嘗試（帶有 SYN 標誌的封包）通常會被防火牆封鎖。
但帶有 ACK 標誌 的封包，防火牆通常會允許通過，因為它無法判斷這個連線是否是由內部網路主動建立的，還是來自外部網路的回應。

### SYN-Scan  SYN-Scan 掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:56 CEST
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:22 S ttl=53 id=22412 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:25 S ttl=50 id=62291 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:21 S ttl=58 id=38696 iplen=44  seq=4092255222 win=1024 <mss 1460>
RCVD (0.0329s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=40884 iplen=72 ]
RCVD (0.0341s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
RCVD (1.0386s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
SENT (1.1366s) TCP 10.10.14.2:57348 > 10.129.2.28:25 S ttl=44 id=6796 iplen=44  seq=4092320759 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.0053s latency).

PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

## ACK-Scan  ACK 掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:57 CEST
SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A ttl=49 id=12381 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A ttl=41 id=5146 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A ttl=49 id=5800 iplen=40  seq=0 win=1024
RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=55628 iplen=68 ]
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R ttl=64 id=0 iplen=40  seq=1660784500 win=0
SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A ttl=59 id=21915 iplen=40  seq=0 win=1024
Nmap scan report for 10.129.2.28
Host is up (0.083s latency).

PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```
## Nmap 掃描選項說明

| 掃描選項               | 說明                                      |
|-------------------------|-------------------------------------------|
| `10.129.2.28`           | 掃描指定的目標。                         |
| `-p 21,22,25`           | 只掃描指定的埠口（ports）。              |
| `-sS`                   | 對指定埠口進行 SYN 掃描。                |
| `-sA`                   | 對指定埠口進行 ACK 掃描。                |
| `-Pn`                   | 停用 ICMP Echo 請求（跳過 ping 檢測）。  |
| `-n`                    | 停用 DNS 解析。                          |
| `--disable-arp-ping`    | 停用 ARP ping。                           |
| `--packet-trace`        | 顯示所有發送與接收的封包。                |

注意我們從目標收到RCVD 封包以及其設定的flag：
- 使用 SYN (-sS) 掃描時，目標會嘗試建立TCP 連線，並回傳一個設定了SYN-ACK flag 的封包。
- 使用 ACK (-sA) 時，由於TCP port 22 is open，因此我們會收到帶有RST flag 的回應。
TCP port 25 沒有收到任何回應封包，這顯示封包被丟棄 dropped。

## Detect IDS/IPS  檢測 IDS/IPS
不同於防火牆及其規則，偵測 IDS/IPS 系統要困難得多，因為這些系統是被動式的流量監控系統。
IDS 系統會檢查主機之間的所有連線。
如果 IDS 發現封包中包含了預定義的內容或特徵，系統會通知管理員，並在最糟的情況下由管理員採取相應的行動。

IPS 系統會根據管理員事先設定的規則，自主採取行動，自動阻止潛在的攻擊。
需要特別注意的是，**IDS 和 IPS 是不同的應用，且 IPS 是作為 IDS 的補充來運作的。**

---
在滲透測試過程中，建議使用多個不同IP 的虛擬私人服務器(VPS), 以判斷目標網路上是否存在此類防護系統。如果管理員真測試目標網路上有潛在的攻擊行為，通常第一步是封鎖該攻擊來源的IP 位置。
如此，我們將無法使用該IP 存取網路，而且可能會被通知網路服務供應商(ISP)，進而導致被全面封鎖網際網路的存取權。

