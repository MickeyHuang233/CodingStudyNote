# Linux_RH134_02_任務排程
- at單次排程
- crond周期性排程：使用[[⏱cron定時表達式]])執行的特定的命令或腳本(Shell)
	1. system cron
	2. user cron

[Linux 設定 crontab 例行性工作排程教學與範例](https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/)

## 🐧重複執行指令 watch
檢查排程是否執行

- `watch <指令>`
	1. `-n <秒數>`，每n秒執行一次，無指定則為3秒
	```bash
	[mickey@mickey ~]$ watch "ls -ail ~"
	```

## 🐧單次排程 at
at排程寫在/var/spool/at/，因此就算重新開機後排程還在
```bash
[root@mickey ~]# ll /var/spool/at/
total 4
-rwx------. 1 mickey student 3204 Apr 13 21:31 a00002019ba3b0
drwx------. 2 root   root       6 Apr 13 21:23 spool
```

### 設置at排程
- `at <執行時間>`
	1. `now +5 min`，五分鐘後執行
	2. `teatime tomorrow`，明天下午四點執行
	3. `noon +4 days`，四天後中午十二點執行
	4.  `5pm august 3 2021`或`16:05`，指定時間執行
- `-q <a~z>`，指定排程隊列
- ==Ctrl + D==，結束並儲存設罝單次排程
	```bash
	[mickey@mickey ~]$ at now + 5min
	warning: commands will be executed using /bin/sh
	at> ls -ail /home/mickey > /home/mickey/atTime.txt
	at> date >> /home/mickey/atTime.txt
	at> <EOT>
	job 1 at Tue Apr 13 21:23:00 2021
	```
- 非交談式指定at排程
	1. `echo "執行指令" | at now +5 min`
	2. `at now +5 min < myscript.txt`，執行的script寫在文件

### 檢查at排程
- `atq`
	![Linux_RH134_02_任務排程_01_檢查排程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_02_%E4%BB%BB%E5%8B%99%E6%8E%92%E7%A8%8B_01_%E6%AA%A2%E6%9F%A5%E6%8E%92%E7%A8%8B.png?raw=true)
- `at -c <排程ID>`，查看指定排程詳細信息，也就是看/var/spool/at/中的文件內容
	`-c`-->cat
```bash
[root@mickey ~]# at -c 2 | head -n 3
#!/bin/sh
# atrun uid=1000 gid=1001
# mail mickey 0
```

### 刪除at排程
`atrm <排程ID>`
```bash
[mickey@mickey ~]$ atq
3       Thu Jul  1 16:00:00 2021 g mickey
4       Thu Jul  1 16:00:00 2021 g mickey
5       Thu Jul  1 16:05:00 2021 b mickey
[mickey@mickey ~]$ atrm 3
[mickey@mickey ~]$ atq
4       Thu Jul  1 16:00:00 2021 g mickey
5       Thu Jul  1 16:05:00 2021 b mickey
```

### at排程使用權限
- /etc/at.allow存在，則at.allow中的user可使用at排程
- /etc/at.allow不存在
	- /etc/at.deny存在，默認，則at.deny中的user不可使用at排程
	```bash
	[mickey@mickey ~]$ sudo cat /etc/at.deny
	mickey
	[mickey@mickey ~]$ at now +5 min
	You do not have permission to use at.
	```
	- /etc/at.deny不存在，只有root可用at

## 🐧system cron
- `man 5 crontab`，說明文件

### 方法一：排程設定檔
系統排程設定檔路徑/etc/crontab
`run-parts`，表示執行目錄下的所有shell
```bash
[mickey@mickey ~]$ cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
10 * * * *      mickey  ls- ail > /home/mickey/systemcron.txt
12 * * * *      mickey  run-parts /home/mickey/cron.d
```

### 方法二：排程路徑
- 可以在排程設定檔放至/etc/cron.d/
- 也可按要執行指定頻率的shell script放在指定資料夾
	1. /etc/cron.hourly，每個小時一開始時執行
	2. /etc/cron.daily，每天一開始時執行
	3. /etc/cron.weekly，每周一開始時執行
	4. /etc/cron.monthly，每月一開始時執行

### .timer服務
- system中*.timer的服務，主要作用為計時器，會有對應的*.service服務配套，時間到會觸發*.service
	相關指令可參考：[[🐧Linux_RH124_10_服務管理]]
	```bash
	[mickey@mickey system]$ systemctl --type=timer
	UNIT                         LOAD   ACTIVE SUB     DESCRIPTION
	dnf-makecache.timer          loaded active waiting dnf makecache --timer
	systemd-tmpfiles-clean.timer loaded active waiting Daily Cleanup of Temporary Directories
	unbound-anchor.timer         loaded active waiting daily update of the root trust anchor for DNSSEC

	LOAD   = Reflects whether the unit definition was properly loaded.
	ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
	SUB    = The low-level unit activation state, values depend on unit type.

	3 loaded units listed. Pass --all to see loaded but inactive units, too.
	To show all installed unit files use 'systemctl list-unit-files'.
	[mickey@mickey system]$ systemctl status systemd-tmpfiles-clean.timer
	● systemd-tmpfiles-clean.timer - Daily Cleanup of Temporary Directories
	   Loaded: loaded (/usr/lib/systemd/system/systemd-tmpfiles-clean.timer; static; vendor preset: disabl>
	   Active: active (waiting) since Wed 2021-04-14 18:54:10 CST; 31min ago
	  Trigger: Thu 2021-04-15 19:09:10 CST; 23h left
		 Docs: man:tmpfiles.d(5)
			   man:systemd-tmpfiles(8)
	[mickey@mickey system]$ systemctl status systemd-tmpfiles-clean.service
	● systemd-tmpfiles-clean.service - Cleanup of Temporary Directories
	   Loaded: loaded (/usr/lib/systemd/system/systemd-tmpfiles-clean.service; static; vendor preset: disa>
	   Active: inactive (dead) since Wed 2021-04-14 19:09:10 CST; 17min ago
		 Docs: man:tmpfiles.d(5)
			   man:systemd-tmpfiles(8)
	  Process: 2620 ExecStart=/usr/bin/systemd-tmpfiles --clean (code=exited, status=0/SUCCESS)
	 Main PID: 2620 (code=exited, status=0/SUCCESS)
	```
- 從`systemctl status <服務名>.timer`，可得到timer的配置文件路徑
	```bash
	[Timer]
	# 開機後15min執行一次
	OnBootSec=15min
	# 每天執行一次
	OnUnitActiveSec=1d
	# 指定時間執行
	# OnCalendar=2019-07-01 12:32:10
	# 2019年7月 12:00到12:50，每10分鐘執行一次
	# OnCalendar=2019-07-* 12:00,10,20,30,40,50:00
	# 每10分鐘執行一次
	# OnCalendar=*:00/10
	```
- 修改後需要重新讀設定檔，`systemctl daemon-reload`
- 啟動服務，`systemctl enable --now <服務名>.timer`

### cron使用權限
- /etc/cron.allow存在，則cron.allow中的user可使用cron排程-->相當於白名單
- /etc/cron.allow不存在
	- /etc/cron.deny存在，默認，則cron.deny中的user不可使用cron排程-->相當於於黑名單
	- /etc/cron.deny不存在，只有root可用cron

## 🐧user cron
- `crontab -l`，查詢自己的user cron任務
	```bash
	[mickey@mickey system]$ crontab -l
	*/10 * * * *    echo "user cron tab : " $(date) >> /home/mickey/usercrontab.txt
	```
- `crontab -e`，新增/編輯自己的user cron任務
- `crontab -r`，刪除自己的user cron任務
	```bash
	[mickey@mickey system]$ crontab -l
	*/10 * * * *    echo "user cron tab : " $(date) >> /home/mickey/usercrontab.txt
	[mickey@mickey system]$ crontab -r
	[mickey@mickey system]$ crontab -l
	no crontab for mickey
	```
- `-u <用戶名>`，root才可查看/修改/刪除其他用戶的user cron
- 實際上user cron是存放在/var/spool/cron/<用戶名>中
	```bash
	[root@mickey ~]# cd /var/spool/cron/
	[root@mickey cron]# ll
	total 4
	-rw-------. 1 mickey student 81 Apr 14 22:20 mickey
	[root@mickey cron]# cat mickey
	*/10 * * * *    echo "user cron tab : " $(date) >> /home/mickey/usercrontab.txt
	```

# temp檔管理
temp檔管理主要是通過systemd-tmpfiles-clean.timer管理
- `systemctl cat systemd-tmpfiles-clean.timer`，查看systemd-tmpfiles-clean.timer服務文件內容
	```bash
	[root@mickey cron]# systemctl cat systemd-tmpfiles-clean.timer
	[Unit]
	Description=Daily Cleanup of Temporary Directories
	Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)

	[Timer]
	OnBootSec=15min
	OnUnitActiveSec=1d
	```
- `cat /usr/lib/systemd/system/systemd-tmpfiles-clean.service`
	```bash
	[root@mickey tmpfiles.d]# cat /usr/lib/systemd/system/systemd-tmpfiles-clean.service
	[Unit]
	Description=Cleanup of Temporary Directories
	Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
	DefaultDependencies=no
	Conflicts=shutdown.target
	After=local-fs.target time-sync.target
	Before=shutdown.target

	[Service]
	Type=oneshot
	# 執行清理的語句
	ExecStart=/usr/bin/systemd-tmpfiles --clean
	SuccessExitStatus=65
	IOSchedulingClass=idle
	```
- `/usr/bin/systemd-tmpfiles --help`，可查到`--clean`會清理被標記的內容
- `man 5 tmpfiles.d`說明文件，可查到定義標記的意思及設定檔放在：
	1. /etc/tmpfiles.d/\*.conf，優先級最高，若有相同名稱的.conf檔，則只會執行高優先級的.conf檔
	2. /run/tmpfiles.d/\*.conf，優先級中等
	3. /usr/lib/tmpfiles.d/\*.conf，優先級最低，系統預設設定檔，若要修改系統對temp檔的處理內容，不建議直接改此文件內容，建議在較高優先級的設定檔目錄加相同名稱的.conf檔
- 設定檔內容(范例)
	```bash
	# 看指定目錄存不存在，不存在則建立目錄
	d /home/mickey/Document/testTempfiles.d 0755 mickey student -

	# 指定目錄下的檔案超過1天沒動就刪
	D /home/mickey/Document 0700 mickey student 1d

	# 建立link至指定目錄
	L /home/mickey/Document/testLink - mickey student - /etc/tmpfiles.d
	```
- `systemd-tmpfiles --clean [設定檔路徑]`，立即執行指定設定檔