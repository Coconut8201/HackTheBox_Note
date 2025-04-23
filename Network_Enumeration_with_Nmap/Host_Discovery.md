# Host Discovery  主機發現
當我們需要對一家公司的整個網路進行內部滲透測試時，首先應該做的就是了解有哪些系統在線上，並且我們可以操作的。為了了解發現網路上的這些系統，我們可以使用nmap 提供的各種主機偵測選項。nmap 提供了許多方法判斷目標是否在線，其中座有效的host 偵測方式就是使用**ICMP 回應請求**

始終建議存儲每一次掃描。這稍後可用於比較、記錄和報告。畢竟，不同的工具可能會產生不同的結果。因此，區分哪個工具產生哪些結果可能是有益的。

## Scan Network Range  掃描網路範圍
```
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```
| 掃描選項 | 描述 |
| ---     | --- |
|10.129.2.0/24|目標網路範圍|
|-sn|禁用port 掃描|
|-oA tnet|將所有檔名以'tnet'為開頭的結果儲存|

## Scan IP List  掃描 IP 清單
在進行內部滲透測試時，我們經常會被提供一份包含目標主機的 IP 清單。Nmap 也提供了一個方便的選項，讓我們可以直接從這個清單中讀取要掃描的主機，而不用一個一個手動輸入或定義。

清單可能看起來像這樣
```
Coconut820@htb[/htb]$ cat hosts.lst

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

如果我們使用相同的掃描技術針對預先定義的 IP 清單進行掃描，指令會像這樣：
```
Coconut820@htb[/htb]$ sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

| 掃描選項 | 描述 |
| ---     | --- |
|-sn|禁用port 掃描|
|-oA tnet|將所有檔名以'tnet'為開頭的結果儲存|
|-iL|執行針對 hosts.lst 檔案中所提供目標的定義掃描|

在這個範例中，我們可以看到 7 台主機中只有 3 台是活躍的。請記住，這可能表示其餘的主機因為防火牆的設定而忽略了預設的 ICMP 回應請求。由於 Nmap 沒有收到回應，它就會將這些主機標記為非活動主機。

## Scan Multiple IPs  掃描多個 IP
有時我們只需要掃描網路中的一小部分。這時，一種替代上次使用的方法是直接指定多個 IP 位址來進行掃描。
```
Coconut820@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

如果這些 IP 位址是連續的，我們也可以在對應的八位元組中定義一個範圍來表示它們。
```bash
Coconut820@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```
grep for | cut -d" " -f5 => 從輸入中找出包含 for 的行，然後取出每行的第 5 個欄位

## Scan Single IP  掃描單個 IP
在掃描單個主機的開放埠及其服務之前，我們首先必須確定它是否處於活動狀態。為此，我們可以使用與以前相同的方法
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-14 23:59 CEST
Nmap scan report for 10.129.2.18
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```
| 掃描選項 | 描述 |
| ---     | --- |
|10.129.2.28|對目標執行定義的掃描|
|-sn|禁用port 掃描|
|-oA host|將所有檔名以'host'為開頭的結果儲存|

如果使用了 -sn 禁用port 掃描，nmap 會自動使用 **ICMP Echo Request (-PE)** 進行ping 掃描。當發出這種請求時，如果目標主機是活的，我們通常會預期它會回傳一個 ICMP 回應。比較有趣的是，先前的掃描並沒有這樣做。那是因為，在 Nmap 發出 ICMP Echo Request 之前，它其實已經先發送了 ARP ping，而主機會回應一個 ARP 回應。這就是為什麼我們會知道主機是活的，即使沒有看到 ICMP。我們可以使用 --packet-trace 選項來驗證這一點，它會顯示 Nmap 實際發送的封包內容。若我們希望確保一定使用 ICMP Echo Request 來檢查主機是否在線，則可以手動加上 -PE 參數來強制執行這種方式。
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

| 掃描選項 | 描述 |
| ---     | --- |
|10.129.2.18|對目標執行定義的掃描|
|-sn|禁用port 掃描|
|-oA host|將所有檔名以'host'為開頭的結果儲存|
|-PE|使用ICMP 對目標執行ping 掃描|
|--packet-trace|顯示和發送接收的所有數據包|
|--reason| 在每個主機的掃描結果中附上理由，說明它為什麼認為這個主機是「alive」或「unreachable」|

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:10 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency). => **--reason 結果**
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

| 掃描選項 | 描述 |
| ---     | --- |
|10.129.2.18|對目標執行定義的掃描|
|-sn|禁用port 掃描|
|-oA host|將所有檔名以'host'為開頭的結果儲存|
|-PE|使用ICMP 對目標執行ping 掃描|
|--reason|顯示特定結果的原因|
|-O|判斷作業系統|

Nmap 確實僅通過 ARP 請求和 ARP 回復來檢測主機是否存活。要禁用 ARP 請求並使用所需的 ICMP 回顯請求掃描我們的目標，我們可以通過設置“--disable-arp-ping”選項來禁用 ARP ping。然後，我們可以再次掃描目標並查看發送和接收的數據包。

```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

ICMP 回顯請求可以幫助我們確定目標是否處於活動狀態並識別其系統。

## ICMP Echo Reply 的 TTL 值判斷作業系統:
|作業系統|預設TTL|
|---|---|
|Windows|128|
|Linux/Unix|64|
|Cisco/NetWorking|255|
|BSD/MacOs|64~255(視版本而定)|