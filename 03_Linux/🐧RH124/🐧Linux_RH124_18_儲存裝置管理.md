# 儲存裝置管理
注意：此部分需要與[[🐧Linux_RH134_06_硬碟分區管理分區管理]]一起配合練習

## 🐧簡介
- 每個設備都會對應一個檔案，稱為設備檔(device file)，放至/dev
- 常見設備檔名

	|設備類型|命名規則|
	|---|---|
	|SATA、SAS、USB|從Red Hat 5後硬碟通通視為SCSI設備，sd*<br/>1. 多個設備命名規則：sda、sdb、sdc…<br/>	2. 設備分區命名規則：sda1、sda2、sda3…|
	|vitrio-blk(Red Hat虛擬機才有)<br>Red Hat盤擬化技術-->open source|vda、vdb、vdc…|
	|NVMe介面的SSD|nvme0、nvme1、nvme2…|
	|SD、MMC、eMMC(SD卡)|mmcblk0、mmcblk1、mmcblk2…|
- 虛擬化技術
	![[Linux_RH124_18_儲存裝置管理_01_虛擬化技術.png]]
	1. 無虛擬化技術，用戶可直接在機器操作APP，但當需要全機備份或是還原會很麻煩
	2. 主虛擬化技術，會將虛擬機的硬碟虛擬化並放至qemu-kvm，APP主要對這個虛擬化SCSI硬碟執行讀寫，虛擬化硬碟再對機器的硬碟進行讀寫，所又I/O性能差
	3. 半虛擬化技術，APP的讀寫直接由虛擬機kenel直接寫至機器硬碟，效能快很多，但需要較新的kenel版本

### 分區介紹
1.  mbr分區
	- 最多支持四個主分區
	- 系統只能安裝在主分區
	- 擴展分區要占一個主分區
	- MBR最大只支持2TB，但擁有最好的兼容性
2.  gtp分區
	- 支持無限多個主分區(但操作系統可能限制，比如windows下最多128個分區)
	- 最大支持18EB的容量(EB=1024PB，PB=1024TB)
	- windows7 64位以後支持gtp

windows下的磁盤分區
![[Linux_RH124_12_磁盤分區和挂載_01.png]]

## 🐧查看儲存裝置
### 查看SCSI設備 lsscsi
`lsscsi`，查看系統的SCSI設備
```bash
[mickey@mickey ~]$ lsscsi
[1:0:0:0]    cd/dvd  VBOX     CD-ROM           1.0   /dev/sr0
[2:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda
```

### 查看分區 lsblk
`lsblk -fp`，查看磁碟(block device)和分區對應的目錄，可參考：[lsblk命令](https://man.linuxde.net/lsblk)
1. NAME，分區情況
2. FSTYPE，分區file system類型
3. UUID，唯一標識分區的40位不重複的字符串
4. MOUNTPOINT，挂載點
```bash
[root@mickey ~]# lsblk -fp
NAME                      FSTYPE      LABEL UUID                                   MOUNTPOINT
/dev/sda
├─/dev/sda1               xfs               3944cc4a-5642-491a-a18f-5cd3dd41adfc   /boot
└─/dev/sda2               LVM2_member       xHWtP9-LJJ0-W6Oi-l1gR-Fwzb-o8rl-hmX2Oy
  ├─/dev/mapper/rhel-root xfs               d8486839-239c-49b7-92cc-79c981f4fd92   /
  └─/dev/mapper/rhel-swap swap              ca6d85f5-5860-446c-b240-0884b6617ddf   [SWAP]
/dev/sdb
└─/dev/sdb1               ext4              f3a4a540-ec0e-4264-be83-f4f483ab99d7
/dev/sr0
```

### 查看使用中設備 df
`df [設備檔名]`，列出使用中的儲存設備，也可指定查看儲存設備
1K-blocks、Used、Available，都是以k為單位
```bash
[mickey@mickey ~]$ df
Filesystem            1K-blocks    Used Available Use% Mounted on
devtmpfs                1918228       0   1918228   0% /dev
tmpfs                   1935896       0   1935896   0% /dev/shm
tmpfs                   1935896    9308   1926588   1% /run
tmpfs                   1935896       0   1935896   0% /sys/fs/cgroup
/dev/mapper/rhel-root  15923200 4486484  11436716  29% /
/dev/sda1               1038336  183188    855148  18% /boot
tmpfs                    387176    1180    385996   1% /run/user/42
tmpfs                    387176       4    387172   1% /run/user/1000
```
1. `-h`，以1024計算，差別可參考：[[🗄️為什麼硬盤容量總是縮水？]]
2. `-H`，以1000計算
```bash
[mickey@mickey ~]$ df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.9G     0  1.9G   0% /dev
tmpfs                  1.9G     0  1.9G   0% /dev/shm
tmpfs                  1.9G  9.1M  1.9G   1% /run
tmpfs                  1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   16G  4.3G   11G  29% /
/dev/sda1             1014M  179M  836M  18% /boot
tmpfs                  379M  1.2M  377M   1% /run/user/42
tmpfs                  379M  4.0K  379M   1% /run/user/1000
```

### 查看占用空間 du
`du -hs <目錄路徑>`，查看指定目錄所占用的空間
1. `-h`，占用空間易於查看
2.  `-s`，只要結果
3. `-a`，含文件
4. `-c`，列出明細的同時，增加匯總值
5. `--max-depth=1`，子目錄深度
```bash
[root@mickey ~]# du -hs /var/log
19M     /var/log
```

```bash
[root@localhost ~]# du -hac --max-depth=1 /opt
0       /opt/rh
0       /opt
0       總計
```

## 🐧掛載儲存裝置 mount
![[Linux_RH124_12_磁盤分區和挂載_02.png]]
此部分需要搭配[[🐧Linux_RH134_06_硬碟分區管理分區管理]]使用。

### 以分區名掛載
`mount <設備路徑> <目錄路徑>`，較不常用，因為分區名稱可能會改變
```bash
[root@mickey ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.9G     0  1.9G   0% /dev
tmpfs                  1.9G     0  1.9G   0% /dev/shm
tmpfs                  1.9G  9.1M  1.9G   1% /run
tmpfs                  1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   16G  4.3G   11G  29% /
/dev/sda1             1014M  179M  836M  18% /boot
tmpfs                  379M  1.2M  377M   1% /run/user/42
tmpfs                  379M  4.0K  379M   1% /run/user/0
[root@mickey ~]# mount /dev/sdb1 /home/mickey
[root@mickey ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               1.9G     0  1.9G   0% /dev
tmpfs                  1.9G     0  1.9G   0% /dev/shm
tmpfs                  1.9G  9.1M  1.9G   1% /run
tmpfs                  1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/rhel-root   16G  4.3G   11G  29% /
/dev/sda1             1014M  179M  836M  18% /boot
tmpfs                  379M  1.2M  377M   1% /run/user/42
tmpfs                  379M  4.0K  379M   1% /run/user/0
/dev/sdb1              7.9G   36M  7.4G   1% /home/mickey
```

### 以設備UUID掛載
因為UUID是不會變的，所以比較推薦使用此方法
`mount UUID="<UUID>" <目錄路徑>`，以設備UUID掛載
```bash
[root@mickey ~]# lsblk -fp
NAME                      FSTYPE      LABEL UUID                                   MOUNTPOINT
/dev/sda
├─/dev/sda1               xfs               3944cc4a-5642-491a-a18f-5cd3dd41adfc   /boot
└─/dev/sda2               LVM2_member       xHWtP9-LJJ0-W6Oi-l1gR-Fwzb-o8rl-hmX2Oy
  ├─/dev/mapper/rhel-root xfs               d8486839-239c-49b7-92cc-79c981f4fd92   /
  └─/dev/mapper/rhel-swap swap              ca6d85f5-5860-446c-b240-0884b6617ddf   [SWAP]
/dev/sdb
└─/dev/sdb1               ext4              f3a4a540-ec0e-4264-be83-f4f483ab99d7
/dev/sr0
[root@mickey ~]# mount UUID="f3a4a540-ec0e-4264-be83-f4f483ab99d7" /home/mickey
[root@mickey ~]# lsblk -fp
NAME                      FSTYPE      LABEL UUID                                   MOUNTPOINT
/dev/sda
├─/dev/sda1               xfs               3944cc4a-5642-491a-a18f-5cd3dd41adfc   /boot
└─/dev/sda2               LVM2_member       xHWtP9-LJJ0-W6Oi-l1gR-Fwzb-o8rl-hmX2Oy
  ├─/dev/mapper/rhel-root xfs               d8486839-239c-49b7-92cc-79c981f4fd92   /
  └─/dev/mapper/rhel-swap swap              ca6d85f5-5860-446c-b240-0884b6617ddf   [SWAP]
/dev/sdb
└─/dev/sdb1               ext4              f3a4a540-ec0e-4264-be83-f4f483ab99d7   /home/mickey
/dev/sr0
```

### 可移除式儲存設備掛載
- 可移除式儲存設備，如：CD、USB…
- 圖形模式會自動掛載至：/run/media/<用戶名>/<設備名稱>
- 如不需要則`umount`

### 移除掛載
- `umount <設備路徑>`，移除掛載
	```bash
	[root@mickey ~]# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	devtmpfs               1.9G     0  1.9G   0% /dev
	tmpfs                  1.9G     0  1.9G   0% /dev/shm
	tmpfs                  1.9G  9.1M  1.9G   1% /run
	tmpfs                  1.9G     0  1.9G   0% /sys/fs/cgroup
	/dev/mapper/rhel-root   16G  4.3G   11G  29% /
	/dev/sda1             1014M  179M  836M  18% /boot
	tmpfs                  379M  1.2M  377M   1% /run/user/42
	tmpfs                  379M  4.0K  379M   1% /run/user/0
	/dev/sdb1              7.9G   36M  7.4G   1% /home/mickey
	[root@mickey ~]# umount /dev/sdb1
	[root@mickey ~]# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	devtmpfs               1.9G     0  1.9G   0% /dev
	tmpfs                  1.9G     0  1.9G   0% /dev/shm
	tmpfs                  1.9G  9.1M  1.9G   1% /run
	tmpfs                  1.9G     0  1.9G   0% /sys/fs/cgroup
	/dev/mapper/rhel-root   16G  4.3G   11G  29% /
	/dev/sda1             1014M  179M  836M  18% /boot
	tmpfs                  379M  1.2M  377M   1% /run/user/42
	tmpfs                  379M  4.0K  379M   1% /run/user/0
	```
- 如果有用戶還在umount的路徑則會umount失敗，因此可以用`lsof <路徑>`查看有哪些用戶在指定路徑
	```bash
	[root@mickey ~]# umount /dev/sdb1
	umount: /home/mickey: target is busy.
	[root@mickey ~]# lsof /home/mickey
	COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
	bash    2368 root  cwd    DIR   8,17     4096    2 /home/mickey
	```
