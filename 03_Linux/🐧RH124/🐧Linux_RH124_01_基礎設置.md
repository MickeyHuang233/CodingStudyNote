# Linux_RH124_01_基礎設置
## 🐧環境安裝
- Virtual Box：[VirtualBox 虛擬機器軟體安裝與設定教學](https://www.kjnotes.com/freeware/17)
- IOS：
	- [SentOS官網](https://www.centos.org/)
	- [Rocky Linux官網](https://rockylinux.org/)
- CentOS 8 練習環境需要先裝[EPEL](https://fedoraproject.org/wiki/EPEL)
	```bash
	yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	```

## 🐧網路設置
參考：[Virtual Box 網路卡設定介紹篇 – 初階NAT](https://ithelp.ithome.com.tw/articles/10107536)
1. VirtualBox設置
\[虛擬機\]-->\[設定\]-->\[網路\]
![Linux_RH124_RH124_01_基礎設置_01_網路設置](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_01_%E5%9F%BA%E7%A4%8E%E8%A8%AD%E7%BD%AE_01_%E7%B6%B2%E8%B7%AF%E8%A8%AD%E7%BD%AE.png?raw=true)

2. 設置CentOS
	- 若NAT連線成功則可連外網
	- 設置固定ip，否則重新啟動後ip會不一樣
		```bash
		nmcli conn modify "Wired connection 1" ipv4.addresses 192.168.56.101/24 ipv4.gateway 255.255.255.0 ipv4.method manual
		```
	- 更新連線信息、重新連線
		```bash
		nmcli conn down "Wired connection 1"
		nmcli conn reload
		nmcli conn up "Wired connection 1"
		```

3. 取得虛擬機(CentOS)ip
	```bash
	[mickey@localhost ~]$ ifconfig
	enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
			inet 192.168.56.101  netmask 255.255.255.0  broadcast 192.168.56.255
			inet6 fe80::6aeb:2736:b1e2:99f9  prefixlen 64  scopeid 0x20<link>
			ether 08:00:27:e2:61:1a  txqueuelen 1000  (Ethernet)
			RX packets 362  bytes 33536 (32.7 KiB)
			RX errors 0  dropped 0  overruns 0  frame 0
			TX packets 244  bytes 29549 (28.8 KiB)
			TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	// 其他省略
	```
	
4. 主機(Windows)ping虛擬機(CentOS)
	```bash
	PS C:\Users\a0909> ping 192.168.56.101

	Ping 192.168.56.101 (使用 32 位元組的資料):
	回覆自 192.168.56.101: 位元組=32 時間<1ms TTL=64
	回覆自 192.168.56.101: 位元組=32 時間<1ms TTL=64
	回覆自 192.168.56.101: 位元組=32 時間<1ms TTL=64
	回覆自 192.168.56.101: 位元組=32 時間<1ms TTL=64

	192.168.56.101 的 Ping 統計資料:
		封包: 已傳送 = 4，已收到 = 4, 已遺失 = 0 (0% 遺失)，
	大約的來回時間 (毫秒):
		最小值 = 0ms，最大值 = 0ms，平均 = 0ms
	```
          
## 🐧使用putty遠端連線
- 也可使用xshell：[官網](https://www.netsarang.com/zh/xshell/)

![[Linux_RH124_01_基礎設置_02_網路設置.png]]

## 🐧使用WinSCP遠端傳輸文件
- 也可使用XFTP：[官網](https://www.netsarang.com/zh/xftp/)

![Linux_RH124_01_基礎設置_03_網路設置](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_01_%E5%9F%BA%E7%A4%8E%E8%A8%AD%E7%BD%AE_03_%E7%B6%B2%E8%B7%AF%E8%A8%AD%E7%BD%AE.png?raw=true)
![Linux_RH124_01_基礎設置_04_網路設置](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_01_%E5%9F%BA%E7%A4%8E%E8%A8%AD%E7%BD%AE_04_%E7%B6%B2%E8%B7%AF%E8%A8%AD%E7%BD%AE.png?raw=true)

## 🐧VirtualBox設定共用資料夾
參考：[VirtualBox 4.0設定共用資料夾](http://blog.xuite.net/yh96301/blog/41935362-VirtualBox+4.0%E8%A8%AD%E5%AE%9A%E5%85%B1%E7%94%A8%E8%B3%87%E6%96%99%E5%A4%BE)