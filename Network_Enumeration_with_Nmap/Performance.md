# Performance  性能

當我們需要掃描大型網路或處理低頻寬網路時，掃描效能扮演著重要角色。
我們可以使用各種選項來告訴 Nmap 掃描時的速度（-T <0-5>）、並行連線的頻率（--min-parallelism <數量>）、封包的最大逾時時間（--max-rtt-timeout <時間>）、同時發送的封包數量（--min-rate <數量>），以及掃描目標的重試次數（--max-retries <數量>）

## Timeouts  超時
當 Nmap 發送一個封包時，從掃描的端口接收到回應會需要一段時間（稱為往返時間，Round-Trip-Time，簡稱 RTT）。
通常，Nmap 會以較高的初始逾時設定開始（--min-rtt-timeout），預設為 100 毫秒。
讓我們透過一個範例來了解：掃描一個包含 256 台主機的整個網路，並針對每台主機的前 100 個常見端口進行掃描。

### Default Scan  默認掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

### Optimized RTT  優化的 RTT
```
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms

<SNIP>
Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

|Scanning Options 掃描選項 | Description 描述|
|---|---|
|10.129.2.0/24 | 掃描指定的目標網路。|
|-F | 掃描前 100 個埠。|
|--initial-rtt-timeout 50ms | 將指定的時間值設為初始 RTT 逾時。|
|--max-rtt-timeout 100ms | 將指定的時間值設為最大 RTT 逾時。|

在比較兩次掃描時，我們可以看到，使用優化掃描后，我們發現少了兩台主機(host)，但掃描只花了四分之一的時間。由此我們可以得出結論，將初始 RTT 超時 （--initial-rtt-timeout） 設置為太短的時間段可能會導致我們忽略主機。

提高掃描速度的另一種方法是指定已發送數據包的重試率 （--max-retries）。默認值為 10，但我們可以將其減少為 0。這意味著如果 Nmap 沒有收到一個埠的回應，它就不會再向那個埠發送任何數據包，而是跳過它。

### Default Scan  默認掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l

23
```

### Reduced Retries  減少重試次數
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l

21
```
|Scanning Options 掃描選項 | Description 描述|
|---|---|
|10.129.2.0/24 | 掃描指定的目標網路。|
|-F | 掃描前 100 個埠。|
|--max-retries 0 | 設置掃描期間將執行的重試次數。|

同樣，我們認識到加速也會對我們的結果產生負面影響，這意味著我們可能會忽略重要資訊。

## Rates  速率
在白箱滲透測試（white-box penetration test）期間，我們有時會被列入安全系統的白名單，
以便能檢查網路中的系統是否存在漏洞，而不僅僅是測試防護措施。
如果我們了解網路頻寬，就可以調整封包的發送速率，這能大幅加快使用 Nmap 的掃描速度。
當設定封包發送的最小速率（--min-rate <數量>）時，我們是告訴 Nmap 要同時發送指定數量的封包，
Nmap 會試圖維持這個速率來進行掃描。

### Default Scan  默認掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```

### Optimized Scan  優化掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```
|Scanning Options 掃描選項 | Description 描述|
|---|---|
|10.129.2.0/24 | 掃描指定的目標網路。|
|-F | 掃描前 100 個埠。|
|-oN tnet.minrate300 | 以正常格式保存結果，並啟動指定的檔名。|
|--min-rate 300 （最小速率 300） | 設置每秒要發送的最小封包數量。|

### Default Scan - Found Open Ports 默認掃描 - 找到打開的埠
```bash
Coconut820@htb[/htb]$ cat tnet.default | grep "/tcp" | wc -l

23
```

### Optimized Scan - Found Open Ports 優化掃描 - 找到打開的埠
```bash
Coconut820@htb[/htb]$ cat tnet.minrate300 | grep "/tcp" | wc -l

23
```

## Timing  定時
```bash
由於在黑箱滲透測試（black-box penetration test）中，這類設定無法總是手動最佳化，
Nmap 提供了六種不同的時間範本（-T <0-5>）供我們使用。
這些數值（0-5）決定了掃描的積極程度。
不過，如果掃描過於積極，也可能產生負面影響，
例如因為產生大量網路流量而被安全系統封鎖。
若我們沒有特別指定其他設定，Nmap 預設使用的時間範本是一般模式（-T 3）。
```

|選項 | 模式名稱 | 說明（簡述）|
|---|---|
|-T 0 | paranoid | 偏執模式（非常慢，避免被偵測）|
|-T 1 | sneaky | 隱匿模式（很慢，降低偵測風險）|
|-T 2 | polite | 禮貌模式（比正常慢，減少網路負載）|
|-T 3 | normal | 正常模式（Nmap 預設模式）|
|-T 4 | aggressive | 積極模式（較快，但容易被偵測）|
|-T 5 | insane | 瘋狂模式（非常快，但極高被偵測風險）|

### Default Scan  默認掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default 

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 32.44 seconds
```

### Insane Scan  瘋狂掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5

<SNIP>
Nmap done: 256 IP addresses (10 hosts up) scanned in 18.07 seconds
```

|Scanning Options 掃描選項 | Description 描述|
|---|---|
|10.129.2.0/24 | 掃描指定的目標網路。|
|-F | 掃描前 100 個埠。|
|-oN tnet.T5 | 以正常格式保存結果，並啟動指定的檔名。|
|-T 5 | 指定使用瘋狂模式（insane）的時間範本。Scanning Options 掃描選項 | Description |描|述|
|10.129.2.0/24 | 掃描指定的目標網路。|
|-F | 掃描前 100 個埠。|
|-oN tnet.T5 | 以正常格式保存結果，並啟動指定的檔名。|
|-T 5 | 指定使用瘋狂模式（insane）的時間範本。|

### Default Scan - Found Open Ports
```bash
Coconut820@htb[/htb]$ cat tnet.default | grep "/tcp" | wc -l

23
```

### Insane Scan - Found Open Ports
```bash
Coconut820@htb[/htb]$ cat tnet.T5 | grep "/tcp" | wc -l

23
```