# Linux_RH124_12_遠端傳送文件
- winscp2：Linux<-->windows文件傳輸工具
- 遠端路徑表示方式：`<遠端用戶>@<遠端IP|電腦名稱>:<遠端目錄路徑>`，如：
	1. `root@127.0.0.1:/home/mickey/Documents`
	2. `root@localhost:/root/Documents`

## 🐧scp
缺點：文件和目錄的屬性(UID、GID…等)會被刷成當前使用者的

### 本地-->遠端
`scp <本地文檔路徑(可多個)> <遠端用戶>@<遠端IP|電腦名稱>:<遠端目錄路徑>`，SSH加密傳送檔案至遠端
- `-r`，遞歸傳送目錄
```bash
[mickey@localhost testDir]$ scp empty1 empty2 root@localhost:/home
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:QsLxggmRYrtugyyvFE7b0m1fnXI0REVj/WMJPZxQrs8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
root@localhost's password:
empty1                                                                      100%    0     0.0KB/s   00:00
empty2                                                                      100%    0     0.0KB/s   00:00
[mickey@localhost testDir]$ ll /home
total 4
-rw-r-----.  1 root    root       0 Mar 23 08:35 empty1
-rw-r-----.  1 root    root       0 Mar 23 08:35 empty2
```

### 遠端-->本地
`scp <遠端用戶>@<遠端IP|電腦名稱>:<遠端文件路徑> <本地目錄路徑>`
- `-r`，遞歸傳送目錄
```bash
[root@localhost ~]# scp mickey@localhost:/home/mickey/Documents/testDir/empty2 /root
mickey@localhost's password:
empty2                                                                      100%    0     0.0KB/s   00:00
[root@localhost ~]# ll |grep empty
-rw-r-----. 1 root root     0 Mar 23 08:46 empty2
```


## 🐧ftp
[linux ftp連線](https://crmne0707.pixnet.net/blog/post/322655646-linux-ftp%e9%80%a3%e7%b7%9a)

## 🐧sftp
sftp只能抓取已知路徑的文件
`sftp <遠端用戶>@<遠端IP|電腦名稱>`，sftp至指定IP

### 本地-->遠端
`put <文件路徑>`
```bash
[mickey@localhost ~]$ sftp root@127.0.0.1
root@127.0.0.1's password:
Connected to root@127.0.0.1.
sftp> cd /root
sftp> put ls.txt
Uploading ls.txt to /root/ls.txt
ls.txt                                                                      100%   95     4.1KB/s   00:00
sftp> exit
[mickey@localhost ~]$ su -
Password:
[root@localhost ~]# ll | grep ls
-rw-r-----. 1 root root    95 Mar 23 10:30 ls.txt
```

### 遠端-->本地
`get <文件路徑>`
```bash
[mickey@localhost ~]$ sftp root@127.0.0.1
root@127.0.0.1's password:
Connected to root@127.0.0.1.
sftp> cd /etc
sftp> get hosts
Fetching /etc/hosts to hosts
/etc/hosts                                                                  100%  158   281.5KB/s   00:00
sftp> exit
[mickey@localhost ~]$ ll | grep host
-rw-r-----. 1 mickey student   158 Mar 23 10:27 hosts
```

## 🐧rsync
rsync使用SSH加密
`rsync -aAXH --progress <來源路徑> <目標路徑>`，建議用`-aAXH`參數可保留所有信息
- `-r`，遞歸
- `-a`，保留所有屬性，但不含hard link保息、ACL權限及SELinux權限
	1. `-l`，保留soft link
	2. `-p`，保留權限
	3. `-t`，保留修改時間
	4. `-g`，保留GID
	5. `-o`，保留UID
- `-H`，保留hard link信息
- `-A`，保留ACL權限
- `-X`，保留SELinux權限
- `--progress`，顯示進度條

### 本地-->本地
相當於`cp`
```bash
[root@localhost ~]# rsync -aAXH --progress passwd.pdf /home/mickey/Documents/
sending incremental file list
passwd.pdf
         30,106 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=0/1)
[root@localhost ~]# ll /home/mickey/Documents/|grep passwd
-rw-r--r--. 1 root   root    30106 Mar 14 10:30 passwd.pdf
```

### 本地-->遠端
```bash
[root@localhost ~]# rsync -aAHX --progress initial-setup-ks.cfg mickey@127.0.0.1:/home/mickey/Documents
mickey@127.0.0.1's password:
sending incremental file list
initial-setup-ks.cfg
          1,547 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=0/1)
[root@localhost ~]# ll /home/mickey/Documents|grep initial-setup-ks.cfg
-rw-r--r--. 1 mickey student  1547 Mar  7 20:13 initial-setup-ks.cfg
```

### 遠端-->本地
```bash
[root@localhost ~]# rsync -aAHX --progress mickey@127.0.0.1:/home/mickey/Documents/tryzip.zip ~/Documents
mickey@127.0.0.1's password:
receiving incremental file list
tryzip.zip
          1,896 100%    1.81MB/s    0:00:00 (xfr#1, to-chk=0/1)
[root@localhost ~]# ll Documents|grep tryzip.zip
-rw-r-----. 1 mickey student 1896 Mar 22 07:29 tryzip.zip
```