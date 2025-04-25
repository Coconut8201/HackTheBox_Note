# Service Enumeration  服務枚舉

## 🔍 Nmap 掃描快速筆記
<details>
  <summary>展開內容</summary>

  <h3>🎯 掃描目的</h3>
   - 準確辨識應用程式與版本：有助於找出已知漏洞與精準匹配 Exploit(開發)
   - 版本號越精確，漏洞利用越精準

  <h3>⚡ 掃描流程建議</h3>
  1. 快速掃描（低流量）

   - 指令範例：nmap &lt;IP&gt;
   - 減少被防火牆偵測風險

  2. 完整埠掃描（背景執行）

   - 指令範例：nmap -p- -sV &lt;IP&gt;
   - 掃描所有 65535 個 TCP 埠，抓出服務版本

  3. 查看掃描進度
   - 在掃描中按 Space 或 Enter 可即時顯示進度
   -或使用 --stats-every=5s 每 5 秒輸出一次進度

  4. 提高輸出詳盡度
   - 使用 -v 或 -vv，可即時看到開放埠資訊

  <h3>📦 範例指令</h3>

  ```
  sudo nmap 10.129.2.28 -p- -sV
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s
sudo nmap 10.129.2.28 -p- -sV -v
  ```

  <h3>📜 Banner Grabbing（橫幅抓取）</h3>

  - Nmap 主要透過 banner 判斷服務與版本。
  - 若 banner 無法辨識，則改用 特徵比對方式（速度較慢）
  - 自動化結果有可能 **遺漏關鍵資訊**

  <h3>🔎 深入分析（手動補強 Nmap 資訊）</h3>
  
  - 使用 `nc` 手動連線port服務，如 SMTP。
  - 搭配 `tcpdump` 攔截封包，觀察伺服器回應橫幅
  - 可以看到 Nmap 沒有顯示的詳細資訊，例如作業系統（Ubuntu）
  
  <h3>🧱 TCP 通訊流程解析</h3>

  1. `SYN`:我們發起連線
  2. `SYN-ACK`:伺服器回應
  3. `ACK`:完成三次握手
  4. `PSH-ACK`:伺服器推送資料(banner)並確認連線
  5. `ACK`:我們的回應，表示已接收

  <h3>🛠 工具補充</h3>

   - Netcat(nc): 手動抓取banner
   - Tcpdump: 封包攔截分析

  ```bash
  nc -nv <IP> 25         # 連線 SMTP
tcpdump -i eth0 host <本機IP> and <目標IP>
  ```

</details>

<hr/>
<hr/>

## 詳細筆記
對我們來說，準確地確定應用程式及其版本是非常重要的。我們可以利用這些資訊來掃描已知的漏洞，並在找到對應版本的情況下分析其原始碼。準確的版本號能讓我們搜尋出更精確、與該服務及目標作業系統相符的漏洞利用程式。

通常，我們會先執行快速的port scan，以便我們能對可用的port 有一個初步的概念。這樣產生較少的流量，否則可以會被安全機制發現並封鎖。我們可以先處理這些初步結果，並在背景中執行完整的port scan (-p-)，以找出全部開放的port。接者可以透過版本掃描(-sV)對特定port 進行服務及版本的偵測。

完整的埠掃描需要花費相當長的時間。若要查看掃描進度，我們可以在掃描過程中按下 空白鍵（Space Bar），這樣 Nmap 就會顯示目前的掃描狀態給我們看(或是按下enter 鍵)。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:44 CEST
[Space Bar]
Stats: 0:00:03 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

**另一個選項 （--stats-every=5s） 是定義狀態應該如何顯示。在這裡，我們可以指定秒數 （s） 或分鐘 （m），之後我們想要獲取狀態。**

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV --stats-every=5s

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 19:46 CEST
Stats: 0:00:05 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 13.91% done; ETC: 19:49 (0:00:31 remaining)
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 39.57% done; ETC: 19:48 (0:00:15 remaining)
```

**我們也可以提高詳細輸出等級（使用 -v 或 -vv），這樣當 Nmap 偵測到開放的埠時，就會立即顯示出來。**
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -v 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:03 CEST
NSE: Loaded 45 scripts for scanning.
Initiating ARP Ping Scan at 20:03
Scanning 10.129.2.28 [1 port]
Completed ARP Ping Scan at 20:03, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:03
Completed Parallel DNS resolution of 1 host. at 20:03, 0.02s elapsed
Initiating SYN Stealth Scan at 20:03
Scanning 10.129.2.28 [65535 ports]
Discovered open port 995/tcp on 10.129.2.28
Discovered open port 80/tcp on 10.129.2.28
Discovered open port 993/tcp on 10.129.2.28
Discovered open port 143/tcp on 10.129.2.28
Discovered open port 25/tcp on 10.129.2.28
Discovered open port 110/tcp on 10.129.2.28
Discovered open port 22/tcp on 10.129.2.28
<SNIP>
```

## Banner Grabbing  橫幅抓取
掃描完成後，我們將看到所有 TCP 連接埠以及系統上處於活動狀態的相應服務及其版本。
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:00 CEST
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.73 seconds
```

Nmap 主要是透過掃描到的port所回傳的橫幅（banner）來判斷服務資訊並將其顯示出來。如果無法透過橫幅辨識版本，Nmap 會嘗試使用基於特徵比對的方式來識別，但這會大幅增加掃描所需的時間。

Nmap 自動掃描結果的一個缺點是，有時候它會遺漏某些資訊，因為它可能無法正確解析某些回應內容。接下來，我們來看看一個範例。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
<SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..
Service scan match (Probe NULL matched with NULL line 3104): 10.129.2.28:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
```

接著我們會發現，目標主機上的 SMTP 伺服器實際上提供了比 Nmap 顯示的更多資訊。因為在這裡，我們看到它是基於 Ubuntu 的 Linux 發行版。這種情況發生的原因是，在完成三向握手（three-way handshake）之後，伺服器通常會傳送一段識別用的橫幅（banner），用來告知客戶端目前所連接的是哪種服務。

在網路層級上，這通常是透過 TCP 標頭中的 PSH 標誌來傳送的。不過，也有可能某些服務並不會立即提供這樣的資訊，或者這些橫幅已經被移除或修改過。

<u>**如果我們使用 nc（netcat）手動連接到 SMTP 伺服器、抓取它回傳的 banner，同時用 tcpdump 來攔截封包，我們就能看到一些 Nmap 沒有顯示出來的資訊。**</u>

### Tcpdump
```bash
Coconut820@htb[/htb]$ sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

### netCat
```bash
Coconut820@htb[/htb]$  nc -nv 10.129.2.28 25

Connection to 10.129.2.28 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```

### Tcpdump - Intercepted Traffic tcpdump - 攔截的流量
```bash
18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], seq 1798872233, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 331260178 ecr 0,sackOK,eol], length 0
18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], seq 1130574379, ack 1798872234, win 65160, options [mss 1460,sackOK,TS val 1800383922 ecr 331260178,nop,wscale 7], length 0
18:28:07.255281 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], ack 1, win 2058, options [nop,nop,TS val 331260304 ecr 1800383922], length 0
18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], seq 1:36, ack 1, win 510, options [nop,nop,TS val 1800383985 ecr 331260304], length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)
18:28:07.319426 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], ack 36, win 2058, options [nop,nop,TS val 331260368 ecr 1800383985], length 0
```

前三行向我們展示了三次握手。
|||
|---|---|
|1. [SYN]|18:28:07.128564 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S], <SNIP>|
|2. [SYN-ACK]|18:28:07.255151 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.], <SNIP>|
|3. [ACK]|18:28:07.255281 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>|

之後，目標的 SMTP 伺服器會傳送一個同時帶有 PSH 和 ACK 標誌的 TCP 封包給我們，其中：

- PSH（Push） 表示伺服器正在主動將資料傳送給我們，並請求立即處理這些資料。
- ACK（Acknowledgment） 則表示這個封包同時也是對先前我們傳送資料的確認，告知我們所需的資料已經完整收到並且現在要回傳結果了。

簡而言之，這個封包代表伺服器已經準備好傳送資料（如 banner），並確認雙方的通訊狀態正常。
|||
|---|---|
|4. [PSH-ACK]|18:28:07.319306 IP 10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.], <SNIP>|


我們所發送的最後一個 TCP 封包帶有 ACK 標誌，這表示我們已經確認接收到伺服器傳送的資料。這是三向通訊中的標準流程，確保雙方都知道資料已成功傳遞。
|||
|---|---|
|5. [ACK]|18:28:07.319426 IP 10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.], <SNIP>|