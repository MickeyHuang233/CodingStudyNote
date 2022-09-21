# 文檔權限管理
## 🐧文檔權限角色
- 文檔所有者(user owner)
- 文檔所在組(group owner)
- 其它用戶(others)，除所有者和所在組外的用戶

## 🐧文檔權限介紹
![[Linux_RH124_09_文檔權限管理_01_文件和目錄的權限介紹.png]]
- 文件類型還有兩種比較少見：s，Unix Domain Soket；p，pipeline
- 若用戶同時符合此多個角色，權限：所有者 > 所在組 > 其它

### 權限說明
| 權限        | 文件                                      | 目錄                                                                         |
| ----------- | ----------------------------------------- | ---------------------------------------------------------------------------- |
| r(read)     | 讀取、查看文件內容權限                    | 讀取、查看目錄內容權限，如：`ls`                                             |
| w(write)    | 修改權限，但不代表可以刪除該文件          | 修改，目錄內創建、刪除、重命名目錄權限                                       |
| x(execute)  | 被執行權限                                | 進入該目錄權限，如：`cd`                                                     |
| u+s(suid)   | 所有用戶在執行時都是以user owner權限執行  | 無效果                                                                       |
| g+s(gid)    | 所有用戶在執行時都是以group owner權限執行 | 1. 建立文件、目錄會繼承上層目錄的group owner<br/>2. 建立目錄還會繼承sgid權限 |
| o+t(sticky) | 無效果                                    | 受sticky保護的目錄，裡面的文檔只有root和user owner可刪除                     |

## 🐧更改權限
- u，所有者；g，所有組；o，其他人；a，所有人
- user owner權限只能root更改
- 用戶次要群組也適用於group owner權限

### 更改文檔一般權限 chmod
- `chmod u+w,g+r,o+r <文件或目錄路徑>`，增加權限
- `chmod u-r,g-w,o-x <文件或目錄路徑>`，除去權限
- `chmod u=rwx,g=rx,o=x <文件或目錄路徑>`，指定權限
- `chmod 744 <文件或目錄路徑>`，指定權限(數字代表權限)
	![[Linux_RH124_09_文檔權限管理_02_一般權限計算.png]]
	```bash
	[mickey@localhost Documents]$ ll
	total 12
	-rw-r--r--. 1 mickey student 1422 Mar 18 09:50 hosts
	[mickey@localhost Documents]$ chmod 754 hosts
	[mickey@localhost Documents]$ ll
	total 12
	-rwxr-xr--. 1 mickey student 1422 Mar 18 09:50 hosts
	```
- `chmod -R <權限修改> <目錄路徑>`，遞歸
	`-X`，代表x(執行)的權限只會更動到目錄，忽略文件
	```bash
	[root@localhost mickey]# ll
	total 64
	drw-r-xr-x. 2 mickey student     6 Mar  7 20:13 Desktop
	drw-r-xr-x. 2 mickey student   121 Mar 18 09:50 Documents
	drw-r-xr-x. 2 mickey student     6 Mar  7 20:13 Downloads
	-rw-rw-r--. 1 mickey student     0 Mar 14 11:18 error.txt
	-rw-rw-r--. 1 mickey student   119 Mar 14 11:28 file_1
	-rw-rw-r--. 1 mickey student     0 Mar 14 11:28 file_2
	[root@localhost mickey]# chmod -R u+X .
	[root@localhost mickey]# ll
	total 64
	drwxr-xr-x. 2 mickey student     6 Mar  7 20:13 Desktop
	drwxr-xr-x. 2 mickey student   121 Mar 18 09:50 Documents
	drwxr-xr-x. 2 mickey student     6 Mar  7 20:13 Downloads
	-rw-rw-r--. 1 mickey student     0 Mar 14 11:18 error.txt
	-rw-rw-r--. 1 mickey student   119 Mar 14 11:28 file_1
	-rw-rw-r--. 1 mickey student     0 Mar 14 11:28 file_2
	```

### 更改文檔特殊權限 chmod
- `chmod u+s,g+s,o+t <文件或目錄路徑>`，增加權限
- `chmod u-s,g-s,o-t <文件或目錄路徑>`，刪除權限
- `chmod u=srwx,g=srwx,o=trwx <文件或目錄路徑>`，指定權限
	```bash
	[mickey@localhost Documents]$ chmod u=srwx,g=srwx,o=trwx hosts
	[mickey@localhost Documents]$ ll hosts
	-rwsrwsrwt. 1 mickey student 1422 Mar 18 09:50 hosts
	```
- `chmod 5744 <文件或目錄路徑>`，指定權限(數字代表權限)
	![[Linux_RH124_09_文檔權限管理_03_特殊權限計算.png]]
	```bash
	[mickey@localhost Documents]$ ll hosts
	-rwsrwsrwt. 1 mickey student 1422 Mar 18 09:50 hosts
	[mickey@localhost Documents]$ chmod 5754 hosts
	[mickey@localhost Documents]$ ll hosts
	-rwsr-xr-T. 1 mickey student 1422 Mar 18 09:50 hosts
	```
- 特殊權限查看：因為只有三個字元顯示角色的文件權限，所以特殊權限和x(execute)欄位共用，**有特殊權限時，大寫代表無x(execute)權限，小寫代表有x(execute)權限**，以下以suid為例：

	||rwx|rw-|
	|---|---|---|
	|u-s(無suid權限)|rwx|rw-|
	|u+s(有suid權限)|rw==s==|rw==S==|

### 更改所有者 chown
`chown <用戶名> <文件/目錄路徑>`，修改所有者
`chown -R <用戶名> <文件/目錄路徑>`，遞歸
`chown <用戶名>:<組名> <文件/目錄路徑>`，修改所有者及所有組
```bash
[root@localhost 下載]# ls -ahl
總計 32K
drwxr-xr-x.  3 root root  135  1月 12 10:44 .
dr-xr-x---. 15 root root 4.0K  1月  5 15:33 ..
-rw-r--r--.  1 root root  283  1月 12 10:38 testTest
[root@localhost 下載]# chown mickey testTest
[root@localhost 下載]# ls -ahl
總計 32K
drwxr-xr-x.  3 root   root  135  1月 12 10:44 .
dr-xr-x---. 15 root   root 4.0K  1月  5 15:33 ..
-rw-r--r--.  1 mickey root  283  1月 12 10:38 testTest
```

### 更改所有組 chgrp
當文件/目錄被創建後，這個文件/目錄的所在組就是該用戶所在的組
`chgrp <組名> <文件名>`，修改所有組
`chgrp -R <組名> <文件名>`，遞歸
```bash
[root@localhost 下載]# ls -ahl
總計 32K
drwxr-xr-x.  3 root   root  135  1月 12 10:44 .
dr-xr-x---. 15 root   root 4.0K  1月  5 15:33 ..
drwxr-xr-x.  2 mickey root   22  1月 12 10:06 testCreate
-rw-r--r--.  1 mickey root  283  1月 12 10:38 testTest
-rw-r--r--.  1 root   root  263  1月  5 12:48 testViEditor
-rw-r--r--.  1 root   root  12K  1月  5 12:49 .testViEditor.swp
[root@localhost 下載]# chgrp testTeam testTest
[root@localhost 下載]# ls -alh
總計 32K
drwxr-xr-x.  3 root   root      135  1月 12 10:44 .
dr-xr-x---. 15 root   root     4.0K  1月  5 15:33 ..
drwxr-xr-x.  2 mickey root       22  1月 12 10:06 testCreate
-rw-r--r--.  1 mickey testTeam  283  1月 12 10:38 testTest
-rw-r--r--.  1 root   root      263  1月  5 12:48 testViEditor
-rw-r--r--.  1 root   root      12K  1月  5 12:49 .testViEditor.swp
```

## 🐧權限遮罩 umask
- umask是一個確定掩碼設置的命令，該掩碼控制如何預設檔案、目錄權限。
- umask計算方式：
	![[Linux_RH124_09_文檔權限管理_04_umask計算.png]]

### 查看
`umask`
```bash
[mickey@localhost Documents]$ umask
0022
```

### 修改
- `umask <指定權限(數字)>`，修改umask值，作用域只在此Shell(登出後失效)
	```bash
	[mickey@localhost Documents]$ umask 542
	[mickey@localhost Documents]$ umask
	0542
	```
- 修改全局配置文件/etc/bashrc，不建議直接改(怕改錯)
	```bash
	[root@localhost ~]# cat -n /etc/bashrc| grep umask
		70      # By default, we want umask to get set. This sets it for non-login shell.
		75         umask 002
		77         umask 022
	```
- 修改單一用戶umask，可改`~/.bashrc`
- 新增文件/etc/profile.d/local-umask.sh
	```bash
	# Overrides default umask configuration
	umask 027
	```