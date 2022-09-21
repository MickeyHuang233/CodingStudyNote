# Log查看
## 🐧簡介
![[Linux_RH124_15_Log查看_01_舊架構底下syslog結構.png]]
1. 以前Red Hat的kernal的log是交給klogd管理，而klogd會交由syslogd處理，而其他的應用程序的log則直接交由syslogd處理
2. syslogd會查看設定檔/etc/syslog.conf，將log分類後存至/var/log/中

![[Linux_RH124_15_Log查看_02_當前架構底下syslog結構.png]]
1. Red Hat架構則全交由system-journald負責，而system-journald除了自己留一份log外還交給rsyslog打印log
2. 下一個版本可能就會把rsyslog刪掉了

## 🐧log産生器
`logger -p <類型>.<log記錄等級> <log內容>`
```bash
[root@localhost log]# logger -p mail.alert "test logger print"
[root@localhost log]# cat /var/log/maillog
Apr  1 22:33:43 localhost mickey[4323]: test logger print
```

## 🐧rsyslog
rsyslog的log檔就是文字檔，直接看就可以了

### 設定檔
- rsyslog設定檔路徑：(兩者可則一使用)
	1. rsyslog.conf
	2. /etc/rsyslog.d/\<任意檔名\>.conf
	3. 注意：修改設定檔後，需要用systemctl重啟服務`systemctl restart rsyslog`
- 內容說明
	```
	類型.log記錄等級	處理方式
	
	authpriv.*                                              /var/log/secure
	mail.*                                                  -/var/log/maillog
	uucp,news.crit                                          /var/log/spooler
	```
	1. 類型：類型是寫死在程式中的，需要查看說明文件可得知此APP的log類型、記錄等級，\*.info代表所有的info log
	2. log記錄等級：mail.info代表比info等級高的log，mail.=info代表僅info等級，mail.=!info代表不為info等級，mail.\*代表全部等級，mail.none代表所有等級都不要
		- 0，emerg
		- 1，alert
		- 2，crit
		- 3，err
		- 4，warning 
		- 5，notice
		- 6，info
		- 7，debug
	3. 處理方式：log儲存路徑、給終端機視窗、另一Linux主機…等
- log打印內容
	![[Linux_RH124_15_Log查看_03_log內容.png]]

### logrotate 定期打包工具
路徑：/etc/logrotate.conf
打包格式：maillog-20210314
```bash
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
# 表示保留4個log檔
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may be also be configured here.
```

## 🐧journalctl
- journalctl管理systemd-journal，因為查詢的功能比較強大，所以一般都是用journalctl
- systemd-journal預設放在/run/log中(/run為內存)，若想保留log需要另外設置
- journalctl是查找log資料庫工具，直接看資料庫是二進制

### log查看
- `journalctl -n <行數>`，查看最後幾行log
	```bash
	[root@localhost log]# journalctl -n 5
	-- Logs begin at Thu 2021-04-01 21:05:37 EDT, end at Thu 2021-04-01 22:43:26 EDT. --
	Apr 01 22:43:26 localhost.localdomain NetworkManager[1109]: <info>  [1617331406.9728] device>
	Apr 01 22:43:26 localhost.localdomain NetworkManager[1109]: <info>  [1617331406.9735] dhcp4 >
	Apr 01 22:43:26 localhost.localdomain avahi-daemon[979]: Joining mDNS multicast group on int>
	Apr 01 22:43:26 localhost.localdomain avahi-daemon[979]: New relevant interface enp0s8.IPv6 >
	Apr 01 22:43:26 localhost.localdomain avahi-daemon[979]: Registering new address record for >
	```
- `journalctl -f`，相當於`tail -f`
- `journalctl -p <記錄層級>`，查看指定層級log
	debug、info、notice、warning、err、crit、alert、emerg
	```bash
	[root@localhost log]# journalctl -p err
	```
- `journalctl --since <YYYY-MM-DD> --until <YYYY-MM-DD>`，查看時間段
	```bash
	[root@localhost log]# journalctl --since 2020-04-01 --until 2020-04-05
	[root@localhost log]# journalctl --since yesterday --until today
	[root@localhost log]# journalctl --until today
	[root@localhost log]# journalctl --since "-1 hour"
	[root@localhost log]# journalctl --since "-1 min"
	```
- `journalctl -o verbose`，查看log詳細信息，(verbose：冗長的)
	```bash
	[root@localhost log]# journalctl -o verbose
	-- Logs begin at Thu 2021-04-01 21:05:37 EDT, end at Thu 2021-04-01 23:16:56 EDT. --
	Thu 2021-04-01 21:05:37.482525 EDT [s=b70645cc161743f688ae46ce3a247ae6;i=194;b=e740f1580aca4>
		_BOOT_ID=e740f1580aca4f89af7267c94026453a
		_MACHINE_ID=25584f8132cc49d4ac22b8d243d95c8c
		_HOSTNAME=localhost.localdomain
		PRIORITY=6
		SYSLOG_FACILITY=3
		SYSLOG_IDENTIFIER=systemd
		_UID=0
		_GID=0
		_SELINUX_CONTEXT=kernel
		_CAP_EFFECTIVE=3fffffffff
		CODE_FILE=../src/core/job.c
		CODE_LINE=827
		CODE_FUNC=job_log_status_message
		MESSAGE=Started dracut cmdline hook.
		JOB_TYPE=start
		JOB_RESULT=done
		UNIT=dracut-cmdline.service
		INVOCATION_ID=c15dcb4d42b543eab3d05563ee9f63f0
		MESSAGE_ID=39f53479d3a045ac8e11786248231fbf
		_TRANSPORT=journal
		_PID=1
		_COMM=systemd
		_EXE=/usr/lib/systemd/systemd
		_CMDLINE=/init rhgb
		_SYSTEMD_CGROUP=/init.scope
		_SYSTEMD_UNIT=init.scope
		_SYSTEMD_SLICE=-.slice
		_SOURCE_REALTIME_TIMESTAMP=1617325537482525
	```
- `journalctl <欄位>=<條件>`篩選欄位(可多筆)，欄位名在`journalctl -o verbose`中可看
	```bash
	[root@localhost log]# journalctl _PID=1
	```

### 設定檔
- 路徑：/etc/systemd/journald.conf
- 修改設定檔需要重啟：`systemctl restart system-journald`
- 設置log儲存策略，Storage
	其他設定可參考：[journald.conf -Ubuntu Manpage](http://manpages.ubuntu.com/manpages/bionic/zh_TW/man5/journald.conf.5.html)
	```bash
	# persistent，log存至/var/log/journal
	# volatile，log存至/run/log/journal
	# auto，log存至/journal
	# none，不存log
	Storage=auto
	```
- `journalctl | grep -E 'Runtime|System journal'`，查看log剩余空間
	```bash
	[root@localhost log]# journalctl | grep -E 'Runtime|System journal'
	Apr 01 21:05:37 localhost.localdomain systemd-journald[183]: Runtime journal (/run/log/journal/25584f8132cc49d4ac22b8d243d95c8c) is 8.0M, max 189.0M, 181.0M free.
	Apr 01 21:05:42 localhost.localdomain systemd-journald[695]: Runtime journal (/run/log/journal/25584f8132cc49d4ac22b8d243d95c8c) is 8.0M, max 189.0M, 181.0M free.
	Apr 01 21:05:42 localhost.localdomain systemd-journald[695]: Runtime journal (/run/log/journal/25584f8132cc49d4ac22b8d243d95c8c) is 8.0M, max 189.0M, 181.0M free.
	Apr 01 21:05:48 localhost.localdomain systemd[1]: Starting Tell Plymouth To Write Out Runtime Data...
	Apr 01 21:05:48 localhost.localdomain systemd[1]: Started Tell Plymouth To Write Out Runtime Data.
	```


