# Linux_RH124_10_服務管理
- 服務(service)本質就是進程，但是運行在後台的，通常都會監聽某個端口，等待其它程序的請求，如：mysql、sshd、防火墻…等。
- 傳統的Linux都是靠System V的init來啟動各項系統服務，而現行主流的Linux發行版都用systemd來管理系統服務，主要systemd有相依性檢查，會將有相關性的服務一并開啟，所以systemd管理下開機速度更快、效能更好。
- 所有被systemd管理的服務被稱為Service Unit
- systemd的PID一定為1
	```bash
	[mickey@localhost system]$ ps -aux|grep systemd
	root         1  0.0  0.3 179248 13720 ?        Ss   07:07   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	```
- 套件安裝時會在/lib/systemd/system中建立相應服務的文件
	```bash
	[mickey@localhost system]$ ll /lib/systemd/system | head -10
	total 1516
	-rw-r--r--. 1 root root  729 Jun 18  2019 accounts-daemon.service
	-rw-r--r--. 1 root root  502 May 27  2019 alsa-restore.service
	-rw-r--r--. 1 root root  465 May 27  2019 alsa-state.service
	-rw-r--r--. 1 root root  722 Nov  8  2018 anaconda-direct.service
	-rw-r--r--. 1 root root  240 Nov  8  2018 anaconda-nm-config.service
	-rw-r--r--. 1 root root  660 Nov  8  2018 anaconda-noshell.service
	-rw-r--r--. 1 root root  585 Nov  8  2018 anaconda-pre.service
	-rw-r--r--. 1 root root  442 Nov  8  2018 anaconda.service
	-rw-r--r--. 1 root root  532 Nov  8  2018 anaconda-shell@.service
	```
	1. \*.service，通常為stay a long service(長駐服務)
	2. \*.socket，臨時服務，如：systemd監聽port是否有異動，有則呼叫服務
	3. \*.path，systemd監聽目錄是否有異動，有則呼叫服務

## 🐧查看服務 systemctl
使用`systemctl`對systemd控制和操作，參考：[systemctl命令](https://wiki.archlinux.org/index.php/Systemd_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87))

### 查看服務列表
- `systemctl list-units --type=<service|socket|path>`，查看所有指定副檔名的已啟動服務
	`systemctl`，列出所有已啟動的服務
	`list-units`，`systemctl`默認就是列出所有服務，可省略
	`--type=<servic|socket|path>`，篩選指定副檔名
	`--all`，啟動失敗的服務也一并列出
	```bash
	[mickey@localhost system]$ systemctl list-units --type=service|head -10
	  UNIT                               LOAD   ACTIVE SUB     DESCRIPTION
	  accounts-daemon.service            loaded active running Accounts Service
	  alsa-state.service                 loaded active running Manage Sound Card State (restore and store)
	  atd.service                        loaded active running Job spooling tools
	  auditd.service                     loaded active running Security Auditing Service
	  avahi-daemon.service               loaded active running Avahi mDNS/DNS-SD Stack
	  chronyd.service                    loaded active running NTP client/server
	  colord.service                     loaded active running Manage, Install and Generate Color Profiles
	  crond.service                      loaded active running Command Scheduler
	  cups.service                       loaded active running CUPS Scheduler
	```
	1. UNIT，service unit檔名
	2. LOAD，是否被systemd讀進記憶體中
	3. ACTIVE，服務正常啟動中
	4. SUB，running-->正在執行，exited-->服務已結束
	5. DESCRIPTION，服務描述
- `systemctl list-unit-files`，查看已安裝service unit
	```bash
	[mickey@localhost system]$ systemctl list-unit-files --type=service|head -10
	UNIT FILE                                  STATE
	accounts-daemon.service                    enabled
	alsa-restore.service                       static
	alsa-state.service                         static
	anaconda-direct.service                    static
	anaconda-nm-config.service                 static
	anaconda-noshell.service                   static
	anaconda-pre.service                       static
	anaconda-shell@.service                    static
	debug-shell.service                        disabled
	```
	1. UNIT FILE，service unit檔名
	2. STATE，service unit狀態
		1. enabled，電腦啟動時此服務會自動開啟
		2. disabled，enabled相反
		3. static，不會自動啟動，也無法手動啟動，通常為其他程式的相依服務
		4. masked，沖突性服務，設定則無法開啟

### 查看指定服務狀態
- `systemctl status <UNIT>`，查看指定服務狀態，.service可省略
	1. vendor preset: enabled，表示工廠初始值
	```bash
	[mickey@localhost system]$ systemctl status sshd.service
	● sshd.service - OpenSSH server daemon
	   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
	   Active: active (running) since Thu 2021-03-25 07:07:21 EDT; 2h 45min ago
		 Docs: man:sshd(8)
			   man:sshd_config(5)
	 Main PID: 1119 (sshd)
		Tasks: 1 (limit: 23977)
	   Memory: 5.5M
	   CGroup: /system.slice/sshd.service
			   └─1119 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,a>

	Mar 25 07:07:21 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
	Mar 25 07:07:21 localhost.localdomain sshd[1119]: Server listening on 0.0.0.0 port 22.
	Mar 25 07:07:21 localhost.localdomain sshd[1119]: Server listening on :: port 22.
	Mar 25 07:07:21 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
	Mar 25 07:08:55 localhost.localdomain sshd[2361]: Accepted password for mickey from 192.168.56.1 port 2445 ssh2
	Mar 25 07:08:55 localhost.localdomain sshd[2361]: pam_unix(sshd:session): session opened for user mickey by (uid=0)
	```
- `systemctl is-active <UNIT>`，查看指定服務是否被啟動，.service可省略
	```bash
	[mickey@localhost system]$ systemctl is-active sshd.service
	active
	```
- `systemctl is-enabled <UNIT>`，查看指定服務開機是否會自動啟動，.service可省略
	```bash
	[mickey@localhost system]$ systemctl is-enabled sshd.service
	enabled
	```
- `systemctl is-failed <UNIT>`，查看指定服務啟動是否夫敗，.service可省略
	```bash
	[mickey@localhost system]$ systemctl is-failed sshd.service
	active
	```

### 重新載入systemd
- `systemctl daemon-reload`，重新載入systemd
	```bash
	[mickey@localhost system]$ systemctl daemon-reload
	==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
	Authentication is required to reload the systemd state.
	Authenticating as: mickey
	```

## 🐧操作服務 systemctl
### 基本操作
`systemctl <start|stop|restart|reload> <UNIT>`，.service可省略
- start，啟動
- stop，關閉
	```bash
	[mickey@localhost system]$ systemctl stop sshd.service
	==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
	Authentication is required to stop 'sshd.service'.
	Authenticating as: mickey
	```
- restart，重啟
- reload，重新載入配置
- mask，禁用，實際上是將服務建立soft link至/dev/null
	```bash
	[mickey@localhost system]$ systemctl mask sendmail.service
	Unit sendmail.service does not exist, proceeding anyway.
	==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
	Authentication is required to manage system service or unit files.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	Created symlink /etc/systemd/system/sendmail.service → /dev/null.
	==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
	Authentication is required to reload the systemd state.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	[mickey@localhost system]$ systemctl status sendmail
	● sendmail.service
	   Loaded: masked (Reason: Unit sendmail.service is masked.)
	   Active: inactive (dead)
	```
- unmask，取消禁用，實際上是刪除soft link
	```bash
	[mickey@localhost system]$ systemctl unmask sendmail
	==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
	Authentication is required to manage system service or unit files.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	Removed /etc/systemd/system/sendmail.service.
	==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
	Authentication is required to reload the systemd state.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	```

### 搜索相依服務
`systemctl list-dependencies <UNIT>`，.service可省略
```bash
[mickey@localhost system]$ systemctl list-dependencies sshd.service
sshd.service
● ├─system.slice
● ├─sshd-keygen.target
● │ ├─sshd-keygen@ecdsa.service
● │ ├─sshd-keygen@ed25519.service
● │ └─sshd-keygen@rsa.service
● └─sysinit.target
●   ├─dev-hugepages.mount
```

### 開機啟用/關閉服務
- `systemctl enable <UNIT>`，設置開機啟動服務
	當服務開機啟用設置enable時，系統會將/etc/systemd/system/服務名 建立soft link至/userr/lib/systemd/system/服務名
	```bash
	[mickey@localhost system]$ systemctl enable sshd.service
	==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
	Authentication is required to manage system service or unit files.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.
	==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
	Authentication is required to reload the systemd state.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	```
- `systemctl disable <UNIT>`，設置開機不啟動服務，.service可省略
	實際上是將soft link刪除
	```bash
	[mickey@localhost system]$ systemctl disable sshd.service
	==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
	Authentication is required to manage system service or unit files.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	Removed /etc/systemd/system/multi-user.target.wants/sshd.service.
	==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
	Authentication is required to reload the systemd state.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	```

# 其他
## telnet-windows使用
在windows cmd中可使用`telnet`檢查linux的某個端口是否在監聽
參考：[telnet 不是内部或外部命令怎么辦呢?](https://goodlucky.pixnet.net/blog/post/48400569-%5bwin10%5d-telnet-%e4%b8%8d%e6%98%af%e5%86%85%e9%83%a8%e6%88%96%e5%a4%96%e9%83%a8%e5%91%bd%e4%bb%a4%e6%80%8e%e4%b9%88%e8%be%a6%e5%91%a2%3f)
```bash
telnet 192.168.0.14 22
```

## setup
`setup`，查看服務名
![Linux_RH124_10_服務管理_02_setup](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_10_%E6%9C%8D%E5%8B%99%E7%AE%A1%E7%90%86_02_setup.png?raw=true)
