#  Nmap 簡介
Nmap 是用許多不同的程式編寫的開源網路分析工具核安全審計工具，他可以識別主機的作業系統和版本。除此之外，nmap 提供掃描功能，可以根據需要設定封包過濾器、防火牆或是入侵偵測系統(IDS)

## Use Case
- 審計網路安全
- 模擬滲透測試
- 檢查防火牆和IDS設定和配置
- 可能連線的類型
- 網路映射(network mapping)
- 回應分析
- 識別開放端口
- 弱點評估

## Nmap Architecture  Nmap 架構
nmap 可以分為以下幾種技術掃描:
- host discovery 主機發現
- port scanning 連接阜掃描
- Service enumeration and detection 服務枚舉和檢測
- OS detection  作業系統偵測
- Scriptable interaction with the target service (Nmap Scripting Engine) 與目標服務的可腳本互動（Nmap 腳本引擎）

## Scan Techniques  掃描技術
Nmap 提供的所有掃描技術

```bash
Coconut820@htb[/htb]$ nmap --help

<SNIP>
SCAN TECHNIQUES:
  -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
  -sU: UDP Scan
  -sN/sF/sX: TCP Null, FIN, and Xmas scans
  --scanflags <flags>: Customize TCP scan flags
  -sI <zombie host[:probeport]>: Idle scan
  -sY/sZ: SCTP INIT/COOKIE-ECHO scans
  -sO: IP protocol scan
  -b <FTP relay host>: FTP bounce scan
<SNIP>
```

例如，TCP-SYN 掃描（ -sS ）是預設值之一，除非我們另有定義。也是最受歡迎的掃描方法之一。這種掃描方法可以每秒掃描數千個連接埠。 TCP-SYN 掃描會傳送一個帶有 SYN 標誌的封包，因此永遠無法完成三次握手，從而導致無法與被掃描連接埠建立完整的 TCP 連線。

- If our target sends a SYN-ACK flagged packet back to us, Nmap detects that the port is open.
如果我們的目標向我們發送一個帶有 SYN-ACK 標誌的封包，Nmap 就會偵測到該連接埠是 open 。
- If the target responds with an RST flagged packet, it is an indicator that the port is closed.
如果目標以具有 RST 標誌的資料包回應，則表示連接埠已 closed 。
- If Nmap does not receive a packet back, it will display it as filtered. Depending on the firewall configuration, certain packets may be dropped or ignored by the firewall.
如果 Nmap 沒有收到傳回的資料包，它將把它顯示為 filtered 。根據防火牆配置，某些資料包可能會被防火牆丟棄或忽略。

```bash
Coconut820@htb[/htb]$ sudo nmap -sS localhost

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 22:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```
在這個例子中，我們可以看到我們打開了四個不同的 TCP 連接埠。在第一列中，我們看到連接埠號碼。然後，在第二列中，我們看到服務的狀態以及它是什麼類型的服務。