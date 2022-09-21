# Linux_RH124_06_IO流轉向
![Linux_RH124_07_IO流轉向_01](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_01.png?raw=true)

| channel number | channel name | description     | default connection | usage             |
| -------------- | ------------ | --------------- | ------------------ | ----------------- |
| 0              | stdin        | standard input  | keyboard           | read only         |
| 1              | stdout       | standard output | Terminal           | write only        |
| 2              | stderr       | standard output | Terminal           | write only        |
| 3+             | filename     | other files     | none               | read and/or write |

## 🐧輸出轉向
`>`，將原來的文件的內容覆蓋
`>>`，不會覆蓋原來文件的內容，而是追加到文件的尾部

- `<指令> 1><文檔路徑>`，可省略1為`<指令> ><文檔路徑>`，將輸出的**一般**信息轉向至指定文件
	![Linux_RH124_07_IO流轉向_02](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_02.png?raw=true)
	```bash
	[root@localhost mickey]# ls 1>ls.txt
	[root@localhost mickey]# ls *txt
	ls.txt
	[root@localhost mickey]# cat ls.txt
	Desktop
	Documents
	Downloads
	ls.txt
	```
- `<指令> 2><文檔路徑>`，將輸出的**錯誤**信息轉向至指定文件
	![Linux_RH124_07_IO流轉向_03](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_03.png?raw=true)
	```bash
	[mickey@localhost ~]$ passwd root 2>error.txt
	```
- `<指令> > /dev/null`，將輸出信息丟到垃圾桶
	![Linux_RH124_07_IO流轉向_04](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_04.png?raw=true)
	```bash
	[mickey@localhost ~]$ ls >/dev/null
	```
- `<指令> > file_1 2>file_2`，將輸出的**一般**信息、**錯誤**信息分別轉向至兩個指定文件
	![Linux_RH124_07_IO流轉向_05](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_05.png?raw=true)
	```bash
	[mickey@localhost ~]$ ls > file_1 2>file_2
	```
- `<指令> > file_1 2>&1`，**一般**信息轉向指定文件，**錯誤**信息轉向stdout渠道
	![Linux_RH124_07_IO流轉向_06](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_06.png?raw=true)

## 🐧輸入轉向
![Linux_RH124_07_IO流轉向_07](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_07.png?raw=true)
```bash
[mickey@localhost ~]$ tr a-z A-Z < /etc/hosts
127.0.0.1   LOCALHOST LOCALHOST.LOCALDOMAIN LOCALHOST4 LOCALHOST4.LOCALDOMAIN4
::1         LOCALHOST LOCALHOST.LOCALDOMAIN LOCALHOST6 LOCALHOST6.LOCALDOMAIN6
```
[tr命令說明-菜鳥教程](https://www.runoob.com/linux/linux-comm-tr.html)

## | pipelines
`<指令>|<指令>`，將前面的輸出結果轉至後面的語句執行
![Linux_RH124_07_IO流轉向_08](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_07_IO%E6%B5%81%E8%BD%89%E5%90%91_08.png?raw=true)
```bash
[mickey@localhost ~]$ ifconfig enp0s10 | grep 'inet ' | awk '{print $2}'
192.168.56.102
```

## tee測錄器
`<指令> | tee <文件路徑>`，將結果同時輸出至屏幕及文件
參考：[\[Linux\] tee 指令：將結果同時輸出到螢幕和檔案](https://www.onejar99.com/linux-command-tee/)
```bash
[mickey@localhost ~]$ ifconfig enp0s10 | grep 'inet ' | tee teeFile| awk '{print $2}'
192.168.56.102
[mickey@localhost ~]$ cat teeFile
        inet 192.168.56.102  netmask 255.255.255.0  broadcast 192.168.56.255
```