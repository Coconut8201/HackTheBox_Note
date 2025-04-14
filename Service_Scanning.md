# Service Scanning 服務掃描
- **確定作系統和可能正在運行的任何可用服務**

- 埠號(port)的範圍從 1 到 65,535
- port 的 1 ~ 1023 為特權服務保留
- port 0 是 TCP/UDP 中的保留埠(reserved port)
- 如果有任何內容嘗試綁定到port 0(例如服務)，它將綁定到port 1024 後的下一個可用port，因為埠 0 被視為“通配符”埠("wild card" port)。

## **Nmap 掃描工具**

透過這個掃描工具可以自動畫的掃描port 範圍
- 假設 目標ip 為 10.129.42.253
```
nmap 10.129.42.253
```
> **_Note_**
> <br />1. 如不指定任何額外選項，nmap 預這只掃描1000 個常用的port 
> <br />2. 默認情況下namp 進行的是tcp 掃描
> <br />3. STATE 有些情況下可能會出現不同狀態, 比如 **filtered** , 這個情況下代表 防火牆預設僅允許從特定ip address 訪問該port 
> <br />4. SERVICE 通常表示該服務的名稱，並對應到特定的埠號, 默認掃描(default scan)不會告訴我們該port 上正在監聽什麼

>[!Important] Nmap 在預設掃描（default scan）中所顯示的服務名稱，可能並不準確，因為它只是根據「埠號對應的常見服務」來推測的。

```
'nmap' option
-sC '嘗試獲取更詳細的資訊'
-sV '執行版本掃描。查出有哪些服務在運作，包含服務名稱與埠號。'
-p- '掃描全部 65,535 個TCP port'
-p  '指令port 進行scan'
-v  '顯示進行中掃描的進度過程資訊'
enter 查看當前掃描進度
```

### Nmap Scripts Nmap 腳本
-sC 將針對目標運行預設腳本，但某些情況下需要執行特定腳本:
<br /> 例如 指定對一套大型的 Citrix 系統進行稽核
<br /> 腳本網址: https://raw.githubusercontent.com/cyberstruggle/DeltaGroup/master/CVE-2019-19781/CVE-2019-19781.nse

``` 
Service Scanning 
$ locate scripts/citrix
```

running an Nmap script
```
nmap --script <script name> -p<port> <host> 
```

### Banner Grabbing 橫幅抓取
- Banner Grabbing(橫幅抓取)是一種快速對服務進行指紋識別的有用技術。通常，一旦啟動連接，服務就會通過顯示橫幅來標識自己

Nmap grab the banners:
```
 nmap -sV --script=banner <target>
 // ex: nmap -sV --script=banner -p21 10.10.10.0/24
```
NetCat grab the banners:
```
nc -nv ipAddr port
// ex:  nc -nv 10.129.42.253 21
```

## FTP (File Transfer Protocol) 
對 FTP 的預設埠(21)的 Nmap 掃描顯示了我們之前確定的 vsftpd 3.0.3 安裝。此外，它還報告已啟用匿名身份驗證並且 pub 目錄可用。
```
nmap -sC -sV -p21 10.129.42.253

Starting Nmap 7.80 ( https://nmap.org ) at 2020-12-20 00:54 GMT
Nmap scan report for 10.129.42.253
Host is up (0.081s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Dec 19 23:50 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
Service Info: OS: Unix
```

使用 ftp 命令行實用程式連接到服務。
```
ftp -p 10.129.42.253

Connected to 10.129.42.253.
220 (vsFTPd 3.0.3)
Name (10.129.42.253:user): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls
227 Entering Passive Mode (10,129,42,253,158,60).
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Feb 25 19:25 pub
226 Directory send OK.

ftp> cd pub
250 Directory successfully changed.

ftp> ls
227 Entering Passive Mode (10,129,42,253,182,129).
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            18 Feb 25 19:25 login.txt
226 Directory send OK.

ftp> get login.txt
local: login.txt remote: login.txt
227 Entering Passive Mode (10,129,42,253,181,53).
150 Opening BINARY mode data connection for login.txt (18 bytes).
226 Transfer complete.
18 bytes received in 0.00 secs (165.8314 kB/s)

ftp> exit
221 Goodbye.
```

> [!Note]
> - FTP 支援 cd 和 ls 等常用命令
> - 允許使用 get 命令下載檔案
> - 檢查下載的 login.txt 會顯示我們可以用來進一步訪問系統的憑據。


## SMB (Server Message Block)
SMB（伺服器消息塊）是 Windows 電腦上流行的協定，它為垂直和橫向移動提供了許多向量。敏感數據（包括憑據）可能位於網路文件共用中，並且某些 SMB 版本可能容易受到 RCE 漏洞（如 EternalBlue）的攻擊。仔細列舉這個相當大的潛在攻擊面至關重要。Nmap 有許多用於枚舉 SMB 的腳本，例如 smb-os-discovery.nse，它將與 SMB 服務交互以提取報告的作系統版本。

Service Scanning  服務掃描

```
Coconut820@htb[/htb]$ nmap --script smb-os-discovery.nse -p445 10.10.10.40

Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-27 00:59 GMT
Nmap scan report for doctors.htb (10.10.10.40)
Host is up (0.022s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: CEO-PC
|   NetBIOS computer name: CEO-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-12-27T00:59:46+00:00

Nmap done: 1 IP address (1 host up) scanned in 2.71 seconds
```
在這種情況下，主機運行的是舊版 Windows 7 作系統，我們可以進行進一步的枚舉以確認它是否容易受到 EternalBlue 的攻擊。Metasploit 框架有幾個用於 EternalBlue 的模組 ，可用於驗證漏洞並利用它，我們將在下一節中看到。我們可以針對此模組的目標部分運行掃描，以從 SMB 服務收集資訊。我們可以確定主機運行的是 Linux 內核，Samba 版本 4.6.2，主機名是 GS-SVCSCAN。
> [!Note] 我們可以針對此模組的目標部分運行掃描，以從 SMB 服務收集資訊。我們可以確定主機運行的是 Linux 內核，Samba 版本 4.6.2，主機名是 GS-SVCSCAN。

SMB 允許使用者和管理員共用資料夾，並允許其他使用者遠端存取這些資料夾。這些共用通常包含包含敏感資訊（如密碼）的檔案

### **smbclient**
smbclient 是一個可以列舉並與 SMB 分享資源互動的工具
```
'smbclient'
-N '禁止顯示密碼提示'
-L '取得遠端主機上可用的分享資源清單'
ex: smbclient -N -L \\\\10.129.42.253
```
這會顯示出非預設的共享使用者


以 guest 使用者身份進行連接
```
smbclient \\\\10.129.42.253\\users
```
以 Bob 使用者進行登錄聯接(不能使用-N)
```
smbclient -U bob \\\\10.129.42.253\\users
```

用get 指令可以下載 有訪問許可權的檔案
ex: get passwords.txt


### SNMP (SNMP Community strings)
製造商預設的 community string，像是 public 和 private，經常未被更改。在 SNMP 第 1 與第 2c 版中，存取是透過明文的 community string 來控制的；只要我們知道這個名稱，就能存取相關資源。

而加密與驗證機制，則是直到 SNMP 第 3 版才被加入。透過 SNMP，可以取得大量的資訊。

Service Scanning
```
Coconut820@htb[/htb]$ snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0

iso.3.6.1.2.1.1.5.0 = STRING: "gs-svcscan"
```

Service Scanning
```
Coconut820@htb[/htb]$ snmpwalk -v 2c -c private  10.129.42.253 

Timeout: No Response from 10.129.42.253
```

可以使用 onesixtyone 等工具使用常見社區字串的字典檔（例如該工具的 GitHub 儲存庫中包含的 dict.txt 檔）暴力破解社區字串名稱。

Service Scanning
```
Coconut820@htb[/htb]$ onesixtyone -c dict.txt 10.129.42.254

Scanning 1 hosts, 51 communities
10.129.42.254 [public] Linux gs-svcscan 5.4.0-66-generic #74-Ubuntu SMP Wed Jan 27 22:54:38 UTC 2021 x86_64
```