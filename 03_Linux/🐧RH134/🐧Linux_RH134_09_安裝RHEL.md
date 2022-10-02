# Linux_RH134_09_安裝RHEL
RHEL，Red Hat Enterprise Linux
安裝Linux的方法：
1. ISO檔安裝：\*\_boot.iso為Red Hat安裝導引程式，需要連接至YUM Server
	- ISO檔燒成光碟(不建議)
	- 使用[linuxliveusb](https://www.linuxliveusb.com/)將.iso燒至隨身碟
2. 隨身碟還沒mount下指令`dd if-rhel-8.2-x86.64-boot.iso of=/dev/sdb bs=2M`
3. 將已安裝好的作業系統映像檔放入隨身碟/CD，又稱為liveUSB、liveCD，但操作系統中的東西是唯讀的
4. ：將事先做好的磁碟影像檔放入虛擬機
	VM磁碟映像檔路徑：/var/lib/libvirt/images/

## 🐧手動安裝
![Linux_RH134_09_安裝RHEL_01_安裝畫面](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_01_%E5%AE%89%E8%A3%9D%E7%95%AB%E9%9D%A2.png?raw=true)

## 🐧自動化安裝 Kickstart
- Kickstart是用於自動化安裝的程序
- 如何取得Kickstart檔
	1. 安裝完成後此次安裝的Kickstart檔路徑：/root/anaconda-ks.cfg
	2. 圖形界面工具：system-config-kickstart==>Red Hat 7後就沒了
	3. 圖形界面工具：Kickstart Generator website-->付費
	![kickstart-generator](https://www.linuxprobe.com/links/RH134/images/kickstart/kickstart-generator-1.png)
- 注意：取得的kickstart檔需要測試
	1. `yum install pykickstart`，安裝套件
	2. `ksvalidator /root/anaconda-ks.cfg`，沒信息顯示代表測試成功
- 參考：[KICKSTART 語法參照 -Red Hat 7](https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax)

### Kickstart語法
- `#`，注釋
- `%<區塊內容>%end`，區塊
	```bash
	%anaconda
	pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
	pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
	pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
	%end
	```
- `@^`，指定環境
	```bash
	# 安裝所有屬於「Infrastracture Server」（基礎架構伺服器）環境的所有套件
	%packages
	@^Infrastructure Server
	%end
	```
- `@`，指定群組(package group)，前面加`-`表示不安裝，如`-@Graphical Internet`
	```bash
	%packages
	@X Window System
	@Desktop
	@Sound and Video
	%end
	```
- 指定個別套件；前面加`-`表示不安裝，如`-autofs`
	```bash
	%packages
	sqlite
	curl
	aspell
	docbook*
	%end
	```
- `%pre <shell script> %end`，kickstart安裝前執行
- `%post <shell script> %end`，kickstart安裝後執行
	```bash
	%post --nochroot
	cp /etc/resolv.conf /mnt/sysimage/etc/resolv.conf
	%end
	```

### Kickstart常用選項
相關配置可看官方文件，只要知道在哪改就可以了
1. 安裝相關選項
	- `url`，套件安裝的資料來源(`HTTP`、`HTTPS`、`FTP`或`file`)
		```bash
		url --url="http://classroom.example.com/content/rhel8.2/x86_64/dvd/"
		```
	- `repo`，套件安裝額外的資料來源(yum server url)
		```bash
		repo --name="appstream" --baseurl=http://classroom.example.com/content/rhel8.2/x86_64/dvd/AppStream
		```
	- `text`，安裝過程純文字表示
	- `vnc --password=redhat`，可通過VNC client看安裝進度，`--password`則設定密碼
2. 磁碟分區相關選項
	- `clearpart`，在建立新分割區之前，從系統移除分割區
		```bash
		clearpart --drives=hda,hdb --all
		```
	- `part`，分區設置
		```bash
		part /home --fstype=ext4 --label=homes --size=4096 --maxsize=8192 --grow
		```
	- `autopart`，自動建立分區
	- `ignoredisk`，設置安裝系統時不要動指定硬碟，但建議安裝時將硬碟拆掉
		```bash
		ignoredisk --dives=sdc
		```
	- `bootloader`，安裝bootloader至指定硬碟的MBR
		```bash
		bootloader --location=mbr --boot-drive=sda
		```
	- `volgroup`、`logvol`，建立LVM
3. 網絡相關選項
	- `network`，網絡配置
		```bash
		network --device=eth0 --bootproto=dhcp
		```
	- `firewall`，防火墻配置
		```bash
		firewall --enable --service=ssh,http
		```
4. 地區、安全相關配置
	- `lang`，語言配置
		```bash
		lang en_US.UTF-8
		```
	- `keyboard`，鍵盤設置
		```bash
		keyboard --vckeymap=us --xlayouts=''
		```
	- `timezone`，時區
		```bash
		timezone --utc --ntpservers=time.example.com Europ/Amsterdam
		```
	- `authselect`
	- `rootpw`，設置root密碼
		```bash
		rootpw --plaintext redhat
		```
	- `selinux`
		```bash
		selinux --enforcing
		```
	- `services`，設置是否開機啟用的服務
		```bash
		services --disable=network,iptables,ip6tables --enabled=NetworkManager,firewalld
		```
	- `group`、`user`，建立用戶、群組至系統
		```bash
		group --name=admins --gid=10001
		user --name=jdoe --gecos=""John Doe
		 --groups=admins --password=changeme --plaintext
		 ```
5. 安裝完成後的動作設置
	- `logging --host=<log service位置>`，若無指定kickstart安裝完後，log就沒了
	- `firstboot`，第一次開機是否顯示歡迎頁面
		```bash
		firstboot --disabled
		```
	- `reboot`、`poweroff`、 `half`，安裝完成後執行的動作

### Kickstart安裝步驟
1. 將boot.iso放入電腦(CD/隨身碟)
2. 將Kickstart file放入電腦(http/FTP/NFS Server/CD/隨身碟)
3. 開機，在boot loader menu按Tab，並輸入指令指定Kickstart file位置：
	- HTTP Server：inst.ks=http://server/dir/file
	- FTP Server：inst.ks=server/dir/file
	- NFS Server：inst.ks=nfs:server:/dir/file
	- 隨身碟：inst.ks=hd:device:/dir/file
	- CD：inst.ks=cdrom:device
	![Linux_RH134_09_安裝RHEL_02_Kickstart](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_02_Kickstart.png?raw=true)

## 🐧遠端安裝Linux系統
### 機器有公網IP
![Linux_RH134_09_安裝RHEL_03_遠端安裝](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_03_%E9%81%A0%E7%AB%AF%E5%AE%89%E8%A3%9D.png?raw=true)
- 情境描述：被安裝的機器有公網IP，無論安裝者的電腦是否有公網IP
- 執行思路：安裝者的電腦直接向被安裝的機器請求畫面
- 執行步驟：
	1. A：`yum install tigervnc-server`準備
	vnc server配置參考：[CentOS 7 安裝與設定 VNC 伺服器](https://www.footmark.info/linux/centos/centos7-vnc-install-setup/)
	1. B：用光碟機開機，按ESC，輸入指令
		```bash
		boot: linux vnc vncpassword=redhat ip=172.25.250.10::172.25.250.254:24:servera.lab.example.com:enp1s0:none
		```
	2. A：vncviewer
	3. A：`<用戶publice IP>:1`

### 機器有無公網IP
![Linux_RH134_09_安裝RHEL_04_遠端安裝](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_04_%E9%81%A0%E7%AB%AF%E5%AE%89%E8%A3%9D.png?raw=true)
- 情境描述：被安裝的機器沒有公網IP，安裝者的電腦一定要有公網IP
- 執行思路：安裝者的電腦先監聽請求，被安裝的機器向安裝者電腦發出請求
- 執行步驟：
	1. A：`vncviewer --listen`
	2. B：用光碟機開機，按ESC，輸入指令
		```bash
		boot: linux vnc vncconnect=172.25.254.250 ip=172.25.250.10::172.25.250.254:24:servera.lab.example.com:enp1s0:none
		```

## 🐧虛擬化技術
![Linux_RH134_10_Container管理_01_虛擬機架構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_10_Container%E7%AE%A1%E7%90%86_01_%E8%99%9B%E6%93%AC%E6%A9%9F%E6%9E%B6%E6%A7%8B.png?raw=true)
- KVM(Kernel-based Virtual Machine)，Linux開源管理虛擬機工具，使用`virsh`指令
- RHV(Red Hat Virtualization)，使用瀏覽器管理虛擬機工具(只有一個管理節點)
	![Linux_RH134_09_安裝RHEL_05_RHVM示意](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_05_RHVM%E7%A4%BA%E6%84%8F.png?raw=true)
- Red Hat OpenStack Platform(RHOSP)，虛擬機器管理平台

### 使用虛擬化條件
1. `yum module install virt`，下載套件
2. `virt-host-validate`，檢查環境是否符合虛擬化技術
	前兩個代表硬體支援虛擬化技術，必須PASS才可用
	```bash
	[mickey@vm104 ~]$ virt-host-validate
	  QEMU: Checking for hardware virtualization                                 : FAIL (Only emulated CPUs are available, performance will be significantly limited)
	  QEMU: Checking if device /dev/vhost-net exists                             : PASS
	  QEMU: Checking if device /dev/net/tun exists                               : PASS
	  QEMU: Checking for cgroup 'cpu' controller support                         : PASS
	  QEMU: Checking for cgroup 'cpuacct' controller support                     : PASS
	  QEMU: Checking for cgroup 'cpuset' controller support                      : PASS
	  QEMU: Checking for cgroup 'memory' controller support                      : PASS
	  QEMU: Checking for cgroup 'devices' controller support                     : PASS
	  QEMU: Checking for cgroup 'blkio' controller support                       : PASS
	WARN (Unknown if this platform has IOMMU support)
	  QEMU: Checking for secure guest support                                    : WARN (Unknown if this platform has Secure Guest support)
	```

### Cockpit界面管理
- 工具安裝
	1. `yum install cockpit-machines`
	2. `systemctl enable --now cockpit.socket`
- 不建議使用，因為功能很陽春
	![Cockpit管理虛擬機](https://www.tecmint.com/wp-content/uploads/2021/01/Select-Virtual-Machines.jpg)

### virsh工具管理
![Linux_RH134_09_安裝RHEL_06_virsh](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_06_virsh.png?raw=true)
- 界面管理工具：virtual machine manager，使用參考官方文檔：[Red Hat 7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_getting_started_guide/virt-manager-user-interface-description)
	![virtual machine manager](https://access.redhat.com/webassets/avalon/d/Red_Hat_Enterprise_Linux-7-Virtualization_Getting_Started_Guide-en-US/images/5a5567cf374c05ba00ecfe248ea00c64/virt-machine-manager_vm_list-window.png)
- 指令管理工具：`virsh`
	- 基本用法
		1. `virsh shutdown servera`
		2. `virsh start servera`
		3. `virsh reset servera`
		4. `virsh reboot servera`
		5. `virsh destroy servera`，相當於把電腦插座拔掉
	- 備份虛擬機
		1. `virsh dumpxml servera > /tmp/servera.xml`，配置檔備份
		2. 備份虛擬機系統映像檔：/var/lib/libvirt/images
	- 還原虛擬機
		1. 映像檔放至：/var/lib/libvirt/images
		2. `virsh define /tmp/servera.xml`，配置檔還原
	- 設置網路橋接器(bridge)：指令可參考-->[[🐧Linux_RH124_16_網絡管理]]
	![Linux_RH134_09_安裝RHEL_07_網路橋接器示意](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_07_%E7%B6%B2%E8%B7%AF%E6%A9%8B%E6%8E%A5%E5%99%A8%E7%A4%BA%E6%84%8F.png?raw=true)
		1. `nmcli connection add con-name br0 type bridge ifname virbr0 ipv4.address 192.168.56.102/24 ipv4.gateway 192.168.250.254 ipv4.dns 8.8.8.8 ipv4.method manual`，建立網路橋接器
		2. `nmcli connection add con-name eth0 type bridge-slave ifname enp0s8 master br0`，將網卡接上網路橋接器，eth0為bridge所有，因此eth0沒有IP，IP在br0
		3. 設置bridge IP為固定IP
	- fedora virtio-->在windows虛擬機用virtio：/user/share/virtio-win/installer/virtio-win-guest-tool.exe

### 虛擬機映像檔處理工具 qemu-img
- 建立磁碟映像檔
	1. SSD用：`qemu-img create -f qcowz Windows1-.qcow2 100G`
	2. 機械硬碟用：`qemu-img create -f raw Windows1-.qcow2 100G`
- 磁碟映像檔格式轉換
	1. 實體機-->虛擬機：`qemu-imgconrert -f raw -O qcow2 /dev/sdb Window2019.qcow2`
	2. 虛擬機-->虛擬機
	3. 虛擬機-->實體機
- 使用此工具可實現以下虛擬機結構-->共用重覆的虛擬機檔案-->使用空間更小、方便還原
	實體機最好不要用樓械硬碟
	![Linux_RH134_09_安裝RHEL_08_虛擬機映像檔處理工具](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_09_%E5%AE%89%E8%A3%9DRHEL_08_%E8%99%9B%E6%93%AC%E6%A9%9F%E6%98%A0%E5%83%8F%E6%AA%94%E8%99%95%E7%90%86%E5%B7%A5%E5%85%B7.png?raw=true)
	1. `qemu-img create -f qcow2 -b AAA.qcow2 BBB.qcow2`
	2. `qemu-img info BBB.qcow2`
	

	 