# 套件管理
注意：只有root用戶才可以安裝

## 🐧本地安裝 rpm
- 過去在Linux安裝套件需要自行編釋，有時需要自行debug，導致Linux的入門門檻很高，因此Red Hat開發Red Hat Package Manager
- rpm雖然解決套件的編譯問題，但不能解決套件的相依性問題
- rpm安裝檔命名規則：<套件名稱>-<版本號>-<發行序號>.<cpu結構>.rpm
	- cpu結構：現在cpu只有64位元版本
		1. 1686，32位元
		2. x86_64，64位元
		3. aarch64，手機cpu，如：樹莓派
		4. noarch，套件和cpu無關
- rpm安裝包組成部分
	1. 安裝包安裝的文件，`rpm -ql <套件名>`
	2. 安裝包的相關信息(metadata)，如套件名、版本號…等，`rpm -qi <套件名>`
	3. 用於套件初始化(安裝前、安裝後、卸載前、卸載後)的程式碼，`rpm -q --scripts <套件名>`
- 注意：除了kernel外的套件，不同版本不能同時裝在本機
- 參考：[rpm命令](https://man.linuxde.net/rpm)

### 查看套件信息 rpm -q
rpm的查詢功能只有在套件安裝後才可查詢詳細信息，-q-->query
- `rpm -q <套件名>`，查看套件有沒有安裝
	```bash
	[mickey@mickey ~]$ rpm -q zip
	zip-3.0-23.el8.x86_64
	[mickey@mickey ~]$ rpm -q traceroute
	package traceroute is not installed
	```
- `rpm -qi <套件名>`查看套件相關信息，較不可信，因為在rpm打包的時候可以自訂這些信息
	```bash
	[mickey@mickey ~]$ rpm -qi zip
	Name        : zip
	Version     : 3.0
	Release     : 23.el8
	Architecture: x86_64
	Install Date: Mon 08 Mar 2021 08:59:35 AM CST
	Group       : Applications/Archiving
	Size        : 842877
	License     : BSD
	Signature   : RSA/SHA256, Sat 15 Dec 2018 09:33:01 AM CST, Key ID 199e2f91fd431d51
	Source RPM  : zip-3.0-23.el8.src.rpm
	Build Date  : Tue 20 Nov 2018 08:32:18 PM CST
	Build Host  : x86-vm-02.build.eng.bos.redhat.com
	Relocations : (not relocatable)
	Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
	Vendor      : Red Hat, Inc.
	URL         : http://www.info-zip.org/Zip.html
	Summary     : A file compression and packaging utility compatible with PKZIP
	Description :
	The zip program is a compression and file packaging utility.  Zip is
	analogous to a combination of the UNIX tar and compress commands and
	is compatible with PKZIP (a compression and file packaging utility for
	MS-DOS systems).

	Install the zip package if you need to compress files using the zip
	program.
	```
	確認rpm安裝包可靠性：
	1. 官網下載安裝包金鑰
	2. 匯入rpm，`rpm -import <金鑰路徑>`
	3. 檢查，`rpm -k <rpm安裝包路徑>`
- `rpm -qa`，查看所有已安裝套件
	```bash
	[mickey@mickey ~]$ rpm -qa | head -n 5
	libblockdev-lvm-2.19-11.el8.x86_64
	exempi-2.4.5-2.el8.x86_64
	rhn-check-2.8.16-13.module+el8.1.0+3455+3ddf2832.x86_64
	trousers-lib-0.3.14-4.el8.x86_64
	google-noto-sans-sinhala-fonts-20161022-7.el8.noarch
	```
- `rpm -ql <套件名>`，查看安裝包所有的安裝文件路徑
	```bash
	[mickey@mickey ~]$ rpm -ql zip | head -n 5
	/usr/bin/zip
	/usr/bin/zipcloak
	/usr/bin/zipnote
	/usr/bin/zipsplit
	/usr/lib/.build-id
	```
- `rpm -qf <文件|目錄路徑>`，查看檔案是屬於哪個套件
	```bash
	[mickey@mickey ~]$ rpm -qf /usr/bin/zipcloak
	zip-3.0-23.el8.x86_64
	```
- `rpm -qc <套件名>`，查看設定檔路徑
	```bash
	[mickey@mickey ~]$ rpm -qc yum
	/etc/dnf/protected.d/yum.conf
	```
- `rpm -qd <套件名>`，查看說明文件路徑
	```bash
	[mickey@mickey ~]$ rpm -qd zip | head -n 5
	/usr/share/doc/zip/CHANGES
	/usr/share/doc/zip/README
	/usr/share/doc/zip/README.CR
	/usr/share/doc/zip/TODO
	/usr/share/doc/zip/WHATSNEW
	```
- `rpm -q --scripts <套件名>`，查看套件初始化(安裝前、安裝後、卸載前、卸載後)的程式碼
	```bash
	[mickey@mickey ~]$ rpm -q -scripts openssh-server
	preinstall scriptlet (using /bin/sh):
	getent group sshd >/dev/null || groupadd -g 74 -r sshd || :
	getent passwd sshd >/dev/null || \
	  useradd -c "Privilege-separated SSH" -u 74 -g sshd \
	  -s /sbin/nologin -r -d /var/empty/sshd sshd 2> /dev/null || :
	postinstall scriptlet (using /bin/sh):

	if [ $1 -eq 1 ] ; then
			# Initial installation
			systemctl --no-reload preset sshd.service sshd.socket &>/dev/null || :
	fi
	preuninstall scriptlet (using /bin/sh):

	if [ $1 -eq 0 ] ; then
			# Package removal, not upgrade
			systemctl --no-reload disable --now sshd.service sshd.socket &>/dev/null || :
	fi
	postuninstall scriptlet (using /bin/sh):

	if [ $1 -ge 1 ] ; then
			# Package upgrade, not uninstall
			systemctl try-restart sshd.service &>/dev/null || :
	fi
	```
- `rpm -q --changelog <套件名>`，查看改版記錄
	```bash
	[mickey@mickey ~]$ rpm -q --changelog zip | head -n 5
	* Tue Nov 13 2018 Jakub Martisko <jamartis@redhat.com> - 3.0-23
	- Set the ziperr function as noreturn
	- Fix email in the previous chnagelog entry
	- Related: #1602741
	```

### 安裝rpm套件(不建議使用)
- `rpm -ivh <安裝包路徑>`
	1. `-i`，install，安裝
	2. `-v`，verbose，提示
	3. `-h`，hash，進度條

### 重置設定檔
設定檔不小心刪掉的做法，主要思路：解開套件-->覆製設定檔至相關路徑
1. `rpm2cpio <安裝包路徑> | cpio -id "*.txt"`，解開套件，cpio檔類似於tar
2. 覆製設定檔至相關路徑

## 🐧yum
- yum只處理相依性問題，底層還是用rpm
- 參考：[yum命令](https://man.linuxde.net/yum)

### 設置yum軟件倉庫
software repositories 
在使用yum安裝套件前需要先指向yum server，否則linux不知道去哪下載套件
![[Linux_RH124_17_套件管理_01_yum server.png]]
- [fedora yum server](https://dl.fedoraproject.org/pub/epel/)
- 設定檔路徑：/etc/yum.repos.d/\*.repo，可一個設定檔寫多個，也可寫多個設定檔
	```bash
	# 倉庫名稱
	[Base]
	# 作用類似於注解
	name=EPEL 8
	# yum server url，每個server放的路徑不一樣，主要看網址底下有/repodata就可以
	baseurl=https://downloads.redhat.com/redhat/rhel/rhel-8-beta/baseos/source/
	# 另外metalink表示備源yum server
	# metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-$releasever&arch=$basearch&infra=$infra&content=$contentdir
	# 是否啟用此倉庫，1-->enable，0-->disable
	enabled=1
	# 是否啟用數位簽章檢查，1-->yes，2-->no
	gpgcheck=1
	# 數位簽章路徑，若在本機可用file://開頭
	gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-beta,file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
	```
- `yum repolist all`，查看所有repository
	```bash
	[root@localhost yum.repos.d]# yum repolist all
	Updating Subscription Management repositories.
	Unable to read consumer identity
	This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
	Last metadata expiration check: 0:08:25 ago on Sat 03 Apr 2021 11:53:12 PM CST.
	Modular dependency problems:

	 Problem 1: conflicting requests
	  - nothing provides module(perl:5.26) needed by module perl-DBD-SQLite:1.58:8010020190322125518:073fa5fe-0.x86_64
	 Problem 2: conflicting requests
	  - nothing provides module(perl:5.26) needed by module perl-DBI:1.641:8010020190322130042:16b3ab4d-0.x86_64
	repo id                                             repo name                                          status
	Base                                                EPEL 8                                             enabled: 7,219
	```
- import repository setting & local gpgkey
	```bash
	[root@localhost yum.repos.d]# rpm --import \
	> https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8
	[root@localhost yum.repos.d]# yum install \
	> https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	```

### 查詢套件
- `yum list '<查詢條件(支援正則)>'`，查看套件列表
	<套件名稱>.<cpu結構>		<版本號>-<發行序號>		倉庫名稱(@表示已安裝)
	```bash
	[root@mickey ~]# yum list 'http*'
	Available Packages
	httping.x86_64                                2.5-8.el8                                  epel
	httping.x86_64                                2.5-8.el8                                  Base
	httpry.x86_64                                 0.1.8-1.el8                                @anaconda
	httpry.x86_64                                 0.1.8-1.el8                                @anaconda
	```
- `yum search all '<關鍵字>'`，查看套件說明是否出現指定關鍵字
	```bash
	[root@mickey ~]# yum search all 'web service'
	======================== Summary & Description Matched: web service =========================
	python3-cornice.noarch : Define Web Services in Pyramid
	cockpit-ws.x86_64 : Cockpit Web Service
	mono-web.x86_64 : ASP.NET, Remoting, and Web Services for Mono
	perl-AWS-Signature4.noarch : Create a version4 signature for Amazon Web Services
	=============================== Summary Matched: web service ================================
	python3-geopy.noarch : Python client for several popular geocoding web services
	============================= Description Matched: web service ==============================
	python2-geoip2.noarch : MaxMind GeoIP2 API
	```
- `yum info <套件名>`，查看套件相關信息
	```bash
	[root@mickey ~]# yum info zip
	Installed Packages
	Name         : zip
	Version      : 3.0
	Release      : 23.el8
	Architecture : x86_64
	Size         : 823 k
	Source       : zip-3.0-23.el8.src.rpm
	Repository   : @System
	From repo    : anaconda
	Summary      : A file compression and packaging utility compatible with PKZIP
	URL          : http://www.info-zip.org/Zip.html
	License      : BSD
	Description  : The zip program is a compression and file packaging utility.  Zip is
				 : analogous to a combination of the UNIX tar and compress commands and
				 : is compatible with PKZIP (a compression and file packaging utility for
				 : MS-DOS systems).
				 :
				 : Install the zip package if you need to compress files using the zip
				 : program.
	```
- `yum provides <文件|目錄路徑>`，查看檔案是屬於哪個套件
	```bash
	[root@mickey ~]# yum provides /usr/bin/zipcloak
	zip-3.0-23.el8.x86_64 : A file compression and packaging utility compatible with PKZIP
	Repo        : @System
	Matched from:
	Filename    : /usr/bin/zipcloak
	```

### 套件操作
#### 安裝套件
`yum install <套件名稱>`
yum下載的安裝包會放在/var/cache/yum中
```bash
[root@mickey yum.repos.d]# yum install lighttpd-filesystem.noarch
Dependencies resolved.
=============================================================================================
 Package                       Architecture     Version                 Repository      Size
=============================================================================================
Installing:
 lighttpd-filesystem           noarch           1.4.55-3.el8            epel            24 k

Transaction Summary
=============================================================================================
Install  1 Package

Total download size: 24 k
Installed size: 0
Is this ok [y/N]:
```

#### 更新套件
`yum update [套件名稱]`，若無指定則更新全部已安裝套件
注意：若是更新kernel，則會直接安裝新版本的kernel(因為kernel對Linux太重要了)
```
[root@mickey cache]# yum update lighttpd-filesystem.noarch
Dependencies resolved.
Nothing to do.
Complete!
```

#### 移除套件
`yum remove <套件名稱>`
```bash
[root@mickey cache]# yum remove lighttpd-filesystem.noarch
Dependencies resolved.
=============================================================================================
 Package                       Architecture     Version                Repository       Size
=============================================================================================
Removing:
 lighttpd-filesystem           noarch           1.4.55-3.el8           @epel             0

Transaction Summary
=============================================================================================
Remove  1 Package

Freed space: 0
Is this ok [y/N]:
```

### 套件群組yum group
套件群組，類似懶人包
- `yum group list`，查看套件群組(有些不會顯示)
	```bash
	[root@mickey cache]# yum group list
	Available Environment Groups:
	   KDE Plasma Workspaces
	Available Groups:
	   Fedora Packager
	   Xfce
	```
- `yum group list hidden`，查看所有的套件群組
	```bash
	[root@mickey cache]# yum group list hidden
	Available Environment Groups:
	   KDE Plasma Workspaces
	Available Groups:
	   Critical Path (KDE)
	   Fedora Packager
	   Firefox Web Browser
	   KDE Applications
	   KDE
	   KDE Educational applications
	   KDE Multimedia support
	   KDE Office
	   KDE Software Development
	   KDE Frameworks 5 Software Development
	   Xfce
	```
- `yum group info "<套件群組名>"`，查看套件群組詳細信息
	```bash
	[root@mickey cache]# yum group info "KDE Office"
	Group: KDE Office
	 Description: KDE Office applications
	 Mandatory Packages:
	   calligra-sheets
	   calligra-stage
	   calligra-words
	   okular
	```
- `yum group install "<套件群組名>"`
- `yum group update "<套件群組名>"`
- `yum group remove "<套件群組名>"`

### 套件安裝記錄
yum所有安裝記錄存放路徑：/var/log/dnf.rpm.log
- `yum history`，查看yum安裝記錄
	```bash
	[root@mickey cache]# yum history
	ID     | Command line             | Date and time    | Action(s)      | Altered
	-------------------------------------------------------------------------------
		 4 | remove lighttpd-filesyst | 2021-04-05 00:32 | Removed        |    1
		 3 | install lighttpd-filesys | 2021-04-05 00:12 | Install        |    1
		 2 | install https://dl.fedor | 2021-04-04 00:05 | Install        |    1
		 1 |                          | 2021-03-08 08:58 | Install        | 1364 EE
	```
- `yum history undo <步驟編號>`，套件反向操作，如：不想更新
	```bash
	[root@mickey cache]# yum history
	ID     | Command line             | Date and time    | Action(s)      | Altered
	-------------------------------------------------------------------------------
		 4 | remove lighttpd-filesyst | 2021-04-05 00:32 | Removed        |    1
		 3 | install lighttpd-filesys | 2021-04-05 00:12 | Install        |    1
		 2 | install https://dl.fedor | 2021-04-04 00:05 | Install        |    1
		 1 |                          | 2021-03-08 08:58 | Install        | 1364 EE
	[root@mickey cache]# yum history undo 4
	Undoing transaction 4, from Mon 05 Apr 2021 12:32:50 AM CST
		Removed lighttpd-filesystem-1.4.55-3.el8.noarch @@System
	Dependencies resolved.
	=============================================================================================
	 Package                       Architecture     Version                 Repository      Size
	=============================================================================================
	Installing:
	 lighttpd-filesystem           noarch           1.4.55-3.el8            epel            24 k

	Transaction Summary
	=============================================================================================
	Install  1 Package
	(以下略)
	```
- `yum clean all`，清空yum安裝包、資料庫
	1. 安裝包：/var/cache/yum
	2. 資料庫：/var/cache/dnf
	```bash
	[root@mickey cache]# yum clean all
	Updating Subscription Management repositories.
	Unable to read consumer identity
	This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
	29 files removed
	```

## 🐧yum module
- 使用yum安裝套件時只會安裝固定版本的套件，若要安裝指定版本套件需要使用module
- Red Hat 8開始出現Package Module
	1. BaseOS，作業系統本身套件
	2. Application Stream，應用程式套件，每個套件視為一個Module，放於/mnt/AppStream

### 查看Module
- `yum module list [module名稱]`，若無指定則查看所有module
	1. \[d\]efault，默認安裝
	2. \[e\]nabled，可安裝
	3. \[x\]disabled，不可安裝
	4. \[i\]nstalled，已安裝
	```bash
	[root@mickey mnt]# yum module list perl*
	@modulefailsafe
	Name                   Stream           Profiles        Summary
	perl-DBD-SQLite        1.58 [e]         common          SQLite DBI driver
	perl-DBI               1.641 [e]        common          A database access API for Perl

	Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
	```
- `yum module info --profile <module名>:<module版本號>`，查看profile，不同profile會安裝不同的套件
	```bash
	[root@mickey mnt]# yum module info --profile perl-DBI:1.641
	Name   : perl-DBI:1.641:8010020190322130042:16b3ab4d:x86_64
	common : perl-DBI
	```

### Module操作
- `yum module install <module名>`，安裝默認module
- `yum module install <module名>:<版本號>/<profile名>`，安裝指定module
	```bash
	[root@mickey mnt]# yum module list perl*
	@modulefailsafe
	Name                   Stream           Profiles        Summary
	perl-DBD-SQLite        1.58 [e]         common          SQLite DBI driver
	perl-DBI               1.641 [e]        common          A database access API for Perl

	Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
	[root@mickey mnt]# yum module install perl-DBI:1.641/common
	```
- `yum module remove <module名>`，移除module
	```bash
	[root@mickey mnt]# yum module remove perl
	Problems in request:
	missing groups or modules: perl
	Dependencies resolved.
	Nothing to do.
	Complete!
	```
- `yum module disable <module名>`，enable-->disable
- `yum module enable <module名>`，disable-->enable
- `yum module reset <module名>`，重置module
	```bash
	[root@mickey mnt]# yum module reset perl-DBI
	Dependencies resolved.
	=============================================================================================
	 Package              Architecture        Version                 Repository            Size
	=============================================================================================
	Resetting modules:
	 perl-DBI

	Transaction Summary
	=============================================================================================

	Is this ok [y/N]: y
	Complete!
	```


## 🐧軟件包或套件
- bash-completion.noarch：按Tab指令參數補齊
- wireshark：分包分析工具(Linux內建)
- gnome-tweaks：換皮膚套件
- 文字檔轉換(主要是控制字元)：dos2unix、unix2dos
---
- postgresql，opensource仿oracle SQL Server




