# Nmap Scripting Engine  Nmap 腳本引擎

Nmap 的腳本引擎（Nmap Scripting Engine，簡稱 NSE）是 Nmap 的另一個實用功能。它讓我們可以使用 Lua 語言撰寫腳本，來與特定的服務進行互動。這些腳本總共可以分為 14 個類別。

|類別|描述|
|---|---|
|auth 認證|確定身份驗證憑證|
|broadcast 廣播|通過廣播和發現的主機用於主機發現的腳本可以自動添加到其餘掃描中|
|brute 野蠻|執行嘗試使用憑據暴力破解來登錄相應服務的腳本|
|default 默認|使用 `-sC` 選項執行的預設腳本。|
|discovery 發現|評估無障礙服務|
|dos|這些腳本用於檢查服務是否存在拒絕服務漏洞，並且使用較少，因為它會損害服務|
|exploit 利用|此類腳本嘗試利用掃描埠的已知漏洞|
|external 外部|使用外部服務進行進一步處理的腳本|
|fuzzer 模糊測試|它使用腳本通過發送不同的字段來識別漏洞和意外的數據包處理，這可能需要很長時間|
|intrusive 侵入|可能對目標系統產生負面影響的侵入性腳本|
|malware 惡意軟體|檢查某些惡意軟體是否感染了目標系統|
|safe 安全|不執行侵入性和破壞性訪問的防禦性腳本|
|version 版本|用於服務檢測的擴展|
|vuln 漏洞|識別特定漏洞|


我們有幾種方法可以在 Nmap 中定義所需的腳本。

## Default Scripts  默認文本
```bash
Coconut820@htb[/htb]$ sudo nmap <target> -sC
```

## Specific Scripts Category 特定腳本類別
```bash
Coconut820@htb[/htb]$ sudo nmap <target> --script <category>
```

## Defined Scripts  定義的腳本
```bash
Coconut820@htb[/htb]$ sudo nmap <target> --script <script-name>,<script-name>,...
```

例如，讓我們繼續使用目標 SMTP 埠，並查看使用兩個定義的腳本獲得的結果。

## Nmap - 指定文稿
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 23:21 CEST
Nmap scan report for 10.129.2.28
Host is up (0.050s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
```

|scan options|描述|
|---|---|
|10.129.2.28|掃描指定的目標|
| -p 25 |僅掃描指定的port|
|--script banner,smtp-commands|使用指定的 NSE 命令稿|

我們可以看到，透過使用 banner 腳本，我們能夠辨識出目標系統是 Ubuntu 發行版的 Linux。而 smtp-commands 腳本則會顯示我們可以與目標 SMTP 伺服器互動時所能使用的指令。在這個例子中，這些資訊可能有助於我們找出目標系統中已存在的使用者。

Nmap 也提供了使用「aggressive」模式（-A）來掃描目標的能力。這個模式會同時使用多種掃描選項，例如服務偵測（-sV）、作業系統偵測（-O）、路由追蹤（--traceroute），以及預設的 NSE 腳本（-sC）來進行全面性的掃描。

## Nmap - Aggressive Scan  Nmap - 主動掃描
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -A
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-17 01:38 CEST
Nmap scan report for 10.129.2.28
Host is up (0.012s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), 
AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Netgear RAIDiator 4.2.28 (94%), 
Linux 2.6.32 - 2.6.35 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

TRACEROUTE
HOP RTT      ADDRESS
1   11.91 ms 10.129.2.28

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.36 seconds
```
|scan options|描述|
|---|---|
| -A|執行服務檢測、作系統檢測、traceroute 並使用預設腳本掃描目標|

透過使用的掃描選項（-A），我們找出了系統上運行的是哪種 Web 伺服器 （Apache 2.4.29），使用了哪種 Web 應用程式 （WordPress 5.3.4），以及網頁的標題 （blog.inlanefreight.com）。外，Nmap 也顯示該作業系統極有可能是 Linux（機率為 96%）

## Vulnerability Assessment  漏洞評估
現在讓我們來看看 HTTP 的 80 號埠，並使用 NSE 中的 vuln 類別來探索可以找到哪些資訊和潛在的漏洞

### Nmap - Vuln Category  Nmap - 漏洞類別
```bash
Coconut820@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sV --script vuln 

Nmap scan report for 10.129.2.28
Host is up (0.036s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-wordpress-users:
| Username found: admin
|_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|     	CVE-2019-0211	7.2	https://vulners.com/cve/CVE-2019-0211
|     	CVE-2018-1312	6.8	https://vulners.com/cve/CVE-2018-1312
|     	CVE-2017-15715	6.8	https://vulners.com/cve/CVE-2017-15715
<SNIP>
```

|scan options|描述|
|---|---|
|10.129.2.28|掃描指定的目標|
| -p 80 |僅掃描指定的port|
| -sV |對指定埠執行服務版本檢測|
|--script vuln|使用指定類別中所有相關的腳本|

在上一輪掃描中所使用的腳本，會與網頁伺服器及其網頁應用程式互動，以取得更多關於其版本的資訊，並查詢各種資料庫，看看是否存在已知的漏洞。