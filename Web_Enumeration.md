# Web Enumeration 網站枚舉
## Gobuster
在發現一個網站應用程式(web application)後，通常很值得檢查是否能找出一些不應對外公開的隱藏檔案或目錄。我們可以使用像是 ffuf 或 GoBuster 這類工具來進行目錄枚舉。有時候，我們可能會發現隱藏的功能、頁面或目錄，這些可能會暴露敏感資料，進而被利用來存取網站應用程式，甚至遠端執行網站伺服器上的程式碼。

GoBuster 是一個多功能工具，可用於執行 DNS、虛擬主機（vhost）與目錄的暴力破解。它還具備其他功能，例如列舉公開的 AWS S3 儲存桶。在本模組中，我們主要關注的是透過 dir 參數所指定的目錄（與檔案）暴力破解模式。接下來，我們將使用 dirb 的 common.txt 字典來進行一次簡單的掃描。

Web Enumeration  
```
Coconut820@htb[/htb]$ gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.121/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/11 21:47:25 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.php (Status: 200)
/server-status (Status: 403)
/wordpress (Status: 301)
===============================================================
2020/12/11 21:47:46 Finished
=============================================================== 
```
HTTP 狀態代碼 200 表示資源的請求成功，而 403 HTTP 狀態代碼表示我們被禁止訪問資源。301 狀態代碼表示我們正在重定向，這不是失敗情況。

## Banner Grabbing / Web Server Headers

Banner Grabbing / Web Server Headers

```
curl -IL  '查網站的 HTTP 回應標頭（header），而不會下載整個網頁內容。'
```
ex:
```
Coconut820@htb[/htb]$ curl -IL https://www.inlanefreight.com

HTTP/1.1 200 OK
Date: Fri, 18 Dec 2020 22:24:05 GMT
Server: Apache/2.4.29 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8
```

> [!Note]
> 另一個方便的工具是 EyeWitness，它可用於截取目標 Web 應用程式的螢幕截圖、對它們進行指紋識別並識別可能的預設憑據。

### Whatweb
使用命令行工具 whatweb 提取 Web 伺服器、支援框架和應用程式的版本。此資訊可以幫助我們查明正在使用的技術並開始搜索潛在的漏洞。
```
whatweb 10.10.10.121
 whatweb --no-errors 10.10.10.0/24
```
是一個方便的工具，包含許多功能，可以跨網路自動枚舉 Web 應用程式。