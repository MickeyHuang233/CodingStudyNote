# Linux文檔壓縮
## 🐧zip
### 壓縮 zip
`zip <壓縮檔名稱> <需壓縮的文檔路徑>`，壓縮，保留原文件
- `-r`，壓縮目錄
- `-y`，只將softlink的名稱壓縮起來；若無此參數預設會將softlink所指的整個目錄一起壓縮
```bash
[mickey@localhost Documents]$ zip -ry tryzip.zip testDir/
  adding: testDir/ (stored 0%)
  adding: testDir/test1 (stored 0%)
  (中間省略)
  adding: testDir/testsoftlink (stored 0%)
[mickey@localhost Documents]$ ll
total 16
drwsrwsrwt. 2 mickey student  161 Mar 22 07:25 testDir
-rw-r-----. 1 mickey student 1896 Mar 22 07:29 tryzip.zip
```

### 解壓縮 unzip
`unzip <壓縮檔路徑>`，解壓縮，保留原文件
- `-d <路徑>`，解壓至指定路徑
```bash
[mickey@localhost Documents]$ unzip tryzip.zip
Archive:  tryzip.zip
   creating: testDir/
 extracting: testDir/test1
  (中間省略)
    linking: testDir/testsoftlink    -> /etc/profile
finishing deferred symbolic links:
  testDir/testsoftlink   -> /etc/profile

```

## 🐧gzip
### 壓縮 gzip
`gzip <文件路徑>`，壓縮，不會保留原文件，不能壓縮資料夾
```bash
[mickey@localhost Documents]$ gzip tryzip.zip
[mickey@localhost Documents]$ ll
total 16
-rw-r-----. 1 mickey student  334 Mar 22 07:29 tryzip.zip.gz
```

### 解壓縮 gunzip
- `gunzip <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ gunzip tryzip.zip.gz
	```
- `gzip -d <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ gzip -d tryzip.zip.gz
	```

## 🐧bzip2
### 壓縮 bzip2
`bzip2 <文件路徑>`，壓縮，不會保留原文件，不能壓縮資料夾
```bash
[mickey@localhost Documents]$ bzip2 tryzip.zip
[mickey@localhost Documents]$ ll
total 16
-rw-r-----. 1 mickey student  371 Mar 22 07:29 tryzip.zip.bz2
```

### 解壓縮 bunzip2
- `bunzip2 <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ bunzip2 tryzip.zip.bz2
	```
- `bzip2 -d <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ bzip2 -d tryzip.zip.bz2
	```

## 🐧xz
### 壓縮 xz
`xz <文件路徑>`，壓縮，不會保留原文件，不能壓縮資料夾
```bash
[mickey@localhost Documents]$ xz tryzip.zip
[mickey@localhost Documents]$ ll
total 16
-rw-r-----. 1 mickey student  324 Mar 22 07:29 tryzip.zip.xz
```

### 解壓縮 unxz
- `unxz <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ unxz tryzip.zip.xz
	```
- `xz -d <壓縮文件路徑>`，解壓縮
	```bash
	[mickey@localhost Documents]$ xz -d tryzip.zip.xz
	```

## 🐧tar
### 打包
`tar -cvf <包裝檔路徑.tar> <文件|目錄路徑>`，可打包多個文件或目錄
1. `-c`，建立新的包裝檔.tar
2. `-v`，顯示執行過程信息
3. `-f`，打包名稱
4. `--xattrs`，連同檔案的[[🐧Linux_RH134_04_ACL管理]]和[[🐧Linux_RH134_05_SELinux]]上下文一同打包，建議使用
```bash
[mickey@localhost Documents]$ tar -cvf testTar.tar testDir hosts
testDir/
(中間省略)
testDir/empty5
testDir/testsoftlink
hosts
[mickey@localhost Documents]$ ll
total 28
-rwsr-xr-T. 1 mickey student  1422 Mar 18 09:50 hosts
drwxrwxrwx. 2 mickey student   161 Mar 22 07:25 testDir
-rw-r-----. 1 mickey student 10240 Mar 23 07:45 testTar.tar
```

### 解包
`tar -xvf <包裝檔路徑.tar>`，解包至當前路徑
1. `-x`，解開包裝檔
```bash
[mickey@localhost tar]$ tar -xvf ../testTar.tar
testDir/
(中間省略)
testDir/empty5
testDir/testsoftlink
hosts
[mickey@localhost tar]$ ll
total 4
-rwxr-x---. 1 mickey student 1422 Mar 18 09:50 hosts
drwxr-x---. 2 mickey student  161 Mar 22 07:25 testDir
```

### 列出打包內容
`tar -tvf <包裝檔路徑.tar>`
1. `-t`，列出包裝檔內容
2. 協同工作
	1. `-z`，gzip-->`-ztvf`
	2. `-j`，bzip2
	3. `-J`，xz
```bash
[mickey@localhost testDir]$ tar -tvf testTar.tar
drwxrwxrwx mickey/student    0 2021-03-22 07:25 testDir/
(中間省略)
-rw-r----- mickey/student    0 2021-03-22 07:23 testDir/empty5
lrwxrwxrwx mickey/student    0 2021-03-22 07:33 testDir/testsoftlink -> /etc/profile
-rwsr-xr-T mickey/student 1422 2021-03-18 09:50 hosts
```

### 壓縮(協同工作)
`tar -cvf <包裝檔路徑.tar> <文件|目錄路徑>`
1. `-z`，gzip-->`-zcvf`
2. `-j`，bzip2
3. `-J`，xz
	```bash
	[root@localhost 下載]# ll
	總計 8
	drwxr-xr-x. 2 root root  22  1月 12 10:06 testCreate
	-rw-r--r--. 1 root root 283  1月 12 10:38 testTest
	-rw-r--r--. 1 root root 263  1月  5 12:48 testViEditor
	[root@localhost 下載]# tar -zcvf testTar.tar.gz testTest testViEditor
	testTest
	testViEditor
	[root@localhost 下載]# ll
	總計 12
	drwxr-xr-x. 2 root root  22  1月 12 10:06 testCreate
	-rw-r--r--. 1 root root 635  1月 12 10:39 testTar.tar.gz
	-rw-r--r--. 1 root root 283  1月 12 10:38 testTest
	-rw-r--r--. 1 root root 263  1月  5 12:48 testViEditor
	```

### 解壓縮(協同工作)
`tar -xvf <包裝檔路徑.tar> <解壓至路徑>`
1. `-z`，gzip-->`-zxvf`
2. `-j`，bzip2
3. `-J`，xz
	```bash
	[root@localhost 下載]# ll
	總計 8
	drwxr-xr-x. 2 root root  22  1月 12 10:06 testCreate
	-rw-r--r--. 1 root root 542  1月 12 10:42 testTarFolder.tar.gz
	-rw-r--r--. 1 root root 635  1月 12 10:39 testTar.tar.gz
	[root@localhost 下載]# tar -zxvf testTar.tar.gz
	testTest
	testViEditor
	[root@localhost 下載]# ll
	總計 16
	drwxr-xr-x. 2 root root  22  1月 12 10:06 testCreate
	-rw-r--r--. 1 root root 542  1月 12 10:42 testTarFolder.tar.gz
	-rw-r--r--. 1 root root 635  1月 12 10:39 testTar.tar.gz
	-rw-r--r--. 1 root root 283  1月 12 10:38 testTest
	-rw-r--r--. 1 root root 263  1月  5 12:48 testViEditor
	```
	注意：解壓指令，解壓到的目錄要事先存在才能成功
	```bash
	[root@localhost 下載]# tar -zxvf testTar.tar.gz /home/home/
	tar: /home/home：在保存檔中找不到
	tar: 由於先前錯誤而以失敗狀態離開
	```







