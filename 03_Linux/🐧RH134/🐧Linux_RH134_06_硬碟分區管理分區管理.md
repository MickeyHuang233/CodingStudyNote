# Linux_RH134_06_硬碟分區管理分區管理
![Linux_RH134_06_硬碟分區管理分區管理_01_硬碟邏輯結構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_01_%E7%A1%AC%E7%A2%9F%E9%82%8F%E8%BC%AF%E7%B5%90%E6%A7%8B%E5%9C%96.png?raw=true)
- 因為過去的設計，硬碟最大分區只能有4個；但extended partition中還可以有多個primary partition
	流程：開機-->BIOS的post程序呼叫int 13h程式-->讀取MBR硬碟
- 過去硬碟最大容量為2T，但現在硬碟廠商將每個block容量加大，所以現在硬碟的容量可以到16T
	![Linux_RH134_06_硬碟分區管理分區管理_02_硬碟最大容量計算](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_02_%E7%A1%AC%E7%A2%9F%E6%9C%80%E5%A4%A7%E5%AE%B9%E9%87%8F%E8%A8%88%E7%AE%97.png?raw=true)

## 🐧建立分區 parted
### 交談式
- `help`，查看幫助
- `print`，查看分區
- `mklabel`，建立partition table格式
	1. msdos，代表MBR
	2. gpt
- `mkpart`，建立分區；第一個分區的start需要為2048s(4k對齊)
- `quit`，離開
```bash
[root@vb101 ~]# parted /dev/sdc
GNU Parted 3.2
Using /dev/sdc
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel
New disk label type? msdos
(parted) mkpart
Partition type?  primary/extended? primary
File system type?  [ext2]? ext4
Start? 2048s
End? 25M
(parted) print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdc: 105MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  25.2MB  24.1MB  primary  ext4         lba

(parted) quit
Information: You may need to update /etc/fstab.
```

### 非交談式
只是將交談式的語句用空格隔開即可
```bash
[root@mickey ~]# parted /dev/sdd mkpart primary xfs 26M 50M print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdd: 105MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  25.2MB  24.1MB  primary
 2      26.2MB  50.3MB  24.1MB  primary  xfs          lba

Information: You may need to update /etc/fstab.
```

### 確認分區完成
`udevadm settle`

## 🐧分區備份
- partition table備份
	`dd if=/dev/sda of=/tmp/sda.dd bs=512 count=1`，備份512 bytes且只複製一次
	1. `if`，input file
	2. `of`，output file
- `dd if=\tmp\sda.dd of=/dev/sda`，partition table還原
- `dd if=/dev/sda1 of=/tmp/sda1.dd bs=2M`，分區備份(一次寫2M)
- `dd if=/dev/sda1 bs=2M | gzip -9 | dd of=/tmp/sda1.dd.gzip`，分區備份並打包
- 分區還原只需將備份反過來寫即可

---
# Swap
Swap為虛擬記憶體，當實體記憶體(RAM)不夠時才會啟動Swap

## 🐧Swap建立流程
### 建立分區 parted
File system type為`linux-swap`
```bash
[root@vm104 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   28G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   27G  0 part
  ├─cl-root 253:0    0 24.2G  0 lvm  /
  └─cl-swap 253:1    0  2.8G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk
sr0          11:0    1 1024M  0 rom
[root@vm104 ~]# parted /dev/sdb
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel
New disk label type? msdos
(parted) mkpart
Partition type?  primary/extended? primary
File system type?  [ext2]? linux-swap
Start? 2048s
End? 5GB
(parted) print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system     Flags
 1      1049kB  5000MB  4999MB  primary  linux-swap(v1)  lba

(parted) quit
Information: You may need to update /etc/fstab.

[root@vm104 ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   28G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   27G  0 part
  ├─cl-root 253:0    0 24.2G  0 lvm  /
  └─cl-swap 253:1    0  2.8G  0 lvm  [SWAP]
sdb           8:16   0   10G  0 disk
└─sdb1        8:17   0  4.7G  0 part
sr0          11:0    1 1024M  0 rom
```

### 建立file system
`mkswap <分區路徑>`
```bash
[root@vm104 ~]# lsblk -fs | grep "sdb1"
sdb1
[root@vm104 ~]# mkswap /dev/sdb1
Setting up swapspace version 1, size = 4.7 GiB (4998557696 bytes)
no label, UUID=279ab4cf-1316-4f0c-a22e-a40d425ce1de
[root@vm104 ~]# lsblk -fs | grep "sdb1"
sdb1    swap              43d2bd9c-f3e3-4239-a823-a717708372c7
```

### mount
- `pri=<優先程度>`，若有多個swap，數字越大代表優先使用
- swap無mount point
- /etc/fstab
	```bash
	[root@vm104 ~]# cat /etc/fstab
	/dev/mapper/cl-root                             /       xfs     defaults        0       0
	UUID=f80d769d-eb65-47d7-a1ff-7cdc1b9d69ab       /boot   xfs     defaults        0       0
	/dev/mapper/cl-swap                             none    swap    defaults        0       0
	UUID=43d2bd9c-f3e3-4239-a823-a717708372c7       swap    swap    pri=10          0       0
	[root@vm104 ~]# mount -a
	```

## 🐧使用狀況查看
- `free`查看硬碟使用狀況
	```bash
	[root@vm104 ~]# free
				  total        used        free      shared  buff/cache   available
	Mem:        3826004      537160     2797352       10648      491492     3056044
	Swap:       2936828           0     2936828
	```
- `swapon -s`，查看swap使用狀況(`--show`)
	默認情況下優先程度為-2
	```bash
	[root@vm104 ~]# swapon -s
	Filename                                Type            Size    Used    Priority
	/dev/dm-1                               partition       2936828 0       -2
	```
- `swapon <swap路徑|UUID=UUID>`，開啟swap
	```bash
	[root@vm104 ~]# swapon UUID=43d2bd9c-f3e3-4239-a823-a717708372c7
	```
- `swapoff <swap路徑|UUID=UUID>`，關閉swap
- `swapon -a`，和`mount -a`作用一樣

---
# LVM
LVM，Logical Volume Management，特點為可以彈性的調整filesystem的容量
![[Linux_RH134_06_硬碟分區管理分區管理_03_LVM概念.png]]
- PEs(Physical Extents)，partition最基本單位，必須為2<sup>n</sup>，2、4、8、16、32…
- PV(Physical Volumes)：經由硬碟切割出來的實體partition
- VG(Volume Group)：由多個 PV建立出的磁區再組合成一個虛擬的磁碟
- LV(Logical Volume)：最後被使用的部份

## 🐧LVM建立流程
### 1. 建立physical device
Virtual Box新增虛擬硬碟：
![Linux_RH134_06_硬碟分區管理分區管理_03_新增虛擬硬碟](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_03_%E6%96%B0%E5%A2%9E%E8%99%9B%E6%93%AC%E7%A1%AC%E7%A2%9F.png?raw=true)
```bash
[mickey@vb101 ~]$ lsblk -fp
NAME                   FSTYPE LABEL UUID                                   MOUNTPOINT
/dev/sda
├─/dev/sda1            xfs          1059413f-59b6-472a-985e-a1f7a9b9db3e   /boot
└─/dev/sda2            LVM2_m       U8FHEC-0tmm-3fbX-joHW-cekt-oeBs-O4WwXK
  ├─/dev/mapper/rhel-root
  │                    xfs          67edd16c-9653-42fd-a9c7-e80f2455aeca   /
  └─/dev/mapper/rhel-swap
                       swap         dbb5fca2-eca1-4c3f-a5db-4122ba970323   [SWAP]
/dev/sdb
/dev/sdc
/dev/sr0
```

### 2. 建立PV
- `pvscan`，列出所有PV
	```bash
	[root@vb101 ~]# pvscan
	  PV /dev/sda2   VG rhel            lvm2 [<19.00 GiB / 0    free]
	  Total: 1 [<19.00 GiB] / in use: 1 [<19.00 GiB] / in no VG: 0 [0   ]
	```
- `pvcreate <分區|設備路徑>`，建立PV
	```bash
	[root@mickey ~]# pvcreate /dev/sdd1
	  Physical volume "/dev/sdd1" successfully created.
	[root@mickey ~]# pvcreate /dev/sdc
	  Physical volume "/dev/sdc" successfully created.
	```
- `pvdisplay`，查看PV詳細信息
	```bash
	[root@mickey ~]# pvdisplay
	  --- NEW Physical volume ---
	  PV Name               /dev/sdc
	  VG Name
	  PV Size               100.00 MiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               wfXimP-ZkMi-M0mY-0QxD-wZyV-RJ71-Lp9JJ9

	  "/dev/sdd1" is a new physical volume of "23.00 MiB"
	  --- NEW Physical volume ---
	  PV Name               /dev/sdd1
	  VG Name
	  PV Size               23.00 MiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               U1gyND-brIN-Hmj1-3Z4c-3Grv-Orxi-Lomz88
	```

### 3. 建立VG
- `vgscan`，列出所有VG
	```bash
	[root@mickey ~]# vgscan
	  Found volume group "rhel" using metadata type lvm2
	```
- `vgcreate -s <PE大小> <VG名稱> <分區|設備路徑> [分區|設備路徑 ...]`，建立VG，可由多個PV組成
	1. `-s <PE大小>`，PV大小為2<sup>n</sup>，2、4、8、16、32…，若無指定則由系統指定
	2. 一個VG底下所有PV的PE大小都一樣
	```bash
	[mickey@vm101 ~]$ sudo vgcreate -s 4M myvg1 /dev/sdb1
	Volume group "myvg1" successfully created
	```
- `vgdisplay`，查看VG詳細信息
	```bash
	[root@mickey ~]# vgdisplay
	  --- Volume group ---
	  VG Name               rhel
	  System ID
	  Format                lvm2
	  Metadata Areas        1
	  Metadata Sequence No  3
	  VG Access             read/write
	  VG Status             resizable
	  MAX LV                0
	  Cur LV                2
	  Open LV               2
	  Max PV                0
	  Cur PV                1
	  Act PV                1
	  VG Size               <17.00 GiB
	  PE Size               4.00 MiB
	  Total PE              4351
	  Alloc PE / Size       4351 / <17.00 GiB
	  Free  PE / Size       0 / 0
	  VG UUID               ZjX9HI-jC7E-7imK-0Lp6-b3cm-K1bf-yC2Lnr
	```

### 4. 建立LV
- `lvscan`，列出所有LV
	```bash
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/swap' [1.80 GiB] inherit
	  ACTIVE            '/dev/rhel/root' [<15.20 GiB] inherit
	```
- `lvcreate -L <容量(M|G|T)> -n <LV名稱> <VG名稱>`，從指定VG拿指定容量給LV使用
	1. `-L <容量(M|G|T)>`，指定容量(如200M)
	2. `-l <PE數量>`，指定PE數量
	3. `-n <LV名稱>`，指定LV名稱
	```bash
	[root@mickey ~]# lvcreate -L 10M -n lvtest1 rhel
	  Rounding up size to full physical extent 12.00 MiB
	  Logical volume "lvtest1" created.
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/swap' [1.80 GiB] inherit
	  ACTIVE            '/dev/rhel/root' [<15.20 GiB] inherit
	  ACTIVE            '/dev/rhel/lvtest1' [12.00 MiB] inherit
	```
- `lvdisplay`，查看LV詳細信息
	```bash
	[root@mickey ~]# lvdisplay
	  --- Logical volume ---
	  LV Path                /dev/rhel/lvtest1
	  LV Name                lvtest1
	  VG Name                rhel
	  LV UUID                d6jVq2-qxDW-cXvy-Pbo8-1iC3-eF5B-FUjVYg
	  LV Write Access        read/write
	  LV Creation host, time mickey.try, 2021-04-22 21:30:16 +0800
	  LV Status              available
	  # open                 0
	  LV Size                12.00 MiB
	  Current LE             3
	  Segments               1
	  Allocation             inherit
	  Read ahead sectors     auto
	  - currently set to     8192
	  Block device           253:2
	```

### 5. 格式化LV
1. `mkfs.xfs <LV路徑>`，格式化為xfs
2. `mkfs.ext4 <LV路徑>`，格式化為ext4
	```bash
	[root@mickey ~]# mkfs.ext4 /dev/rhel/lvtest1
	mke2fs 1.44.6 (5-Mar-2019)
	Creating filesystem with 12288 1k blocks and 3072 inodes
	Filesystem UUID: aaf21e39-b80f-48d7-b7e3-c2a3dfeca666
	Superblock backups stored on blocks:
			8193

	Allocating group tables: done
	Writing inode tables: done
	Creating journal (1024 blocks): done
	Writing superblocks and filesystem accounting information: done
	```

### 6. mount
可參考：[[🐧Linux_RH124_18_儲存裝置管理]]
- `mount <LV路徑> <挂載點路徑>`
	```bash
	[root@mickey ~]# mkdir /mnt/mickey
	[root@mickey ~]# mount /dev/rhel/lvtest1 /mnt/mickey/
	[root@mickey ~]# df -h
	Filesystem                Size  Used Avail Use% Mounted on
	devtmpfs                  1.9G     0  1.9G   0% /dev
	/dev/mapper/rhel-root      16G  4.8G   11G  32% /
	/dev/sda1                1014M  179M  836M  18% /boot
	/dev/mapper/rhel-lvtest1   11M  204K  9.6M   3% /mnt/mickey
	```

### 7. 設置自動掛載
- 設定檔路徑：/etc/fstab
- 加以下內容：
	LVM可以使用device路徑掛載，因為除非重建一個LV外，LV的名稱不會改變
	```bash
	# 設備路徑				      掛載點路徑       file system類型 掛載參數
	/dev/mapper/rhel-lvtest1	/mnt/mickey     ext4           defaults        0 0
	```
- 注意：設定完成使用`mount -a`測試，無信息顯示表示成功，否則如果設定錯誤將無法開機


## 🐧LVM容量修改流程
### 擴充VG
1.  `pvcreate <分區|設備路徑>`，建立PV
2. `vgextend <VG名> <分區|設備路徑>`，擴充VG
	```bash
	[root@mickey ~]# vgdisplay
	  --- Volume group ---
	  VG Name               rhel
	  (省略)
	  Total PE              4351
	  Alloc PE / Size       4351 / <17.00 GiB
	  Free  PE / Size       0 / 0
	  VG UUID               ZjX9HI-jC7E-7imK-0Lp6-b3cm-K1bf-yC2Lnr

	[root@mickey ~]# vgextend rhel /dev/sdd1
	  Volume group "rhel" successfully extended
	[root@mickey ~]# vgdisplay
	  --- Volume group ---
	  VG Name               rhel
	  (省略)
	  Total PE              4356
	  Alloc PE / Size       4351 / <17.00 GiB
	  Free  PE / Size       5 / 20.00 MiB
	  VG UUID               ZjX9HI-jC7E-7imK-0Lp6-b3cm-K1bf-yC2Lnr
	```

### 移除PV
1. `pvmove <來源PV路徑> [目的PV路徑]`，搬移PV上的資料，若無指定目的PV則由系統指定
	```bash
	[root@mickey ~]# pvmove /dev/sdd1 /dev/sdd2
	  /dev/sdd1: Moved: 100.00%
	```
2. `vgreduce <VG名> <PV路徑>`，VG移除指定PV
	```bash
	[root@mickey ~]# vgreduce rhel /dev/sdd1
	  Removed "/dev/sdd1" from volume group "rhel"
	```
3. `pvremove <PV路徑>`，移除PV
	```bash
	[root@mickey ~]# pvremove /dev/sdd1
	  Labels on physical volume "/dev/sdd1" successfully wiped.
	  [root@mickey ~]# pvscan
	  PV /dev/sda2   VG rhel            lvm2 [<17.00 GiB / 0    free]
	  PV /dev/sdd2   VG rhel            lvm2 [20.00 MiB / 8.00 MiB free]
	  PV /dev/sdb                       lvm2 [8.00 GiB]
	  Total: 3 [<25.02 GiB] / in use: 2 [<17.02 GiB] / in no VG: 1 [8.00 GiB]
	```

### 擴充LV
![Linux_RH134_06_硬碟分區管理分區管理_04_LV包含檔案系統](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_04_LV%E5%8C%85%E5%90%AB%E6%AA%94%E6%A1%88%E7%B3%BB%E7%B5%B1.png?raw=true)
LV包含檔案系統，所以兩個都擴充才算是真正上的擴充
1. 放大LV
	-  `lvextend -L +8M <LV路徑>`，加容量
	-  `lvextend -L 8M <LV路徑>`，指定目標容量
	-  `lvextend -l +5 <LV路徑>`，加PE
	-  `lvextend -l 20 <LV路徑>`，指定PE
	```bash
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/lvtest1' [12.00 MiB] inherit
	[root@mickey ~]# lvextend -L +8M /dev/rhel/lvtest1
  	Size of logical volume rhel/lvtest1 changed from 12.00 MiB (3 extents) to 20.00 MiB (5 extents).
  	Logical volume rhel/lvtest1 successfully resized.
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/lvtest1' [20.00 MiB] inherit
	  [root@mickey ~]# df -h
	Filesystem                Size  Used Avail Use% Mounted on
	/dev/mapper/rhel-lvtest1   11M  204K  9.6M   3% /mnt/mickey
	```
2. 擴充file system
	- `resize2fs <LV路徑>` ，ext4 file system
	- `xfs_growfs <LV路徑>` ，xfs file system
	```bash
	[root@mickey ~]# df -h
	/dev/mapper/rhel-lvtest1   11M  204K  9.6M   3% /mnt/mickey
	[root@mickey ~]# resize2fs /dev/mapper/rhel-lvtest1
	resize2fs 1.44.6 (5-Mar-2019)
	Filesystem at /dev/mapper/rhel-lvtest1 is mounted on /mnt/mickey; on-line resizing required
	old_desc_blocks = 1, new_desc_blocks = 1
	The filesystem on /dev/mapper/rhel-lvtest1 is now 20480 (1k) blocks long.
	[root@mickey ~]# df -h
	Filesystem                Size  Used Avail Use% Mounted on
	/dev/mapper/rhel-lvtest1   19M  204K   18M   2% /mnt/mickey
	```

### 縮小LV
步驟和擴充相反，先縮小file system再縮小LV
注意：xfs檔案系統"只能放大"

1. umount
	```bash
	[root@mickey ~]# umount /dev/rhel/lvtest1
	```
2. `e2fsck -f <LV路徑>`，檔查檔案系統有無問題，有則修複
	```bash
	[root@mickey ~]# e2fsck -f /dev/rhel/lvtest1
	e2fsck 1.44.6 (5-Mar-2019)
	Pass 1: Checking inodes, blocks, and sizes
	Pass 2: Checking directory structure
	Pass 3: Checking directory connectivity
	Pass 4: Checking reference counts
	Pass 5: Checking group summary information
	/dev/rhel/lvtest1: 11/4608 files (0.0% non-contiguous), 1815/20480 blocks
	```
3. `resize2fs <LV路徑> <容量>`，縮小ext4 file system
	```bash
	[root@mickey ~]# resize2fs /dev/rhel/lvtest1 12M
	resize2fs 1.44.6 (5-Mar-2019)
	Resizing the filesystem on /dev/rhel/lvtest1 to 12288 (1k) blocks.
	The filesystem on /dev/rhel/lvtest1 is now 12288 (1k) blocks long.
	```
4. `lvreduce -L <容量> <LV路徑>`，縮小LV，也可指定PE
	```bash
	[root@mickey ~]# lvreduce -L 12M /dev/rhel/lvtest1
	  WARNING: Reducing active logical volume to 12.00 MiB.
	  THIS MAY DESTROY YOUR DATA (filesystem etc.)
	Do you really want to reduce rhel/lvtest1? [y/n]: y
	  Size of logical volume rhel/lvtest1 changed from 20.00 MiB (5 extents) to 12.00 MiB (3 extents).
	  Logical volume rhel/lvtest1 successfully resized.
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/swap' [1.80 GiB] inherit
	  ACTIVE            '/dev/rhel/root' [<15.20 GiB] inherit
	  ACTIVE            '/dev/rhel/lvtest1' [12.00 MiB] inherit
	```

## 🐧LVM移除流程
1. umount
	如果有自動掛載記得從設定檔拿掉
	```
	[root@mickey ~]# umount /dev/mapper/rhel-lvtest1
	[root@mickey ~]# rm -rf /mnt/mickey
	[root@mickey ~]# vi /etc/fstab
	[root@mickey ~]# mount -a
	```
2. 移除LV
	```bash
	[root@mickey ~]# lvremove /dev/mapper/rhel-lvtest1
	Do you really want to remove active logical volume rhel/lvtest1? [y/n]: y
	  Logical volume "lvtest1" successfully removed
	[root@mickey ~]# lvscan
	  ACTIVE            '/dev/rhel/swap' [1.80 GiB] inherit
	  ACTIVE            '/dev/rhel/root' [<15.20 GiB] inherit
	```
3. `vgremove <vg名稱>`，移除VG
4. 移除PV
	```bash
	[root@mickey ~]# pvscan
	  PV /dev/sda2   VG rhel            lvm2 [<17.00 GiB / 0    free]
	  PV /dev/sdd2   VG rhel            lvm2 [20.00 MiB / 20.00 MiB free]
	  PV /dev/sdb                       lvm2 [8.00 GiB]
	  Total: 3 [<25.02 GiB] / in use: 2 [<17.02 GiB] / in no VG: 1 [8.00 GiB]
	[root@mickey ~]# pvremove /dev/sdb
	  Labels on physical volume "/dev/sdb" successfully wiped.
	[root@mickey ~]# pvscan
	  PV /dev/sda2   VG rhel            lvm2 [<17.00 GiB / 0    free]
	  PV /dev/sdd2   VG rhel            lvm2 [20.00 MiB / 20.00 MiB free]
	  Total: 2 [<17.02 GiB] / in use: 2 [<17.02 GiB] / in no VG: 0 [0   ]
	```

## 🐧額外內容：LVM下資料庫備份
為了確保資料庫備分時資料不再異動，備份分為：
1. off line backup，停機時間久，系統損失大
2. on line backup，專業備份軟件，需要花很多錢
3. 折中方法：資料庫關機建立快照後再啟動，用shell做約3~5s
	1. 資料庫停機
	2. `lvcreate -L 100M -s -n backup /dev/ugo/data`，建立快照，快照LV容量不需要一比一建立
	3. 資料庫開機
	4. 進行資料備份

快照原理：快照沒有複製任何文件至LV，當有異動才會將舊的內容貼至快照LV
![Linux_RH134_06_硬碟分區管理分區管理_05_快照原理](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_05_%E5%BF%AB%E7%85%A7%E5%8E%9F%E7%90%86.png?raw=true)

---
# stratis
![Linux_RH134_06_硬碟分區管理分區管理_06_stratis架構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_06_stratis%E6%9E%B6%E6%A7%8B.png?raw=true)
- stratis主要功能：快照(snapshots)、 精簡配置(thin provisioning)、分層(tiering)
- stratis中每個pool可切的file system數量為2<sup>24</sup>個
- 使用套件：`yum install stratisd stratis-cli`
- 開啟服務：`systemctl start --now stratisd`

## 🐧建立一般file system流程
### 建立pool
1. `stratis pool create <pool名稱> <硬碟路徑>`，建立pool
	```bash
	[root@mickey ~]# stratis pool create mypool1 /dev/sdb
	```
2. `stratis pool list`，查看pool列表
	```bash
	[root@mickey ~]# stratis pool list
	Name                    Total Physical   Properties
	mypool1   8 GiB / 37.63 MiB / 7.96 GiB      ~Ca,~Cr
	```
3. `stratis pool add-data <pool名稱> <硬碟路徑>`，在pool底下新增硬碟
	```bash
	[root@vb101 ~]# stratis pool add-data mypool1 /dev/sdc
	[root@vb101 ~]# stratis pool list
	Name                    Total Physical   Properties
	mypool1   4 GiB / 41.63 MiB / 3.96 GiB      ~Ca,~Cr
	```
4. `stratis blockdev list [pool名稱]`，查看由哪些硬碟構成(可看指定pool)
	```bash
	[root@vb101 ~]# stratis blockdev list mypool1
	Pool Name   Device Node   Physical Size   Tier
	mypool1     /dev/sdb              2 GiB   Data
	mypool1     /dev/sdc              2 GiB   Data
	```

### 建立file system
1. `stratis filesystem create <pool名稱> <file system名稱>`，在pool中新增file system，默認為xfs
	```bash
	[root@vb101 ~]# stratis filesystem create mypool1 myfilesystem1
	```
2. `stratis filesystem list`，查看file system列表
	```bash
	[root@vb101 ~]# stratis filesystem list
	Pool Name   Name            Used      Created             Device                           UUID
	mypool1     myfilesystem1   546 MiB   Apr 22 2021 22:36   /stratis/mypool1/myfilesystem1   fdeaa564022240d98f91ac9c70f9cedc
	```

### mount
- 暫時mount
	```bash
	[root@vb101 ~]# mount /stratis/mypool1/myfilesystem1 /mnt/mickey
	[root@vb101 ~]# df -h
	Filesystem                                                                                       Size  Used Avail Use% Mounted on
	/dev/mapper/stratis-1-d9d145dc6e3b411c84e65170d7cf6f0f-thin-fs-fdeaa564022240d98f91ac9c70f9cedc  1.0T  7.2G 1017G   1% /mnt/mickey
	```
- 配置文件/etc/fstab新增後執行`mount -a`
	`x-systemd.requires=stratisd.service`代表此設備需要等stratisd.service啟動後才能mount
	```bash
	/stratis/mypool1/myfilesystem1                  /mnt/mickey     xfs     defaults,x-systemd.requires=stratisd.service        0       0
	```

## 🐧建立快照
1. `stratis filesystem snapshot <pool名稱> <filesystem名稱> <快照名稱>`，在指定file system建立snapshot(快照)
	```bash
	[root@vb101 ~]# stratis filesystem snapshot mypool1 myfilesystem1 filesystem1-snap
	[root@vb101 ~]# stratis filesystem list
	Pool Name   Name               Used      Created             Device                              UUID
	mypool1     myfilesystem1      546 MiB   Apr 22 2021 22:36   /stratis/mypool1/myfilesystem1      fdeaa564022240d98f91ac9c70f9cedc
	mypool1     filesystem1-snap   546 MiB   Apr 22 2021 23:48   /stratis/mypool1/filesystem1-snap   aacb8ba047354e4c980e1076b47aab9d
	```
2. mount
	可以配置/etc/fstab來做自動mount，以下省略
	```bash	
	[root@vb101 ~]# mount /stratis/mypool1/mysnap1 /mnt/mickey.snap/
	```

## 🐧刪除流程
1. umount
	記得配置/etc/fstab，`mount -a`
	```bash
	[root@vb101 ~]# umount /stratis/mypool1/myfilesystem1
	[root@vb101 ~]# rm -rf /stratis/mypool1/myfilesystem1
	```
2. `stratis filesystem destroy <pool名稱> <file system名稱>`，刪除file system
	```bash
	[root@vb101 ~]# stratis filesystem destroy mypool1 myfilesystem1
	[root@vb101 ~]# stratis filesystem list
	Pool Name   Name   Used   Created   Device   UUID
	```
3. `stratis pool destroy <pool名稱>`，刪除pool
	```bash
	[root@vb101 ~]# stratis pool destroy mypool1
	[root@vb101 ~]# stratis pool list
	Name   Total Physical   Properties
	```

---
# VDO
- VDO(Vitual Data Optimizer)為kenel lavel的功能，主要在做資料壓縮(kvdo)和精簡重複資料(uds)，可結合LVM使用，因此適合於雲端環境使用
	![Linux_RH134_06_硬碟分區管理分區管理_07_VDO資料處理過程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_07_VDO%E8%B3%87%E6%96%99%E8%99%95%E7%90%86%E9%81%8E%E7%A8%8B.png?raw=true)
- `yum list installed vdo`

## 🐧操作步驟
- `vdo create --name=<vdo名稱> --device=<設備路徑> --vdoLogicalSize=<容量>`，建立vdo
	注意：vdo剛開始需要的硬碟空間會比較大
	1. `--name`，指定vdo名稱
	2. `--device`，指定硬碟(需要分區但不需要mount)
	3. `--vdoLogicalSize`，對外宣布有50G，但可能實際大小只有5G
	4. `--force`
	```bash
	[root@vb101 ~]# vdo create --name=vdotest --device=/dev/sdb --vdoLogicalSize=8590M --force
	Creating VDO vdotest
	Starting VDO vdotest
	Starting compression on VDO vdotest
	VDO instance 0 volume is ready at /dev/mapper/vdotest
	```
- `vdo list`，查看vdo列表
	```bash
	[root@vb101 ~]# vdo list
	vdotest
	```
- `vdo status --name=<vdo名稱>`，查看指定vdo詳細信息
	```bash
	[root@vb101 ~]# vdo status --name=vdotest | head -n 5
	VDO status:
	  Date: '2021-04-23 04:09:57-04:00'
	  Node: vb101
	Kernel module:
	  Loaded: true
	```
	1. Compression：壓縮功能
	2. Deduplication：挑重覆功能
	```bash
	[root@vb101 ~]# vdo status --name=vdotest | grep -E "Deduplication|Compression"
		Compression: enabled
		Deduplication: enabled
	```
- `udevadm settle`，驗證vdo是否被建立
- `mkfs.xfs -K <設備路徑>`，建立file system
	```bash
	[root@vb101 ~]# lsblk -fp
	NAME                   FSTYPE LABEL UUID                                   MOUNTPOINT
	/dev/sdb               vdo          40633048-2237-41fe-9d18-57d9b1ff267c
	└─/dev/mapper/vdotest
	[root@vb101 ~]# mkfs.xfs -K /dev/mapper/vdotest
	meta-data=/dev/mapper/vdotest    isize=512    agcount=4, agsize=549760 blks
			 =                       sectsz=4096  attr=2, projid32bit=1
			 =                       crc=1        finobt=1, sparse=1, rmapbt=0
			 =                       reflink=1
	data     =                       bsize=4096   blocks=2199040, imaxpct=25
			 =                       sunit=0      swidth=0 blks
	naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
	log      =internal log           bsize=4096   blocks=2560, version=2
			 =                       sectsz=4096  sunit=1 blks, lazy-count=1
	realtime =none                   extsz=4096   blocks=0, rtextents=0
	```
- mount，省略/etc/fstab配置
	新mount的vdo會先使用一部分的空間，那些空間是vdo的預留空間
	```bash
	[root@vb101 ~]# mount /dev/mapper/vdotest /mnt/mickey/
	[root@vb101 ~]# mount | grep vdo
	/dev/mapper/vdotest on /mnt/mickey type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
	[root@vb101 ~]# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	/dev/mapper/vdotest    8.4G   93M  8.3G   2% /mnt/mickey	
	```
- 配置/etc/fstab，自動mount
	`x-systemd.requires=vdo.service`代表此設備需要等vdo.service啟動後才能mount
	```bash
	/dev/mapper/vdotest      /mnt/mickey/     xfs     defaults,x-systemd.requires=vdo.service        0       0
	```
- `vdostats --human-readable`，查看vdo使用狀況
	1. `--human-readable`，儲存空間顯示比較人性化
	```bash
	[root@vb101 ~]# vdostats --human-readable
	Device                    Size      Used Available Use% Space saving%
	/dev/mapper/vdotest       8.0G      4.0G      4.0G  50%           98%
	```
- 測試：複製多個容量較大的檔案
	1. `df -h`顯示的容量是給外面看的容量(使用空間會疊加)
	2. `vdostats`顯示真正使用的容量(重覆文件空間不會疊加)
	```bash
	[root@vb101 ~]# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	/dev/mapper/vdotest    8.4G  121M  8.3G   2% /mnt/mickey
	[root@vb101 ~]# vdostats --human-readable
	Device                    Size      Used Available Use% Space saving%
	/dev/mapper/vdotest       8.0G      4.1G      3.9G  51%           98%
	[root@vb101 ~]# cp /mnt/mickey/etc.tar /mnt/mickey/etc2.tar
	[root@vb101 ~]# cp /mnt/mickey/etc.tar /mnt/mickey/etc3.tar
	[root@vb101 ~]# cp /mnt/mickey/etc.tar /mnt/mickey/etc4.tar
	[root@vb101 ~]# cp /mnt/mickey/etc.tar /mnt/mickey/etc5.tar
	[root@vb101 ~]# df -h
	Filesystem             Size  Used Avail Use% Mounted on
	/dev/mapper/vdotest    8.4G  229M  8.2G   3% /mnt/mickey
	[root@vb101 ~]# vdostats --human-readable
	Device                    Size      Used Available Use% Space saving%
	/dev/mapper/vdotest       8.0G      4.1G      3.9G  50%           99%
	```

# NFS Client
- NFS(Network file System)相當於Linux的網絡上的芳鄰
- Red Hat Enterprise Linux 8 NFS版本為4.2，但也支援NFSv4和NFSv3
	1. NFSv3可支援TCP和UDP協議連線
	2. NFSv4僅支援TCP協議連線
	3. NFSv3沒有固定的服務port，是由rpcbind指定port給NFS Server，因此rpcbind和NFS Server必須要一起開啟/關閉
	4. NFSv4下的NFS Server使用的port固定為2049
- NFSv3 mount
	1. `showmount -e <NFS server IP|host>`，查看NFS server分享的資料夾
		```console
		[mickey@vm101 ~]$ showmount -e 192.168.56.102
		Export list for 192.168.56.102:
		/nfs 192.168.56.101
		```
	2. `mount -t nfs <NFS server IP|host>:<NFS serever資料夾路徑> <本地路徑>`，將NFS serever的資料夾掛載至指定本地資料夾
		```console
		[mickey@vm101 ~]$ sudo mount -t nfs 192.168.56.102:/nfs /mnt/nfs
		[mickey@vm101 ~]$ ll /mnt/nfs
		total 0
		-rw-rw-rw-. 1 root root 0 Mar 29 14:46 hello.txt
		```
- NFSv4 mount
	1. `mount -t nfs <NFS server IP|host>:/ <本地路徑>`，直接將NFS server所有公開的資料夾掛載至本地資料夾
- 配置/etc/fstab
	```bash
	foundation0:/share      /mnt/nfs/     nfs     rw,soft,sync      0       0
	```

---
# mount方式
在Linux mount的方法有三種：
1. 使用`mount`暫時掛載，開機後失敗，參考：[[🐧Linux_RH124_18_儲存裝置管理]]
2. 配置/etc/fstab，使每次開機自動mount
3. 使用**autofs**或**systemd.automount**工具自動mount

## 🐧配置/etc/fstab
```bash
foundation0:/share      /mnt/nfs/     nfs     rw,soft,sync      0       0
```
第四個欄位(`rw,soft,sync`)表示mount帶的參數
1. `rw`，可讀可寫，`defaults`自帶參數
2. `soft`，與NFS Server連三次，連不上就不連了
	默認為`hard`，一直連到連上為此
3. `sync`和`async`與資料傳輸相關參數
	`sync`：kenel直接將資料寫入Disk，效能不好，風險較小
	`async`：kenel通過RAM將資料寫入Disk，效能好，風險較大
	![Linux_RH134_06_硬碟分區管理分區管理_08_sync與async區別](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%A1%AC%E7%A2%9F%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86%E5%88%86%E5%8D%80%E7%AE%A1%E7%90%86_08_sync%E8%88%87async%E5%8D%80%E5%88%A5.png?raw=true)

## 🐧插件自動掛載 autofs
- 主要用於NFS auto mount，但所有可以mount的設備都可以使用
- 安裝套件：`yum install autofs`

### direct map操作步驟
1. 確認是否可以mount
	```bash
	[root@mickey ~]# mount /dev/rhel/testlvm /mnt/mickey
	[root@mickey ~]# umount /dev/rhel/testlvm
	```
2. 配置/etc/auto.master.d/\*.autofs，設置工作路徑
	- `--timeout=60`，60s後無使用則umount
	```bash	
	[root@mickey ~]# vi /etc/auto.master.d/direct.autofs
	# local mount point		file path		options
	/mnt/nfs				/etc/testauto	--timeout=60
	```
3. 建立/etc/auto.master.d/\*.autofs中所指定的工作路徑(/etc/auto.direct)
	```bash
	[root@mickey ~]# vi /etc/testauto
	# mount point           options         locations
	*                       -rw,sync,soft   192.168.56.102:/&
	```
4. 重啟autofs.service
	工作路徑中的mount point會自動建立
	```bash
	[root@mickey /]# systemctl enable --now autofs
	[root@mickey /]# systemctl reboot
	```
5. 測試
	```bash
	[mickey@mickey ~]$ watch -n 1 "df -h"
	[mickey@mickey ~]$ ls -l /external
	```

### indirect map操作步驟
會自動載取要mount的目錄
1. 配置/etc/auto.master.d/\*.autofs，設置工作路徑
	```bash	
	[root@mickey ~]# vi /etc/auto.master.d/indirect.autofs
	# mount point   map name                options
	/internal       /etc/auto.indirect      --timeout=60
	```
2. 建立/etc/auto.master.d/\*.autofs中所指定的工作路徑(/etc/auto.indirect)
	`*`代表變數，使用時會改為使用的目錄名，並代入`&`
	```bash
	[root@mickey ~]# vi /etc/auto.direct
	# mount point     options                 location
	*                 -rw,sync,fstype=ext4    /dev/rhel/testlvm/&
	```
3. 重啟autofs.service
	工作路徑中的mount point會自動建立
	```bash
	[root@mickey /]# systemctl enable --now autofs
	[root@mickey /]# systemctl reboot
	```
4. 測試
	```bash
	[mickey@mickey ~]$ watch -n 1 "df -h"
	[mickey@mickey ~]$ ls -l /external
	```