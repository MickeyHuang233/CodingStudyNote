# ACL管理
ACL(Access Control List)，存取控制安全機制，主要管理一個檔案對應多個owner和多個group owner

## 🐧啟用ACL
- xfs文件系統內建ACL
- ext3、ext4無內建ACL，需額外處理
	1. `mount -o acl /dev/vda3/mnt`
	2. `vi /etc/fstab`
	```bash
	/dev/vda3	/mnt	ext4	defaults.acl	1	2
	```
	3. 使用default mount options，如果該partition有設定default mount options，直接mount設備即可使用ACL
		1. 設置default mount options，`tune2fs -o acl,user_xattr /dev/vdb1`
		2. 從RHEL 6開始，format後的partition預設都會有default mount options

## 🐧設置ACL權限 setfacl
權限的比對順序：ACL user owner --> 傳統user owner --> ACL group owner --> 傳統 group owner --> others

### 操作用戶/組ACL權限
- `setfacl -m u:<用戶名>:<rwx權限> <文件|目錄路徑>`，新增/修改用戶ACL權限，設置多個用`,`分開
	`setfacl -m u::<rwx權限> <文件|目錄路徑>`，修改owner權限
	`setfacl -m g:<組名>:<rwx權限> <文件|目錄路徑>`，新增/修改組ACL權限，設置多個用`,`分開
	`setfacl -m g::<rwx權限> <文件|目錄路徑>`，修改group owner權限
	1. `-R`，遞歸 
	```bash
	[mickey@mickey tmp]$ getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	user::rw-
	group::r--
	other::---
	[mickey@mickey tmp]$ setfacl -m u:student:r-- test3.txt
	[mickey@mickey tmp]$ getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	user::rw-
	user:student:r--
	group::r--
	mask::r--
	other::---
	```
- `setfacl -x <u|g>:<用戶名> <文件|目錄路徑>`，刪除用戶/組ACL權限
	```bash
	[mickey@mickey tmp]$ getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	user::rw-
	user:student:r--
	group::r--
	group:testgroup:r--
	mask::r--
	other::---
	[mickey@mickey tmp]$ setfacl -x u:student test3.txt
	[mickey@mickey tmp]$ getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	user::rw-
	group::r--
	group:testgroup:r--
	mask::r--
	other::---
	```
- `getfacl <覆製的文件|目錄路徑> | setfacl --set-file=- <貼上的文件|目錄路徑>`，ACL權限複製
	```bash
	[mickey@mickey tmp]$ getfacl test.txt
	# file: test.txt
	# owner: mickey
	# group: student
	user::rw-
	group::r--
	other::---

	[mickey@mickey tmp]$ getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	# flags: sst
	user::rwx
	user:student:rwx                #effective:r--
	group::rw-                      #effective:r--
	group:testgroup:rw-             #effective:r--
	mask::r--
	other::r--

	[mickey@mickey tmp]$ getfacl test3.txt | setfacl --set-file=- test.txt
	[mickey@mickey tmp]$ getfacl test.txt
	# file: test.txt
	# owner: mickey
	# group: student
	user::rwx
	user:student:rwx                #effective:r--
	group::rw-                      #effective:r--
	group:testgroup:rw-             #effective:r--
	mask::r--
	other::r--
	```
- `--X`，代表x的權限只會更動到目錄，忽略文件，可參考：[[🐧Linux_RH124_08_文檔權限管理]]
	```bash
	[root@mickey mickey]# setfacl -R -m u:student:r-X /home/mickey
	```

### 修改傳統ugo權限
 `setfacl -m <u|g|o>::<rwx權限> <文件|目錄路徑>`，修改傳統ugo權限，設置多個用`,`分開
注意：不能用`chmod`改有ACL權限的文檔或目錄
```bash
[mickey@mickey tmp]$ setfacl -m u::rwx test3.txt
[mickey@mickey tmp]$ setfacl -m g::rw- test3.txt
[mickey@mickey tmp]$ setfacl -m o::r-- test3.txt
[mickey@mickey tmp]$ getfacl test3.txt
# file: test3.txt
# owner: mickey
# group: student
user::rwx
group::rw-
group:testgroup:r--
mask::rw-
other::r--
```

### 修改特殊權限
一樣使用`chmod`修改，可參考：[[🐧Linux_RH124_08_文檔權限管理]]
在`getfacl`中，flags代表特殊權限
```bash
[mickey@mickey tmp]$ ll test3.txt
-rwsr-Sr-T+ 1 mickey student 0 Apr 18 22:44 test3.txt
[mickey@mickey tmp]$ getfacl test3.txt
# file: test3.txt
# owner: mickey
# group: student
# flags: sst
user::rwx
user:student:rwx                #effective:r--
group::rw-                      #effective:r--
group:testgroup:rw-             #effective:r--
mask::r--
other::r--
```

### 修改權限遮罩mask
1. ACL mask作用：避免將權限設置太大
2. `#effective:r--`，代表權限遮罩後的實際權限
```bash
[mickey@mickey tmp]$ getfacl test3.txt
# file: test3.txt
# owner: mickey
# group: student
user::rwx
user:student:rwx
group::rw-
group:testgroup:rw-
mask::rwx
other::r--
[mickey@mickey tmp]$ setfacl -m m::r-- test3.txt
[mickey@mickey tmp]$ getfacl test3.txt
# file: test3.txt
# owner: mickey
# group: student
user::rwx
user:student:rwx                #effective:r--
group::rw-                      #effective:r--
group:testgroup:rw-             #effective:r--
mask::r--
other::r--
```

### 默認ACL權限
==只能設置於目錄上==，但因為umask(可參考：[[🐧Linux_RH124_08_文檔權限管理]])的原因，新建立的文件一定不能有x的限權
- `setfacl -m d:<u|g|m>:[用戶名|組名]:<rwx權限> <目錄路徑>`，設置默認ACL權限
	```bash
	[root@mickey tmp]# setfacl -m d:u:mickey:r-x .
	[root@mickey tmp]# setfacl -m d:g:student:rw- .
	[root@mickey tmp]# setfacl -m d:m:rw- .
	[root@mickey tmp]# getfacl .
	# file: .
	# owner: root
	# group: root
	# flags: --t
	user::rwx
	group::rwx
	other::rwx
	default:user::rwx
	default:user:mickey:r-x         #effective:r--
	default:group::rwx              #effective:rw-
	default:group:student:rw-
	default:mask::rw-
	default:other::rwx
	[root@mickey tmp]# touch test4.txt
	[root@mickey tmp]# getfacl test4.txt
	# file: test4.txt
	# owner: root
	# group: root
	user::rw-
	user:mickey:r-x                 #effective:r--
	group::rwx                      #effective:rw-
	group:student:rw-
	mask::rw-
	other::rw-
	```
- `setfacl -k <目錄路徑>`，取消默認ACL權限
	```bash
	[root@mickey tmp]# setfacl -k .
	[root@mickey tmp]# getfacl .
	# file: .
	# owner: root
	# group: root
	# flags: --t
	user::rwx
	group::rwx
	other::rwx
	[root@mickey tmp]# touch test5.txt
	[root@mickey tmp]# getfacl test5.txt
	# file: test5.txt
	# owner: root
	# group: root
	user::rw-
	group::r--
	other::---
	```

### 拿掉ACL權限
`setfacl -b <文件|目錄路徑>`，將指定文件的ACL權限完全刪掉
```bash
[root@mickey tmp]# ll test3.txt
-rwsr-Sr-T+ 1 mickey student 0 Apr 18 22:44 test3.txt
[root@mickey tmp]# setfacl -b test3.txt
[root@mickey tmp]# ll test3.txt
-rwsr-Sr-T. 1 mickey student 0 Apr 18 22:44 test3.txt
```

## 🐧查看ACL權限
### ACL權限在ls -l顯示
- `.`代表文件沒有ACL權限
![[Linux_RH134_04_ACL管理_01_傳統權限.png]]
- `+`代表文件有ACL權限，前方的rwx不再代表ugo權限，改為u\[mask\]o權限
![[Linux_RH134_04_ACL管理_02_ACL權限.png]]

### getfacl
`getfacl <文件|目錄路徑>`，查看指定目錄的ACL權限
```bash
[mickey@mickey tmp]$ getfacl test2.txt
# file: test2.txt
# owner: mickey
# group: student
user::rw-
user:student:r-x
group::r--
mask::r-x
other::---
```

## 🐧ACL權限備份、還原
- `getfacl -R <目錄相對路徑> > <備份存放路徑>`，ACL備份
	```bash
	[root@mickey tmp]# getfacl -R . > /tmp/backupacl.txt
	```
- `setfacl --restore=<備份存放路徑>`，ACL還原
	```bash
	[root@mickey tmp]# getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	# flags: sst
	user::rwx
	user:root:rw-
	user:student:rwx
	group::rw-
	group:testgroup:rw-
	mask::rwx
	other::r--
	[root@mickey tmp]# setfacl --restore=/tmp/backupacl.txt
	[root@mickey tmp]# getfacl test3.txt
	# file: test3.txt
	# owner: mickey
	# group: student
	# flags: sst
	user::rwx
	user:student:rwx                #effective:r--
	group::rw-                      #effective:r--
	group:testgroup:rw-             #effective:r--
	mask::r--
	other::r--
	```
