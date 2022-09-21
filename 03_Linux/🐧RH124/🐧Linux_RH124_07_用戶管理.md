# Linux用戶管理
- /home目錄下有各個創建的用戶對應的家目錄(如：/home/mickey)，當用戶登錄時，會自動進入自己的家目錄中。
- Linux用戶需要至少屬於一個組。

## 🐧使用者分類
|UID|類別|說明|
|---|---|---|
|0|superuser account<br/>系統管理員(root)|擁有所有權限|
|1~999|system user<br/>系統使用者|系統所使用的身分(軟體驅動)，用於啟動系統服務<br/>1~200為Red Hat開發<br/>201~999為其他公司開發|
|1000~|regular user<br/>一般使用者||

## 🐧相關文件
用戶操作時會影響到的文檔【不建議直接改動文件】：
- /etc/passwd：存放用戶信息
- /etc/shadow：存放用戶密碼的影子文件
- /etc/group：存放組信息
- /etc/gshadow：存放組密碼的影子文件
- Home Directory，家目錄，一般在/home/用戶名
- 其他與用戶有關的套件安裝路徑，如：mail server

### /etc/passwd
```bash
mickey:x:1000:1000:mickey:/home/mickey:/bin/bash
student:x:1001:1001::/home/student:/bin/bash
```
1. 帳號名，mickey
2. 密碼
	- 影子處理過顯示x，並將密碼移至/etc/shadow，並且只有root可看
	- 否則顯示雜湊處理後的值
3. UID，1000
4. GID，主要群組，1000
5. 注釋，mickey
6. 家目錄路徑，/home/mickey
7. 使用者默認使用的Shell類型，/bin/bash

### /etc/shadow
```bash
[root@localhost usr]# grep mickey /etc/shadow
mickey:$6$MrFag8B1jzGXIIF1$QNA1Fa9CKUv99QfJDgVdIUoMqKjaHVIrn8ylo1tlm.bRdtQOkZ6R9Q8KOhAoa7FPhKz6qRtqEHEtZDXt4h6cg1:18706:0:99999:7:::
```
1. 帳號，mickey
2. 密碼(雜湊)
	1. 雜湊演算法類型，$6
		- $1，采用MD5算法加密
		- $5，采用SHA256算法加密
		- $6，采用SHA512算法加密
	1. key值(亂數)，對密碼進行hash的干擾值，$y.kLOoNS3jOjuy5g
	2. 密碼，$STcrjA7KqaA...(中略)…SUK1w/
3. 最後一次密碼變更日期(從1970年1月1日算起的天數)，18706
	- 若欄位為0，代表此使用者下次登入時要改密碼
	- 若欄位為空，代表此使用者的密碼不會過期
4. 密碼最長使用期限(單位天)，0
	- 若欄位為0或空，代表密碼不會過期
5. 密碼過期警告期限(單位天)，99999
	- 若欄位為0或空，代表不會警告
6. 密碼過期後讓使用者還有多少天可以登入改密碼
	- 若欄位空，代表此功能停用
7. 帳號過期時間(從1970年1月1日算起的天數)
	- 若欄位空，代表此帳號還沒過期過
8. 保留欄位(暫無用處)

### /etc/group
```
student:x:1001:user01,user02,user03
mickey:x:1000:mickey
```
1. 群組名，student
2. 群組密碼，影子處理過顯示x，並將密碼移至/etc/gshadow，現在沒在用
3. GID
4. 群組成員，user01,user02,user03
	- 可在此添加用戶的次要群組，如mickey的主要群組為mickey，次要群組為student
```bash
/etc/group
student:x:1001:user01,user02,user03,mickey

/etc/passwd
mickey:x:1000:1000:mickey:/home/mickey:/bin/bash
```


## 🐧shadow開啟、關閉
默認狀況下shadow是啟動的，也不建議關閉

### 開啟 pwunconv
```bash
[root@localhost ~]# pwunconv
[root@localhost ~]# ls /etc/shadow
ls: cannot access '/etc/shadow': No such file or directory
```

### 關閉 pwconv
```bash
[root@localhost ~]# pwconv
[root@localhost ~]# ls /etc/shadow
/etc/shadow
```

## 🐧查詢用戶信息
### id
* `id <用戶名>`
	1. uid：用戶
	2. gid：主要群組
	3. groups：次要群組
```bash
[root@localhost home]# id mickey
uid=1000(mickey) gid=1000(mickey) groups=1000(mickey),1001(student) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@localhost home]# id mike
id: mike: no such user
```

### whoami
* `whoami`，查看當前用戶
* `who am i`，同樣效果
```bash
[root@localhost home]# whoami
root
[root@localhost home]# who am i
ace      pts/1        2020-01-05 13:54 (gateway)
```

## 🐧用戶管理
### useradd
* `useradd [-d <所對應家目錄>] <用戶名>`：添加用戶至指定家目錄
	新增用戶至已存在的家目錄
	```bash
	[root@localhost home]# useradd -d /home/mike jack
	useradd: warning: the home directory already exists.
	```
	新增用戶至自訂名稱的家目錄
	```bash
	[root@localhost home]# useradd -d /home/progammer tom
	[root@localhost home]# ls
	mickey  mike  progammer
	[root@localhost home]# ll
	總計 4
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	drwx------.  3 mike   mike     78  1月  5 13:11 mike
	drwx------.  3 tom    tom      78  1月  5 13:22 progammer
	```
* `useradd [-g <用戶組>] <用戶名>`：添加用戶至指定組
	```bash
	[root@localhost home]# useradd -g mickey marry
	[root@localhost home]# id marry
	uid=1004(marry) gid=1000(mickey) groups=1000(mickey)
	[root@localhost home]# ll
	總計 4
	drwx------.  3 marry  mickey   78  1月  5 14:16 marry
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	drwx------.  3 ace    ace      78  1月  5 13:22 progammer
	drwx------.  5 ace    ace     107  1月  5 13:54 programmer
	```
* `useradd <用戶名>`：添加無密碼用戶，但無密碼的用戶無法登入
	1. 當創建用戶成功後，會自動創建和用戶同名的家目錄
	2. 若沒有指定新增用戶所對應的組名的話，會自動建立與用戶名相同名稱的組名，並將用戶放至新組下
	```bash
	[root@localhost home]# ll
	總計 4
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	[root@localhost home]# useradd mike
	[root@localhost home]# ll
	總計 4
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	drwx------.  3 mike   mike     78  1月  5 13:11 mike
	```

### passwd
* `passwd [用戶名]`：若無指定則更改自己的密碼，root用戶才可以更改別人密碼
	```bash
	[root@localhost ~]# passwd student
	Changing password for user student.
	New password:
	BAD PASSWORD: The password is shorter than 8 characters
	Retype new password:
	passwd: all authentication tokens updated successfully.
	```

### usermod
- `usermod -g <用戶名>`，修改用戶主要群組
	```bash
	[root@localhost ~]# id mickey
	uid=1000(mickey) gid=1000(mickey) groups=1000(mickey),1001(student)
	[root@localhost ~]# usermod mickey -g student
	[root@localhost ~]# id mickey
	uid=1000(mickey) gid=1001(student) groups=1001(student)
	```
- `usermod -G <用戶名>`，修改用戶次要群組，`-a`用於覆蓋
	```bash
	[root@localhost ~]# id mickey
	uid=1000(mickey) gid=1001(student) groups=1001(student),1000(mickey)
	[root@localhost ~]# usermod mickey -G mickey,wheel
	[root@localhost ~]# id mickey
	uid=1000(mickey) gid=1001(student) groups=1001(student),10(wheel),1000(mickey)
	```
- `usermod -c <注解內容> <用戶名>`，給指定用戶加注解
	```bash
	[root@vb101 ~]# tail -n 5 /etc/passwd
	student:x:1001:1001::/home/student:/bin/bash
	apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
	operator1:x:1002:1002::/home/operator1:/bin/bash
	operator2:x:1003:1003::/home/operator2:/bin/bash
	operator3:x:1004:1004::/home/operator3:/bin/bash
	[root@vb101 ~]# usermod -c "Operator One" operator1
	[root@vb101 ~]# tail -n 5 /etc/passwd
	student:x:1001:1001::/home/student:/bin/bash
	apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
	operator1:x:1002:1002:Operator One:/home/operator1:/bin/bash
	operator2:x:1003:1003::/home/operator2:/bin/bash
	operator3:x:1004:1004::/home/operator3:/bin/bash
	```
- `usermod -d <目錄名> <用戶名>`，改變該用戶登陸的初始目錄
	```bash
	[root@localhost ~]# usermod -d /root/下載 jack
	```
- `usermod -L <用戶名>`，鎖定用戶
- `usermod -U <用戶名>`，解除鎖定用戶

### userdel
- `userdel <用戶名>`：刪除用戶(/etc底下的用戶信息)，但保留家目錄及其他server相關的用戶信息
	```bash
	[root@localhost home]# userdel tom
	[root@localhost home]# ll
	總計 4
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	drwx------.  3 mike   mike     78  1月  5 13:11 mike
	drwx------.  3   1003   1003   78  1月  5 13:22 progammer
	```
- `userdel -r <用戶名>`：【建議使用】刪除用戶和家目錄，以及其他server相關的用戶信息
	```bash
	[root@localhost home]# userdel -r mike
	[root@localhost home]# ll
	總計 4
	drwx------. 16 mickey mickey 4096  1月  5 11:53 mickey
	drwx------.  3   1003   1003   78  1月  5 13:22 progammer
	```
- `find <路徑> -nouser -o -nogroup`，找無所屬用戶或無所屬群組的文檔

## 🐧用戶密碼管理
### chage
- `chage -l <用戶名>`，顯示指定用戶/etc/shadow的內容
	```bash
	[root@localhost ~]# chage -l mickey
	Last password change                                    : Mar 20, 2021
	Password expires                                        : Jun 18, 2021
	Password inactive                                       : Aug 17, 2021
	Account expires                                         : Jun 20, 2021
	Minimum number of days between password change          : 0
	Maximum number of days between password change          : 90
	Number of days of warning before password expires       : 20
	```
	1. Last password change，上次密碼更改日
	2. Password expires，密碼過期
	3. Password inactive，密碼無效
	4. Account expires，帳戶過期日
	5. Minimum number of days between password change，兩次密碼更改的最小天數
	6. Maximum number of days between password change，兩次密碼更改的最大天數
	7. Number of days of warning before password expires，密碼過期前的警告天數
- 修改指定用戶密碼設置
	```bash
	[root@localhost ~]# chage -m 0 -M 90 -W 20 -I 60 mickey
	```
	![[Linux_RH124_08_用戶管理_01_密碼管理.png]]
- `chage -E <YYYY-mm-dd> <用戶名>`，設置指定用戶的密碼有效日期；0表示立即過期，-1表示永不過期
	`date -d +<天數>days +%Y-%m-%d`，計算幾天後的日期
	```bash
	[root@localhost ~]# chage -E 2021-06-20 mickey
	[root@localhost ~]# chage -E $(date -d +90days +%Y-%m-%d) mickey
	```
- 修改新建用戶的密碼過期相關設置路徑：/etc/login.defs
	```bash
	[root@vb101 ~]# cat /etc/login.defs | grep ^[A-Z]
	MAIL_DIR        /var/spool/mail
	PASS_MAX_DAYS   99999
	PASS_MIN_DAYS   0
	PASS_MIN_LEN    5
	PASS_WARN_AGE   7
	UID_MIN                  1000
	UID_MAX                 60000
	SYS_UID_MIN               201
	SYS_UID_MAX               999
	GID_MIN                  1000
	GID_MAX                 60000
	SYS_GID_MIN               201
	SYS_GID_MAX               999
	CREATE_HOME     yes
	UMASK           077
	USERGROUPS_ENAB yes
	ENCRYPT_METHOD SHA512
	```

### usermod
- `usermod -L <用戶名>`，鎖定用戶
- `usermod -L -e <YYYY-mm-dd> <用戶名>`，用戶在指定日期過期
	```bash
	[root@localhost ~]# chage -l mickey
	Last password change                                    : Mar 20, 2021
	Password expires                                        : Jun 18, 2021
	Password inactive                                       : Aug 17, 2021
	Account expires                                         : Jun 20, 2021
	Minimum number of days between password change          : 0
	Maximum number of days between password change          : 90
	Number of days of warning before password expires       : 20
	[root@localhost ~]# usermod -L -e 2021-06-30 mickey
	[root@localhost ~]# chage -l mickey
	Last password change                                    : Mar 20, 2021
	Password expires                                        : Jun 18, 2021
	Password inactive                                       : Aug 17, 2021
	Account expires                                         : Jun 30, 2021
	Minimum number of days between password change          : 0
	Maximum number of days between password change          : 90
	Number of days of warning before password expires       : 20
	```
- `usermod -U <用戶名>`，解除鎖定用戶
- `usermod -s /sbin/nologin <用戶名>`，將用戶寫入/sbin/nologin中，指定用戶無法登入
	```bash	
	[root@localhost ~]# usermod -s /sbin/nologin student
	[root@localhost ~]# su - student
	This account is currently not available.
	```

## 🐧群組管理
### groupadd
用戶組：類似於角色，系統可能對有共性的多個用戶進行統一的管理。
- `groupadd <組名>`，增加組
	```bash
	[root@localhost ~]# groupadd testTeam
	```
	1. `-r`，增加系統組
	```bash
	[root@vm104 ~]# groupadd -r systemgroup
	[root@vm104 ~]# tail -n 3 /etc/group
	nginx:x:975:
	mysql:x:27:
	systemgroup:x:974:
	```

### groupmod
- `-g`，修改GID
	```bash
	[root@vm104 ~]# groupmod -g 123 systemgroup
	```
- `-n`，修改群組名
	```bash
	[root@vm104 ~]# groupmod -n mysysgroup systemgroup
	```

### groupdel
- `groupdel <組名>`，刪除組
	```bash
	[root@localhost home]# groupdel testTeam
	```

## 🐧身份轉換
### 更換用戶 su
- 切換情境：
	1. 一般用戶-->一般用戶
	2. 一般用戶-->root
	3. root-->一般用戶
- `su - [用戶名]`，【建議使用】更換用戶(帶更改後用戶的環境變數)，若無指定用戶名則為root
	```bash
	[mickey@localhost ~]$ su - student
	Password:
	[student@localhost ~]$ exit
	logout
	[mickey@localhost ~]$
	```
- `su [用戶名]`，更換用戶(不帶更改後用戶的環境變數)

### sudo
在執行sudo時會先查看/etc/sudoers中此用戶是否被授權，有的話才會使用root權限執行
![[Linux_RH124_08_用戶管理_02_sudo流程.png]]

- `sudo -i <指令>`，嘗試登入root，用於測試此用戶是否有sudo權限
	```bash
	[mickey@localhost ~]$ sudo -i ls
	[sudo] password for mickey:
	mickey is not in the sudoers file.  This incident will be reported.
	```
- `sudo <指令>`，以root的身分執行指令
	注意：sudo密碼有效時間為10分鐘
- /etc/sudoers及/etc/sudoers.d/\*設定狦戶使用sudo的權限
	```bash
	%wheel        ALL=(ALL)       NOPASSWD: ALL
	```
	1. %wheel，群組；無%代表指定用戶，如：user01
	2. ALL=(ALL)，可從任何終端機，執行所有指令
	3. NOPASSWD: ALL，不需要密碼轉為所有用戶