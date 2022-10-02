# Linux_RH134_05_SELinux
官方文件：[SELinux-Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/part_i-selinux)

## 🐧介紹
Security Enhanced Linux(安全性增強的Linux)，提高Linux系統的安全性，可防御未知攻擊，據稱相當於B1級[^信息安全評估標準]的軍事安全性能，已整合到2.6以上kernel中。

`uname -r`、`uname -a`(較不常用)，查詢kernel內核版本
```bash
[mickey@localhost ~]$ uname -r
4.18.0-147.el8.x86_64
[mickey@localhost ~]$ uname -a
Linux localhost.localdomain 4.18.0-147.el8.x86_64 #1 SMP Thu Sep 26 15:52:44 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

### 特點
- MAC(Mandatory Access Control)：對文件、目錄、端口的訪問徹底控制；這些策略是由管理員設置的，一般用戶無權更改。
- RBAC(Role Base Access Control)：只賦予用戶最小權限。對於用戶會被劃分成一些role(相當於是角色)，即使是root用戶只要不在sysdm_r中，也是不能執行sysdm_t管理操作。
- TE(Type Enforcement)：只賦予進程最小權限。對文件賦予**type的文件類型標簽**，對進程賦予**domain標簽**；規定某個進程只會執行某類文件。

### 運行機制
![SELinux](https://i1.kknews.cc/SIG=109c41k/43390001951q5pr95ors.jpg)
當一個進程要訪問一個資源時，會先取得資源的安全上下文，進到SELinux Server資料庫找此用戶是否有訪問權限，若權限符合才可以訪問。
SELinux資料庫路徑：/etc/selinux/targeted/policy/policy.31

## 🐧SELinux狀態及修改
### 查看SELinux狀態 sestatus
```bash
[root@localhost selinux]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31
```
1. SELinux status，SELinux目前狀態
2. SELinuxfs mount，SELinux相關檔案掛載點
3. SELinux root directory，根目錄
4. Loaded policy name，現在的政策
5. Current mode，現在的工作模式
6. Mode from config file，配置文件所指定的工作模式
7. Policy MLS status，是否含有 MLS 的模式機制
8. Policy deny_unknown status，是否預設抵擋未知的主體程序
9. Memory protection checking
10. Max kernel policy version

### 查看工作模式 getenforce
```bash
[root@localhost ~]# getenforce
Enforcing
```
- enforcing：強制模式，只要SELinux不允許就無法執行
- pemissive：警告模式，記錄不符合權限的事件，但依然允許執行
- disabled：關閉SELinux

### 修改工作模式
- 修改配置文件/etc/selinux/config後，重新開機才會生效，因為SELinux為kenel level的程序
	**disabled --> enforcing**
```properties
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.用於記憶體較小的機器，只載入特定服務的標籤環境
SELINUXTYPE=targeted
```
- `setenforce`，暫時修改SELinux的工作模式，電腦重新啟動還是會讀取/etc/selinux/config
	主要目的為selinux測試
	**enforcing --> permissive**
	```bash
	[root@localhost selinux]# setenforce 0
	[root@localhost selinux]# getenforce
	Permissive
	```
	**permissive --> enforcing**
	```bash
	[root@localhost selinux]# setenforce 1
	[root@localhost selinux]# getenforce
	Enforcing
	```

## 🐧安全上下文 Security Context
SELinux啟動後，所有文件與對象都有安全上下文(security context)
- `cp`會重新産生安全上下文，`mv`則不會改變

### 上下文構成
用戶:角色:類型:擴增項目
```bash
unconfined_u:object_r:user_home_t:s0 ifconfig.txt
```
- user，類似Linux中的UID
	1. user_u，普通用戶登錄的預設用戶，無特權用戶
	2. system_u，開機過程中系統進程的預設，系統用戶
	3. unconfined_u，非限制，沒有定義
- role，用戶角色
	1. 文件與目錄的角色，通常是object_r
	2. 用戶的角色，類似於系統中的GID，不同角色具備不同的權限；用戶可以具備多個角色，但同一時間只能使用一個角色
- type，用來將主體和客體劃分為不同的組，組中每個主體和系統中的客體定義了一個類型；為進程運行提供最低的權限環境

### 查看文檔上下文 ls -Z
- `ls -Z <文檔路徑>`，查看文檔安全上下文
	```bash
	[root@localhost Documents]# ls -Z ifconfig.txt
	unconfined_u:object_r:user_home_t:s0 ifconfig.txt
	```
- `id -Z`，查看用戶上下文
	```bash
	[root@mickey ~]# id -Z
	unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
	```
- `ps -auxZ`，查看進程上下文
	```bash
	[root@mickey ~]# ps -auxZ | head -n 5
	LABEL                           USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
	system_u:system_r:init_t:s0     root         1  0.0  0.3 179304 13712 ?        Ss   19:14   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	system_u:system_r:kernel_t:s0   root         2  0.0  0.0      0     0 ?        S    19:14   0:00 [kthreadd]
	system_u:system_r:kernel_t:s0   root         3  0.0  0.0      0     0 ?        I<   19:14   0:00 [rcu_gp]
	system_u:system_r:kernel_t:s0   root         4  0.0  0.0      0     0 ?        I<   19:14   0:00 [rcu_par_gp]
	```

## 🐧查詢SELinux資料庫 sesearch
可以使用setools-console提供的工具查看SELinux資料庫
- `sesearch -A -s httpd_t`，搜索type為httpd_t允許存取哪些type的檔案或目錄
	1. `-A`，allow
	2. `-s`，來源端標籤
- `sesearch -A -s httpd_t -t lib_t`
	1. `-t`，被允許的標籤(被允許的標籤 )

## 🐧查看SELinux log sealert
- 需先確認安裝setroubleshoot\*/套件
- 先在/var/log/messages查看SELinux log id
```bash
Mar 20 15:27:33 mickey setroubleshoot[7931]: SELinux is preventing httpd from getattr access on the file /custom/index.html. For complete SELinux messages run: sealert -l e9f5a8ec-b38b-41a0-a8e9-198b476706da
```
- log存放路徑：/var/log/audit/audit.log
- `sealert -a /var/log/audit/audit.log`，查看全部SELinux log
	或是`sealert -l <id>`，查看指定SELinux log
	裡面有參考指令，更改權限：`# /sbin/restorecon -v /var/www/html/shadow.html`
	```bash
	[root@mickey html]# sealert -a /var/log/audit/audit.log
	SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/shadow.html.

	*****  Plugin restorecon (99.5 confidence) suggests   ************************

	If you want to fix the label.
	/var/www/html/shadow.html default label should be httpd_sys_content_t.
	Then you can run restorecon. The access attempt may have been stopped due to insufficient permissions to access a parent directory in which case try to change the following command accordingly.
	Do
	# /sbin/restorecon -v /var/www/html/shadow.html

	*****  Plugin catchall (1.49 confidence) suggests   **************************

	If you believe that httpd should be allowed getattr access on the shadow.html file by default.
	Then you should report this as a bug.
	You can generate a local policy module to allow this access.
	Do
	allow this access for now by executing:
	# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
	# semodule -X 300 -i my-httpd.pp


	Additional Information:
	Source Context                system_u:system_r:httpd_t:s0
	Target Context                system_u:object_r:shadow_t:s0
	Target Objects                /var/www/html/shadow.html [ file ]
	Source                        httpd
	Source Path                   /usr/sbin/httpd
	Port                          <Unknown>
	Host                          <Unknown>
	Source RPM Packages           httpd-2.4.37-30.module+el8.3.0+7001+0766b9e7.x86_6
								  4
	Target RPM Packages
	Policy RPM                    selinux-policy-3.14.3-20.el8.noarch
	Selinux Enabled               True
	Policy Type                   targeted
	Enforcing Mode                Enforcing
	Host Name                     mickey.try
	Platform                      Linux mickey.try 4.18.0-147.el8.x86_64 #1 SMP Thu
								  Sep 26 15:52:44 UTC 2019 x86_64 x86_64
	Alert Count                   8
	First Seen                    2021-04-21 19:34:28 CST
	Last Seen                     2021-04-21 19:39:19 CST
	Local ID                      c6ae5f06-2bd6-4c85-ba98-40390048025e

	Raw Audit Messages
	type=AVC msg=audit(1619005159.531:253): avc:  denied  { getattr } for  pid=3494 comm="httpd" path="/var/www/html/shadow.html" dev="dm-0" ino=8403109 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:shadow_t:s0 tclass=file permissive=0


	type=SYSCALL msg=audit(1619005159.531:253): arch=x86_64 syscall=lstat success=no exit=EACCES a0=7f7b7c006e00 a1=7f7b93e948d0 a2=7f7b93e948d0 a3=1 items=0 ppid=3478 pid=3494 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=lstat AUID=unset UID=apache GID=apache EUID=apache SUID=apache FSUID=apache EGID=apache SGID=apache FSGID=apache

	Hash: httpd,httpd_t,shadow_t,file,getattr
	```

## 🐧SELinux策略
策略(policy，控制規則)，哪些進程可以訪問哪些資源。
說明：SELinux的設置一般通過安全上下文和策略完成，策略是安全上下文的補充。

### 限制進程對檔案存取
- 方法：
	1. 直接在SELinux資料庫撈相應目錄的標籤，再用`chcon`設置標籤
	2. 使用`restorecon`還原semanage中的標籤
	3. 使用`semanage fcontext`新增規則
		找相同性質的文件標籤拿來用即可

#### 更改文檔上下文 chcon
- `chcon <文件路徑>`，指定文件上下文
	1. `-u <用戶>`，指定用戶
	2. `-r <角色>`，指定角色
	3. `-t <類型>`，指定類型，般情況下需要專注的標籤
	4. `-R`，遞歸
	```bash
	[root@localhost Documents]# ls -Z ifconfig_1.txt
	unconfined_u:object_r:user_home_t:s0 ifconfig_1.txt
	[root@localhost Documents]# chcon -t var_t ifconfig_1.txt
	[root@localhost Documents]# ls -Z ifconfig_1.txt
	unconfined_u:object_r:var_t:s0 ifconfig_1.txt
	```
- `chcon --reference=<被複製的文件路徑> <貼上的文件路徑>`，複製修改上下文
	1. `-R`，遞歸
	```bash
	[root@localhost Documents]# ls -Z
		  unconfined_u:object_r:var_t:s0 ifconfig_1.txt
	unconfined_u:object_r:user_home_t:s0 ifconfig_2.txt
	[root@localhost Documents]# chcon --reference=ifconfig_1.txt ifconfig_2.txt
	[root@localhost Documents]# ls -Z
		  unconfined_u:object_r:var_t:s0 ifconfig_1.txt
		  unconfined_u:object_r:var_t:s0 ifconfig_2.txt
	[root@localhost Documents]#
	```

#### 恢複默認文檔上下文 restorecon
`restorecon`，恢複默認的安全上下文(Red Hat提供的工具)，前提此文檔為Red Hat系統內建的文檔/目錄(semanage有記錄)，否則為default_t(什麼也做不了)
- `-r`或`-R`，遞歸
- `-v`，顯示執行過程
設置文件的默認安全上下文放至：/etc/selinux/targeted/contexts/files/file_contexts
```bash
[root@localhost Documents]# ls -Z ifconfig_1.txt
      unconfined_u:object_r:var_t:s0 ifconfig_1.txt
[root@localhost Documents]# restorecon -v ifconfig_1.txt
Relabeled /home/mickey/Documents/ifconfig_1.txt from unconfined_u:object_r:var_t:s0 to unconfined_u:object_r:user_home_t:s0
[root@localhost Documents]# ls -Z ifconfig_1.txt
unconfined_u:object_r:user_home_t:s0 ifconfig_1.txt
```

#### 查看上下文規則
參考：[semanage命令 -Linux命令大全](https://man.linuxde.net/semanage)
- `semanage fcontext -l`，查詢，fcontext上下文資料庫-->主要用在安全上下文
```bash
[root@localhost selinux]# semanage fcontext -l| grep /etc/httpd| head -n 3
/etc/httpd(/.*)?                                   all files          system_u:object_r:httpd_config_t:s0
/etc/httpd/alias(/.*)?                             all files          system_u:object_r:cert_t:s0
/etc/httpd/alias/ipasession.key                    regular file       system_u:object_r:ipa_cert_t:s0
```

#### 修改上下文規則
- `semanage fcontext -a -f"<文檔類型>" -t <類型> '<路徑表達式>'`
	1. `-a`，新增
	2. `-f"<文檔類型>"`，'a', 'f', 'd', 'c', 'b', 's', 'l', 'p'，可參考：[[🐧Linux_RH124_08_文檔權限管理]]`ll`的file type
	```bash
	[root@mickey yum.repos.d]# semanage fcontext -a -f"f" -t httpd_sys_content_t '/home/mickey(/*)?'
	[root@mickey yum.repos.d]# semanage fcontext -l | grep 'mickey'
	/home/mickey(/*)?                                  regular file       system_u:object_r:httpd_sys_content_     t:s0
	```
- `semanage fcontext -m -f"<文檔類型>" -t <類型> '<路徑表達式>'`
	1. `-m`，修改
- `semanage fcontext -d -f"<文檔類型>" -t <類型> '<路徑表達式>'`
	1. `-d`，刪除

### 限制應用程式特定功能的啟用
#### 查看應用程式策略
- `semanage boolean -l`，查看所有策略
	![Linux_RH134_05_SELinux_01_應用程式權限](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_05_SELinux_01_%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E6%AC%8A%E9%99%90.png?raw=true)
- `getsebool -a`
	```bash
	[root@mickey ~]# getsebool -a | grep httpd | head -n 5
	httpd_anon_write --> off
	httpd_builtin_scripting --> on
	httpd_can_check_spam --> off
	httpd_can_connect_ftp --> off
	httpd_can_connect_ldap --> off
	```

#### 修改策略
`setsebool -P <標籤>=<on|off> [<value>=<on|off> ...]`，可一次設置多個
- `-P`，改記憶體、資料庫內容
```bash
[root@mickey html]# semanage boolean -l | grep httpd_anon_write
httpd_anon_write               (off  ,  off)  Allow httpd to anon write
[root@mickey html]# setsebool -P httpd_anon_write on
[root@mickey html]# semanage boolean -l | grep httpd_anon_write
httpd_anon_write               (on   ,   on)  Allow httpd to anon write
```

### 限制應用程式存取的端口號
#### 查看端口策略
`semanage port -l`
```bash
[root@mickey html]# semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
```

#### 修改策略
- `semanage port -a -t <標籤> -p tcp <端口號>`，新增port綁定
	```bash
	[root@mickey conf]# semanage port -a -t http_port_t -p tcp 5678
	[root@mickey conf]# semanage port -l | grep http_port_t
	http_port_t                    tcp      5678, 80, 81, 443, 488, 8008, 8009, 8443, 9000
	pegasus_http_port_t            tcp      5988
	```
- `-d`，刪除

[^信息安全評估標準]:信息安全評估標準(低-->高)：C1, C2, B1, B2, B3, A