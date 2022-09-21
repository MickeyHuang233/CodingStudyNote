# Linux_RH124_05_文件目錄操作
## 🐧通用的表達方式
### ~
- `~`，代表用戶家目錄
- `~<用戶名>`，代表指定用戶的家目錄(root才可看)
	```
	[root@localhost ~]# echo ~mickey
	/home/mickey
	```

### {m,n} {m..p}
用於字串顯示
- {m,n}，表示m和n
- {m..p}，表示m~p，m、n、o、p
```bash
[root@localhost Documents]# echo file_{1,2}
file_1 file_2
[root@localhost Documents]# echo file_{m..p}
file_m file_n file_o file_p
[root@localhost Documents]# touch file_{a..b}{1,2}
[root@localhost Documents]# ls
file_a1  file_a2  file_b1  file_b2
```

## 🐧查看文件、目錄
### file
`file <檔案路徑>`，從檔案內容查看檔案格式，而不是從副檔名查看
```bash
[mickey@localhost ~]$ file /home/mickey/.bash_history
/home/mickey/.bash_history: ASCII text
```

### pwd
`pwd`，顯示當前目錄絕對路徑
```bash
[root@localhost mickey]# pwd
/home/mickey
```

### wc
- wc(word count)，`wc [選項] [目錄或文件]`
	輸出說明：行數	文字數(前後空白算一個字)	字元數	文檔路徑
	```bash
	[mickey@localhost ~]$ wc /home/mickey/.bash_history
	 12  16 105 /home/mickey/.bash_history
	```
- 統計包括子文件夾裡的文件個數
	```bash
	[root@localhost ~]# ls -lR /root| grep "d*" | wc -l
	46
	```

### ls
* `ls [選項] [目錄或文件]` ，查看當前目錄所有的文件和目錄
	```bash
	[root@localhost mickey]# ls
	下載  公共  圖片  影片  文件  桌面  模板  音樂
	```
* `-a`，包括穩藏檔案、目錄，以.開頭
	`.`，當前目錄
	`..`，上一層目錄
	```bash
	[root@localhost mickey]# ls -a
	.              .bash_logout   .cache   .esd_auth      .mozilla  下載  影片  模板
	..             .bash_profile  .config  .ICEauthority  .pki      公共  文件  音樂
	.bash_history  .bashrc        .dbus    .local         .viminfo  圖片  桌面
	```
* `-R`，列出目錄下所有的文件、目錄
	```bash
	[root@localhost mickey]# ls -R
	.:
	Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos

	./Desktop:

	./Documents:
	ifconfig.txt  selectName.txt  spilt_aa  spilt_ab  spilt_ac  spilt_ad  spilt_ae

	./Downloads:

	./Music:

	./Pictures:

	./Public:

	./Templates:

	./Videos:
	```
* `-l`，以列表的方式顯示信息，相當於指令`ll`，權限說明可參考：[[🐧Linux_RH124_08_文檔權限管理]]
	```bash
	[root@localhost mickey]# ls -l
	總計 0
	drwxr-xr-x. 3 mickey mickey 21  1月  1 12:15 下載
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 公共
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 圖片
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 影片
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 文件
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 桌面
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 模板
	drwxr-xr-x. 2 mickey mickey  6 12月 31 23:04 音樂
	```
* `-i`，查看檔案的inode編號
	每個信息都是檔案的metadata，這些信息都放在Linux的某個文件中，並將每一個檔案的metadata編號，這個編號就是inode
	```bash
	[root@localhost Documents]# ls -li
	total 28
	27656384 -rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
	27656401 -rw-rw-r--. 1 mickey mickey  273 Mar  9 02:16 selectName.txt
	```
* `-d`，查看目錄
	```bash
	[root@localhost Documents]# ls -d /root/
	/root/
	```
* `ls [正則表達式]`，查看符合條件的文檔
	```bash
	[root@localhost ~]# ls a*
	anaconda-ks.cfg
	```

### stat
`stat <文檔名>`，查看文件inode
```bash
[root@localhost Documents]# stat ifconfig.txt
  File: ifconfig.txt
  Size: 2463            Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 27656384    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/  mickey)   Gid: ( 1000/  mickey)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2021-03-10 08:46:50.055463437 -0500
Modify: 2021-03-09 01:59:27.520592145 -0500
Change: 2021-03-09 01:59:27.520592145 -0500
 Birth: -
```
Access：文件最後被讀取的時間
Modify：文件最後被修改時間
Change：inode內容被改變的最後時間

### tree
`tree <目錄路徑>`，以樹狀顯示目錄結構
```bash
[root@localhost ~]# tree
.
├── anaconda-ks.cfg
├── initial-setup-ks.cfg
├── \344\270\213\350\274\211
│   ├── testCreate
│   │   └── testTest
│   ├── testCrontab.txt
│   ├── testTarFolder.tar.gz
│   ├── testTar.tar.gz
│   ├── testTest
│   └── testViEditor
├── \345\205\254\345\205\261
├── \345\234\226\347\211\207
├── \345\275\261\347\211\207
├── \346\226\207\344\273\266
├── \346\241\214\351\235\242
├── \346\250\241\346\235\277
└── \351\237\263\346\250\202
```

## 🐧查找相關
### locate
- `locate`利用事先建立的系統中所有文件名稱及路徑的locate數據庫實現快速定位給定的文件，因此無需遍歷整個文件系統，查詢速度較快。
- 系統只會在淩晨更新數據庫，資料較不及時，若要求及時數據管理員手動更新數據。
	```bash
	[root@localhost ~]# updatedb
	```
- `locate <關鍵字>`，查詢有指定關鍵字的路徑
	1. `-i`，不區分大小寫
	2. `-n <筆數>`，顯示指定筆數，相當於`| head -n <筆數>`
	3. `-r`，用正則表達式找，參考：[[🍀正則表達式]]
	```bash
	[mickey@mickey Documents]$ locate -n 5 testDir
	/home/mickey/Documents/testDir
	/home/mickey/Documents/testDir_
	/home/mickey/Documents/testDir/empty1
	/home/mickey/Documents/testDir/empty2
	/home/mickey/Documents/testDir/empty3
	```

### find
- 真正在目錄中找條件相符的路徑，因此速度慢，但及時
- 詳細用法可參考：[Unix/Linux 的 find 指令使用教學、技巧與範例整理](https://blog.gtwang.org/linux/unix-linux-find-command-examples/)
- `find [搜索范圍(路徑)] [條件]`，若無指定搜索范圍則從當前路徑開始找
	1. `-name`，檔名(區分大小寫)
	2. `-iname`，檔名(不區分大小寫)
	3. `-type`，檔案類型，d-->目錄，f-->一般檔案，l-->連結檔，s-->socket，p-->pipe
	4. `-user`，所有者
	5. `-group`，所有組
	6. `-uid`
	7. `-gid`
	8. `-links <數量>`，硬連接數量，+代表大於，-代表小於
	```bash
	[root@localhost ~]# find / -name '*test*'
	[root@localhost ~]# find /opt -user root
	```

#### 複合條件
1. 複合條件若無特別指定，關系為and
	```bash
	[root@mickey ~]# find / -name "*test*" -user mickey
	/home/mickey/Documents/testDir_
	/home/mickey/Documents/testDir_/test1
	/home/mickey/Documents/testDir_/test2
	/home/mickey/Documents/testDir_/test3
	/home/mickey/Documents/testDir_/test4
	```
2. `-o`，複合條件關系為or
	```bash
	[root@mickey ~]# find / -name mickey -o -perm 000
	/dev/pts/ptmx
	/run/cron.reboot
	/run/systemd/inaccessible
	/run/systemd/inaccessible/blk
	/run/systemd/inaccessible/chr
	```
3. `!`，複合條件關系為not
	```bash
	[root@mickey ~]# find / -name '*test*' ! -perm 777
	/boot/grub2/i386-pc/functional_test.mod
	/boot/grub2/i386-pc/bswap_test.mod
	/boot/grub2/i386-pc/cmp_test.mod
	```

#### 文件大小
k、M、G
```bash
[root@localhost ~]# find / -size +20M
```
1. `find / -size +20M`，>20M
2. `find / -size 20M`，=20M
3. `find / -size -20M`，<20M

#### 文件權限
1. 查找權限等於777
```bash
[root@mickey ~]# find /home -perm 777
```
2. 權限u含有3(wx)、g含有2(w)、o含有4(rw)，同時成立；0表示忽略
	![Linux_RH124_06_文件目錄操作_02_權限查找](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_06_%E6%96%87%E4%BB%B6%E7%9B%AE%E9%8C%84%E6%93%8D%E4%BD%9C_02_%E6%AC%8A%E9%99%90%E6%9F%A5%E6%89%BE.png?raw=true)
```bash
[root@mickey ~]# find /home -perm -324
[root@mickey ~]# find /home -perm -004
```
3. 取限u為4、g為4、o為2，其中一個成立
```bash
[root@mickey ~]# find /home -perm /442
```

#### 存取、修改時間
||以日為單位|以分鐘為單位|
|---|---|---|
|最後存取時間(access time)|atime|amin|
|最後修改時間(modification time)|mtime|mmin|
|檔案狀態最後修改時間(status time)|ctime|cmin|
- 分別對應`stat`下的時間
	```bash
	[root@mickey ~]# stat /home/mickey/.ssh/authorized_keys
	  File: /home/mickey/.ssh/authorized_keys
	  Size: 580             Blocks: 8          IO Block: 4096   regular file
	Device: fd00h/64768d    Inode: 17294282    Links: 1
	Access: (0600/-rw-------)  Uid: ( 1000/  mickey)   Gid: ( 1001/ student)
	Context: unconfined_u:object_r:ssh_home_t:s0
	Access: 2021-03-29 22:37:45.392360833 +0800
	Modify: 2021-03-29 22:36:29.557503846 +0800
	Change: 2021-03-29 22:36:29.557503846 +0800
	 Birth: -
	```
- 7天前的那天修改過的檔案
	```bash
	[root@mickey ~]# find /home -mtime 7
	```
- 7~14天內修改過的檔案
	```bash
	[root@mickey ~]# find -mtime +7 -mtime -14
	```

### which
`which <指令名>`，找指定指令的執行檔路徑
```bash
[root@localhost ~]# which cat
/usr/bin/cat
```

### grep
`grep`：在文件中找某個字串，支援正則表達式
`egrep`則支援延伸正則表達式-->[[🍀正則表達式]]
- 推薦書籍：[sed & awk](https://docstore.mik.ua/orelly/unix/sedawk/index.htm)-->還有搜索、替代的功能

|需另外注意表達|說明|
|---|---|
|`\{i\}`|i個前面的字元，如：`a\{5\}`|
|`\{i,\}`|大於等於i個前面的字元，如：`a\{5,\}`|
|`\{i,j\}`|i~j個前面的字元，如：`a\{5,8\}`|
|`\+`|一個或多個前面的字元，如：`a\+`等同於`a\{1,\}`|
|`\?`|零個或一個前面的字元，如：`a\?`等同於`a\{0,1\}`|
|`^`|一行的開頭|
|`$`|一行的結尾|
|`\<`|一個單字的開頭|
|`\>`|一個單字的結尾|

- `-n`顯示行號
	```bash
	[root@localhost 下載]# cat testTest | grep -n testTest
	16:-rw-r--r--.  1 root root   542  1月  5 20:09 testTest
	```
- `-i`不區分大小寫
	```bash
	[root@localhost 下載]# cat testTest | grep -ni testtest
	16:-rw-r--r--.  1 root root   542  1月  5 20:09 testTest
	```
- `-v`，反向
	```bash
	[mickey@mickey ~]$ ll | grep -v student
	total 72
	-rw-r--r--. 1 root   root       95 Mar 14 23:11 ls.txt
	-rw-r--r--. 1 root   root    30106 Mar 14 22:32 passwd.pdf
	-rw-r--r--. 1 root   root    19947 Mar 14 22:28 passwd.ps
	```
- `-r`，遞歸
- `-A <行數>`，也顯示符合條件的後n行(after)
	```bash
	[mickey@mickey ~]$ ll | grep -A 2 Music
	drwxr-xr-x. 2 mickey student     6 Mar  8 09:13 Music
	-rw-r--r--. 1 root   root    30106 Mar 14 22:32 passwd.pdf
	-rw-r--r--. 1 root   root    19947 Mar 14 22:28 passwd.ps
	```
- `-B <行數>`，也顯示符合條件的前n行(before)
- `-c`，顯示配對成功數，count
	```bash
	[mickey@mickey ~]$ ll | grep -v student -c
	4
	```
- `-l`，只顯示符合條件的文檔名
	```bash
	[mickey@mickey ~]$ grep localhost -r ./ -l
	./.config/evolution/sources/system-proxy.source
	./.local/share/tracker/data/tracker-store.journal
	./.local/share/app-info/xmls/extensions-web.xml
	```
- `-f`，搜索條件讀取文件
	```bash
	[mickey@mickey ~]$ cat ls.txt
	Desktop
	Documents
	Downloads
	[mickey@mickey ~]$ ll | grep -f ls.txt
	drwxr-xr-x. 2 mickey student     6 Mar  8 09:13 Desktop
	drwxr-xr-x. 4 mickey student   204 Apr 11 11:13 Documents
	drwxr-xr-x. 2 mickey student     6 Mar  8 09:13 Downloads
	```

## 🐧查看文檔內容
### cat
- `cat <文件名>`，瀏覽文件內容(只讀)
- `cat -n <文件名>`，瀏覽文件並顯示行號(只讀)
	```bash
	[root@localhost 下載]# cat -n testViEditor
		 1  public static void main(String[] args){
		 2          System.out.println("Hello World");
		 3  }
		 4  //this is try to create file by vim in linux
	```
- `cat -A <文件名>`，可查看文檔控制字元
	```bash
	[root@localhost Documents]# cat -A selectName.txt
		 1^Ienp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500$
		10^Ienp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500$
		18^Ienp0s9: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500$
		25^Ienp0s10: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500$
	[root@localhost Documents]#
	```

|瀏覽文件常用命令|說明|
|---|---|
|Enter|向下n行，需要定义，默认为1行|
|Ctrl+F 或 空格键|向下滚动一屏|
|Ctrl+B|返回上一屏|
|\=|输出当前行的行号|
|:f|输出文件名和当前行的行号|
|V|调用vi编辑器|
|!命令|调用Shell，并执行命令|
|q|退出more|

### more & less
`more`，使用分頁形式打開文件
```bash
[root@localhost 下載]# cat -n testViEditor | more
```
`less`，和`more`功能類似，但對於顯示大型文件具有較高的效率，參考：[Linux cat，more，less等命令](https://blog.csdn.net/ahafg/article/details/53410632)
```bash
[root@localhost 下載]# less testViEditor
```

### head
`head <文件名>`，顯示文件部分內容，默認情況下head指令顯示的前10行內容
```bash
[root@localhost 下載]# head testTest
總計 8
-rw-------. 1 root root 1522 12月 31 22:49 anaconda-ks.cfg
-rw-r--r--. 1 root root 1570 12月 31 23:02 initial-setup-ks.cfg
drwxr-xr-x. 3 root root   85  1月  5 20:07 下載
drwxr-xr-x. 2 root root    6 12月 31 23:11 公共
drwxr-xr-x. 2 root root    6 12月 31 23:11 圖片
drwxr-xr-x. 2 root root    6 12月 31 23:11 影片
drwxr-xr-x. 2 root root    6 12月 31 23:11 文件
drwxr-xr-x. 2 root root    6 12月 31 23:11 桌面
drwxr-xr-x. 2 root root    6 12月 31 23:11 模板
```
`head -n 5 <文件名>`，顯示文件內容前5行
```bash
[root@localhost 下載]# head -n 5 testTest
總計 8
-rw-------. 1 root root 1522 12月 31 22:49 anaconda-ks.cfg
-rw-r--r--. 1 root root 1570 12月 31 23:02 initial-setup-ks.cfg
drwxr-xr-x. 3 root root   85  1月  5 20:07 下載
drwxr-xr-x. 2 root root    6 12月 31 23:11 公共
```

### tail
`tail <文件名>`，顯示文件後10行內容
`tail -n 5 <文件名>`，顯示文件後5行內容
`tail -f <文件名>`，實時追蹤文檔的所有更新【常用】
```bash
[root@localhost 下載]# tail -f testTest
-rw-r--r--.  1 root root   263  1月  5 12:48 testViEditor
-rw-r--r--.  1 root root 12288  1月  5 12:49 .testViEditor.swp
      一月 2020
日 一 二 三 四 五 六
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31
```

## 🐧文件操作
### cd
* `cd <目錄名(絕對路徑或相對路徑)>`：切換指定目錄
* `cd ..`：回父目錄
* `cd /`：回根目錄
* `cd~`或`cd`：回到自己的家目錄
* `cd -`，回到上一個路徑
	```bash
	[root@localhost Documents]# cd ~/Music/
	[root@localhost Music]# cd -
	/home/mickey/Documents
	[root@localhost Documents]# cd -
	/root/Music
	```

### mkdir
`mkdir <目錄名>`，創建目錄
```bash
[root@localhost 下載]# ls
testViEditor
[root@localhost 下載]# mkdir testCreate
[root@localhost 下載]# ll
總計 4
drwxr-xr-x. 2 root root   6  1月  5 15:54 testCreate
-rw-r--r--. 1 root root 263  1月  5 12:48 testViEditor
```

`mkdir -p <目錄絕對/相對路徑>`：一次建立多級目錄
```bash
[root@localhost mickey]# mkdir /home/mickey/下載/testCreateMultiFolder/newFolder_01
mkdir: 無法建立目錄'/home/mickey/下載/testCreateMultiFolder/newFolder_01': 沒有此一檔案或目錄
[root@localhost mickey]# mkdir -p /home/mickey/下載/testCreateMultiFolder/newFolder_01
[root@localhost mickey]# cd 下載
[root@localhost 下載]# ls
testCreateMultiFolder  testScp
[root@localhost 下載]# cd testCreateMultiFolder/
[root@localhost testCreateMultiFolder]# ls
newFolder_01
[root@localhost testCreateMultiFolder]#
```

### ~~rmdir~~
`rmdir`，刪除空白目錄【很少用】
```bash
[root@localhost testCreateMultiFolder]# ls
newFolder_01
[root@localhost testCreateMultiFolder]# rmdir newFolder_01/
[root@localhost testCreateMultiFolder]# ll
總計 0
```

### rm
`rm -rf <目錄絕對/相對路徑>`，刪除文件或目錄
* `-r`，遞歸刪除整個文件夾
* `-f`，強制刪除不提示
```bash
[root@localhost 下載]# rmdir testCreateMultiFolder/
rmdir: failed to remove 'testCreateMultiFolder/': 目錄不是空的
[root@localhost 下載]# rm -rf testCreateMultiFolder/
[root@localhost 下載]# ls
testScp
```

### touch
`touch <文件名1> [<文件名2>…]`，創建空文件
```bash
[root@localhost 下載]# ls
testScp
[root@localhost 下載]# touch touchNewFile.txt touchNewFile01.txt
[root@localhost 下載]# ls
testScp  touchNewFile.txt touchNewFile01.txt
```

### cp
`cp <文件名> <複製至目錄的絕對/相對路徑>`，複製文件到指定目錄
注意：複製出的文檔的擁有者、所在組會變成複製者、複製者所在組，而不是原擁有者
```bash
[root@localhost Documents]# ll
total 8
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
[root@localhost Documents]# cp ifconfig.txt ifconfig_1.txt
[root@localhost Documents]# ll
total 12
-rw-r--r--. 1 root   root   2463 Mar 10 09:32 ifconfig_1.txt
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
```

`cp -r <文件名> <拷貝至目錄的絕對/相對路徑>`，遞歸複製整個文件夾
```bash
[root@localhost 下載]# cp testCpFolder testCpMultiFolder
cp: 略過 'testCpFolder' 目錄
[root@localhost 下載]# cp -r testCpFolder testCpMultiFolder
cp：是否覆寫 'testCpMultiFolder/testCpFolder/touchNewFile.txt'? y
```

`cp -p`，保留文件原始屬性(擁有者、所在組…)
```bash
[root@localhost Documents]# ll
total 12
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
[root@localhost Documents]# cp -p ifconfig.txt ifconfig_2.txt
[root@localhost Documents]# ll
total 16
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig_2.txt
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
```

`cp -d`，保留文件soft link屬性，如不指定則將soft link中的文檔也一并複製

### mv
`mv`，移動文件目錄或重命名
* 重命名
	```bash
	testCpFolder  testCpMultiFolder  testScp  touchNewFile.txt
	[root@localhost 下載]# mv touchNewFile.txt testMvFile.txt
	[root@localhost 下載]# ls
	testCpFolder  testCpMultiFolder  testMvFile.txt  testScp
	```
* 移動文件
	```bash
	[root@localhost 下載]# mv /testMvFile.txt /home/mickey/下載/testCpFolder/testMvFile.txt
	[root@localhost 下載]# cd testCpFolder/
	[root@localhost testCpFolder]# ls
	testFolder  testMvFile.txt  touchNewFile.txt
	```

### split
`split -b <每個文件byte> <被分解的文檔路徑> [分解成的文件開頭]`，支解大文檔
```bash
[root@localhost Documents]# ll
total 8
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
-rw-rw-r--. 1 mickey mickey  273 Mar  9 02:16 selectName.txt
[root@localhost Documents]# split -b 500 ifconfig.txt spilt_
[root@localhost Documents]# ll
total 28
-rw-rw-r--. 1 mickey mickey 2463 Mar  9 01:59 ifconfig.txt
-rw-rw-r--. 1 mickey mickey  273 Mar  9 02:16 selectName.txt
-rw-r--r--. 1 root   root    500 Mar 10 08:51 spilt_aa
-rw-r--r--. 1 root   root    500 Mar 10 08:51 spilt_ab
-rw-r--r--. 1 root   root    500 Mar 10 08:51 spilt_ac
-rw-r--r--. 1 root   root    500 Mar 10 08:51 spilt_ad
-rw-r--r--. 1 root   root    463 Mar 10 08:51 spilt_ae
[root@localhost Documents]#
```
注：`cat spilt_* > ifconfig.txt`，組合被支解的大文件

### ln
- `ln -s [原文件或目錄] [軟鏈接名] `，建立軟鏈接，類似於Windows的快捷方式，主要存放了鏈接其他文件路徑基本語法
	注意：軟鏈接權限都為全開，不可修改；**最後的權限依據以指向的目錄為主**。
	```bash
	[root@localhost 下載]# ln -s /home/mickey linkToMickey
	[root@localhost 下載]# ll
	總計 8
	lrwxrwxrwx. 1 root root   12  1月  5 20:29 linkToMickey -> /home/mickey
	drwxr-xr-x. 2 root root    6  1月  5 15:54 testCreate
	-rw-r--r--. 1 root root 1038  1月  5 20:12 testTest
	-rw-r--r--. 1 root root  263  1月  5 12:48 testViEditor
	[root@localhost 下載]# cd linkToMickey
	[root@localhost linkToMickey]# pwd
	/root/下載/linkToMickey
	```
- 刪除軟鏈接
	```bash
	[root@localhost 下載]# rm -r linkToMickey
	rm：是否移除符號連結'linkToMickey'? y
	[root@localhost 下載]# ll
	總計 8
	drwxr-xr-x. 2 root root    6  1月  5 15:54 testCreate
	-rw-r--r--. 1 root root 1038  1月  5 20:12 testTest
	-rw-r--r--. 1 root root  263  1月  5 12:48 testViEditor
	```
- 建立硬鏈接
	```bash
	[root@localhost mickey]# ln ./Documents/ifconfig.txt hardlink
	[root@localhost mickey]# ll
	total 4
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Desktop
	drwxr-xr-x. 2 mickey mickey   92 Mar 10 09:33 Documents
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Downloads
	-rw-rw-r--. 2 mickey mickey 2463 Mar  9 01:59 hardlink
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Music
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Pictures
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Public
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Templates
	drwxr-xr-x. 2 mickey mickey    6 Mar  7 20:13 Videos
	```

**目錄、軟鏈接、硬鏈接存儲方式**
hard link不能跨device，因此實務上很少使用，soft link則無此問題。
![Linux_RH124_06_文件目錄操作_01_目錄、軟鏈接、硬鏈接存儲方式](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_06_%E6%96%87%E4%BB%B6%E7%9B%AE%E9%8C%84%E6%93%8D%E4%BD%9C_01_%E7%9B%AE%E9%8C%84%E3%80%81%E8%BB%9F%E9%8F%88%E6%8E%A5%E3%80%81%E7%A1%AC%E9%8F%88%E6%8E%A5%E5%AD%98%E5%84%B2%E6%96%B9%E5%BC%8F.png?raw=true)
