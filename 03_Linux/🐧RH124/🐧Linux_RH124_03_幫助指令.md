# Linux_RH124_03_幫助指令
## 🐧man
### 搜索已知指令
- `man <命令|配置文件>`：查看幫助信息【常用】 
	```bash
	[root@localhost etc]# man ls
	```

- `man [章節數] <命令|配置文件>`：查看指定章節(Section)的幫助信息
	```bash
	[root@localhost ~]# man 1 ls
	```

	| Section(章節數) | 說明                                                     |
	| --------------- | -------------------------------------------------------- |
	| 1               | ⭐一般使用者會用到的指令，常用，但一般不會加章節數查詢   |
	| 2               | System cells，系統呼叫                                   |
	| 3               | 一般函式庫函數                                           |
	| 4               | 特殊檔案（通常位於 `/dev`）                              |
	| 5               | ⭐檔案格式與協定，如`/etc/paswd`，若要查詢一定要加章節數 |
	| 6               | 遊戲用                                                   |
	| 7               | 雜項(protocols, file systems)                            |
	| 8               | ⭐系統管理者會用到的指令，常用，但一般不會加章節數查詢   |
	| 9               | Linux kernel API                                         |

### 搜索未知指令
前提：whatis資料庫已建立，也就是建立mandb(root用戶)
每天凌晨都會更新mandb，whatis是從找相應指令的說明
```bash
[mickey@localhost ~]$ mandb
```
- `whatis <指令名>`，查詢指令
	```bash
	[root@localhost ~]# whatis passwd
	passwd (5)           - password file
	openssl-passwd (1ssl) - compute password hashes
	passwd (1)           - update user's authentication tokens
	```
- `man -k <查詢關鍵詞>`，搜索未知指令
	輸出格式：指令名(章節數)	- 指令說明
	```bash
	[root@localhost ~]# man -k passwd
	chgpasswd (8)        - update group passwords in batch mode
	chpasswd (8)         - update passwords in batch mode
	fgetpwent_r (3)      - get passwd file entry reentrantly
	(以下省略)
	```

### man page
|指令|說明|
|---|---|
|空白鍵<br/>↓<br/>J|⭐向下一行|
|↑<br/>K|⭐向上一行|
|PageDown|向下一頁|
|PageUp|向上一頁|
|/查詢字串|⭐字串搜索|
|N|⭐向下搜索|
|Shift + N|⭐向上搜索|
|G|畫面至頂|
|Shift + G|畫面至底|
|Q|離開man page|

## 🐧help
``help`：獲得shell內置命令的幫助信息
```bash
[root@localhost etc]# help cd
```

## 🐧pinfo
`pinfo <指令名>`，查看幫助信息(以網頁方式顯示)
```
[root@localhost ~]# pinfo passwd
```

### pinfo page
![Linux pinfo](https://www.cyberciti.biz/media/new/cms/2012/07/vivek@wks01-_033.png)
缺點：不能向上搜索關鍵字

**man page和pinfo page比對**
![function key](http://4.bp.blogspot.com/-oFYK3RFcxOE/VdqIQWid6yI/AAAAAAAAAVc/jPDEqOdbS3w/s400/10_57_46.bmp)

## 🐧取得說明文件
### 官網下載
[Red Hat Product Documentation](https://access.redhat.com/documentation/en-US/)
産品名：Red Hat Enterprise Linux

### man page轉pdf
1. man page-->`man -t <指令名> > <指令路徑>.ps`-->.ps文件
	postscript是印表機可直接讀取的格式
	```bash	
	[root@localhost ~]# man -t passwd > /home/mickey/passwd.ps
	[root@localhost ~]# ls /home/mickey/*.ps
	/home/mickey/passwd.ps
	```
2. .ps文件-->`ps2pdf <.ps文檔路徑>`-->.pdf文件
	```bash
	[root@localhost mickey]# ps2pdf passwd.ps
	[root@localhost mickey]# ls *.pdf
	passwd.pdf
	```