1. nmap -sV 10.129.205.212 -Pn -n
    全掃就對了

2. smbclient -L 10.129.205.212

3. smbclient -N 10.129.205.212

4. smbclient \\\\10.129.205.212\\WorkShares

5. smbclient \\\\10.129.205.212\\root
   => tree connect failed: NT_STATUS_BAD_NETWORK_NAME

6. smbclient \\\\10.129.205.212\\WorkShares
7. 

