#📆2021年 
狀態:: #☑DONE  
完成日期:: 2021-04-25
標籤:: #💻編程/🐳容器技術 #🗂Overview 
子筆記:: 
教程:: 
備註:: 

# Container管理
- container裡面包含一個應用程式，應用程式結束，container就結束了
- docker就是一個container管理工具
- container的中各套件的用途：
	1. Control Groups(cgroups)，限制每一支應用程式的資源使用量
	2. Namespaces，用於區分不同隔間，將信息給不同隔間
	3. SELinux，用於隔離container
	4. Seccomp(Secure Computing mode)，過濾container中的程式不應呼叫denel level的API

## 🐧介紹
### 架構
- 虛擬機架構
	缺點：重覆使用的資源太多(如多個kenel)
	![[Linux_RH134_10_Container管理_01_虛擬機架構.png]]
- container架構：將重覆的東西共用
	一個container一般只執行一個ap-->方便管理，使用podman、lxc管理
	缺點：要一個一個管理太麻煩
	![[Linux_RH134_10_Container管理_02_container架構.png]]
- 由container管理工具(如[[🐳Docker_00_Overview]])管理的container架構
	![[Linux_RH134_10_Container管理_03_container管理工具架構.png]]

### container image
建立container之前需要取得container image
- 一個image需要包含：
	1. System libraries
	2. Programming language runtimes
	3. Programming language libraries
	4. Static data files
- 取得container image方式
	1. 使用工具製作，如buildah
	2. 從registy server下載別人已打包好的image
		1. registry.redhat.io，主要是Red Hat原生image
		2. registry.connect.redhat.com，主要是廠商提供的image
		3. registry.access.redhat.com，redhat registry server，漸漸會被上面兩項替代
- 管理container工具
	1. 管理單一container工具：podman、lxc
	2. 管理多個container工具：docker、Kubernetes

## 🐧podman
![[Linux_RH134_10_Container管理_03_podman.png]]
- `yum install container-tools`，套件安裝

### registry server image
- registry server位置的設定檔路徑
	1. global：/etc/containers/registries.conf
	2. local：~/.config/containers/\*.conf
- container image命名原則：registry_name/user_name/image_name:tag
	1. tag，一般為版本號
- 用於設置container registry server位置
	```bash
	# HTTPS協定-->安全的
	[registries.search]
	registries = ['registry.redhat.io', 'registry.access.redhat.com', 'quay.io', 'docker.io']

	# HTTP-->不安全的
	[registries.insecure]
	registries = []

	# Docker only
	[registries.block]
	registries = []
	```
- `podman info`，查看設定檔參數
	```bash
	[mickey@vb101 ~]$ podman info
	(省略)
	registries:
	  blocked: null
	  insecure: null
	  search:
	  - registry.redhat.io
	  - registry.access.redhat.com
	  - quay.io
	  - docker.io
	(省略)
	```
- `podman search <registry server>/<關鍵字>`，在指定registry server找指定關鍵字的image
	1. `--no-trunc`，顯示詳細信息
	2. `--limit <數量>`，只列前幾筆查詢結果
	3. `--filter <條件>`，根據條件查詢
		- `stars=<數量>`，評價星級
		- `is-automated=<true|false>`，是否自動布署(結合git hub)
		- `is-official=<true|false>`，是否為官方image
	```bash
	[mickey@vb101 ~]$ podman search --limit 3 --filter is-official=false registry.redhat.io/rhel8
	INDEX       NAME                                         DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
	redhat.io   registry.redhat.io/openj9/openj9-8-rhel8     OpenJ9 1.8 OpenShift S2I image for Java Appl...   0
	redhat.io   registry.redhat.io/openjdk/openjdk-8-rhel8   OpenJDK 1.8 Image for Java Applications base...   0
	redhat.io   registry.redhat.io/openj9/openj9-11-rhel8    OpenJ9 11 OpenShift S2I image for Java Appli...   0
	```

### podman login
```bash
[mickey@vm101 redis_sentinel1]$ podman login
Username: redhat
Password:
Login Succeeded!
```

### local image
#### 操作image
- `podman images`，查看已下載的image
	```bash
	[root@vb101 ~]# podman images
	REPOSITORY                            TAG      IMAGE ID       CREATED       SIZE
	registry.access.redhat.com/ubi8/ubi   latest   613e5da7a934   2 weeks ago   213 MB
	```
- `podman pull <image名>`，下載image
	```bash
	[root@vb101 ~]# podman pull registry.access.redhat.com/ubi8/ubi:latest
	Trying to pull registry.access.redhat.com/ubi8/ubi:latest...Getting image source signatures
	Copying blob 55eda7743468 done
	Copying blob 4b21dcdd136d done
	Copying config 613e5da7a9 done
	Writing manifest to image destination
	Storing signatures
	613e5da7a934e1963e37ed935917e8be6b8dfd90cac73a724ddc224fbf16da20
	```
- `podman rmi <image名>`，移除image，如果還有container執行此image則無法刪除
	1. `-f`，強制執行
	```bash
	[mickey@vb101 ~]$ podman rmi -f registry.access.redhat.com/ubi8/ubi
	613e5da7a934e1963e37ed935917e8be6b8dfd90cac73a724ddc224fbf16da20
	[mickey@vb101 ~]$ podman images
	REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
	```

#### 查看image詳細信息
- [Red Hat Container目錄](https://access.redhat.com/containers)，Red Hat發行的image
- `skopeo inspect docker://<container名稱>`，查看還沒下載的image內容
	1. `docker://`，表示看registry server的
	2. 注意：需要先下載套件，`yum install skopeo`
	```bash
	[mickey@vb101 ~]$ skopeo inspect docker://registry.redhat.io/openjdk/openjdk-8-rhel8
	```
- `podman inspect <container名稱>`，查看已下載的image內容
	```bash
	[mickey@vb101 ~]$ podman images
	REPOSITORY                            TAG      IMAGE ID       CREATED       SIZE
	registry.access.redhat.com/ubi8/ubi   latest   613e5da7a934   2 weeks ago   213 MB
	[mickey@vb101 ~]$ podman inspect registry.access.redhat.com/ubi8/ubi | head -n 5
	[
		{
			"Id": "613e5da7a934e1963e37ed935917e8be6b8dfd90cac73a724ddc224fbf16da20",
			"Digest": "sha256:70fbfa84a056aa1d4dcc5d45119852887527ae361071a9c4051a17a083aefa06",
			"RepoTags": [
	```

### 操作container
container啟動後的是隔離於實體機、虛擬機和其他container

#### 查看container狀態
`podman ps`，查看正在執行的container狀態
- `-a`，查看所有container(無論是否執行中)
```bash
[mickey@vb101 ~]$ podman ps -a
CONTAINER ID  IMAGE                                       COMMAND       CREATED         STATUS                     PORTS  NAMES
124929e1ecba  registry.access.redhat.com/ubi8/ubi:latest  /bin/bash     10 seconds ago  Exited (0) 10 seconds ago         test1
8040b43c115d  registry.access.redhat.com/ubi8/ubi:latest  --name=test1  32 seconds ago  Created                           distracted_driscoll
```

#### 建立container
`podman run [-it|-d] [--name=<指定container名>] <container名> [指令]`，建立並啟動container
1. `-it`，開啟bash
	```bash
	[root@vb101 ~]# podman run -it registry.access.redhat.com/ubi8/ubi
	[root@e918185cae15 /]#
	```
2. `--name`，給container命名、要給container執行的指令，如果沒有指定，系統則自己指定，不方便管理
	```bash
	[root@vb101 ~]# podman run -it --name=rhel8 registry.access.redhat.com/ubi8/ubi /bin/bash
	[root@de7075ccd36e /]# exit
	(此時container已經關閉，因為已經結束/bin/bash的執行)
	```
3. `-d`，daemon，背景執行
	```bash
	[mickey@vb101 ~]$ podman run -d registry.access.redhat.com/ubi8/ubi
	05fc4c00051f5e77f9349a370b4f0f825e9d21a58b7a62b8a0424ec8d2d35379
	```
4. `--rm`，container執行完成後就會刪除
	```bash
	[mickey@vb101 ~]$ podman run --rm registry.access.redhat.com/ubi8/ubi cat /etc/os-release
	NAME="Red Hat Enterprise Linux"
	VERSION="8.3 (Ootpa)"
	(以下省略)
	```
5. `-p <實體機port>:<container port>`，將實體機port轉移至container port
	port 1~1024只有root才可以用，所以用一般user身份執行container都需要重導request
	```bash
	[mickey@vb101 ~]$ podman run -d --name=test2 -p 8080:8080 registry.access.redhat.com/ubi8/ubi:latest24c8db317e4f48b2e6637c32a616d6a939443a2491f6e76f52ea4a0c31186872
	[mickey@vb101 ~]$ podman ps -a
	CONTAINER ID  IMAGE                                       COMMAND       CREATED         STATUS                     PORTS                   NAMES
	24c8db317e4f  registry.access.redhat.com/ubi8/ubi:latest  /bin/bash     11 seconds ago  Exited (0) 10 seconds ago  0.0.0.0:8080->8080/tcp  test2
	```

#### 操作container
- `podman exec [-it|-d] <container名稱|container ID> <指令>`，在執行中的container再執行指定指令
	1. `-l`，指最後一次操作的container
	```bash
	[mickey@vm104 ~]$ sudo podman exec -it mydb /bin/bash
	bash-4.2$ mysql -u mickey -p --port=3306 --host=127.0.0.1
	Enter password:
	(省略)
	MariaDB [(none)]>
	```
- `podman start <container名稱|container ID>`，啟動container
	```bash
	[mickey@vm104 ~]$ sudo podman ps -a
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED        STATUS                      PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  7 minutes ago  Exited (137) 5 minutes ago  0.0.0.0:3306->3306/tcp  mydb
	[mickey@vm104 ~]$ sudo podman start mydb
	mydb
	[mickey@vm104 ~]$ sudo podman ps -a
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED        STATUS             PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  8 minutes ago  Up 15 seconds ago  0.0.0.0:3306->3306/tcp  mydb
	```
- `podman stop <container名稱|container ID>`，關閉container，container還會存在
	1. `-f`，強制執行
	```bash
	[mickey@vb101 ~]$ podman stop 124929e1ecba
	124929e1ecba8e7abea27ee09d6c86cb6ff74458c2c241dc0813aeff019efcbf
	[mickey@vb101 ~]$ podman stop test1
	124929e1ecba8e7abea27ee09d6c86cb6ff74458c2c241dc0813aeff019efcbf
	```
- `podman kill <container名稱|container ID>`，強制停止container，和`kill -9`類似
	```bash
	[mickey@vm104 ~]$ sudo podman ps
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED         STATUS             PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  35 seconds ago  Up 35 seconds ago  0.0.0.0:3306->3306/tcp  mydb
	[mickey@vm104 ~]$ sudo podman kill mydb
	4b996b53ad3f03d03ebe4fdaf05b74e52967f78d93cb77758702595af36d7589
	[mickey@vm104 ~]$ sudo podman ps
	CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
	```
- `podman rm <container名稱|container ID>`，刪除container
	```bash
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED        STATUS                      PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  4 minutes ago  Exited (137) 2 minutes ago  0.0.0.0:3306->3306/tcp  mydb
	6e77507be47e  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  9 minutes ago  Exited (2) 9 minutes ago    0.0.0.0:3306->3306/tcp  trusting_rosalind
	[mickey@vm104 ~]$ sudo podman rm cranky_cartwright
	c3da3b82a61212143307bf6a61ef769ba4122f7451368f67e2ff2efe0eee46d2
	[mickey@vm104 ~]$ sudo podman ps -a
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED         STATUS                      PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  4 minutes ago   Exited (137) 3 minutes ago  0.0.0.0:3306->3306/tcp  mydb
	```
- `podman logs <container名稱|container ID>`，查看container log

### 執行container傳入參數
- 可以用`podman inspect <image名稱|container名稱|container ID>`查看參數
	```bash
	[mickey@vb101 ~]$ podman inspect registry.access.redhat.com/rhscl/mariadb-102-rhel7
	(省略)
	"summary": "MariaDB 10.2 SQL database server",
				"url": "https://access.redhat.com/containers/#/registry.access.redhat.com/rhscl/mariadb-102-rhel7/images/1-66",
				"usage": "docker run -d -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 rhscl/mariadb-102-rhel7",
	(省略)
	```
- `-e`，啟動container帶入參數，以下為mysql參數
	- MYSQL_USER
	- MYSQL_PASSWORD
	- MYSQL_DATABASE
	- MYSQL_ROOT_PASSWORD，資料庫最高權限管理者密碼，帳號一樣叫root
	```bash
	[mickey@vm104 ~]$ sudo podman run -d -p 3306:3306 --name=mydb \
	> -e MYSQL_USER=mickey -e MYSQL_PASSWORD=mickey -e MYSQL_DATABASE=testdb \
	> registry.access.redhat.com/rhscl/mariadb-102-rhel7
	4b996b53ad3f03d03ebe4fdaf05b74e52967f78d93cb77758702595af36d7589
	[mickey@vm104 ~]$ sudo podman ps
	CONTAINER ID  IMAGE                                                      COMMAND               CREATED        STATUS            PORTS                   NAMES
	4b996b53ad3f  registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest  container-entrypo...  9 seconds ago  Up 9 seconds ago  0.0.0.0:3306->3306/tcp  mydb
	```

### container資料存入硬碟
- container要用的時候才會建立，不需要用的時候會移除，因此container裡面的資料如果要保存，需要做另外的處理
- 執行思路：實體機器的目錄mount至container的目錄
- `-v <實體機目錄>:<container目錄>:Z`，實體機器的目錄mount至container的目錄
	1. `Z`表示使SELinux的type一致，一般都會使用
	```bash
	[mickey@vm104 ~]$ sudo podman images
	REPOSITORY                                           TAG      IMAGE ID       CREATED        SIZE
	registry.access.redhat.com/rhscl/httpd-24-rhel7      latest   4291e206b228   8 days ago     329 MB
	[mickey@vm104 ~]$ sudo podman run -it --name=myweb -p 8080:8080 -v /var/www/html/:/var/www/html/:Z 4291e206b228
	```
- `--privileged=true`，開放實體機器目錄操作root權限

### 開機啟動container
- 執行思路：將container轉交給systemd管理
- systemd有提供user service
	1. root執行的container會一直啟動
	2. 一般情況下，一般用戶執行的container，有被存取時container會啟動；存取結束時container會關閉
	3. 如果一般用戶執行的container需要一直啟動，需要另外設置
- 服務管理可參考：[[🐧Linux_RH124_10_服務管理]]

#### 建立systemd unit file
1. `podman run`，建立container
2. 如果是user container，需要建立~/.config/systemd/user/
3. `podman generate systemd --name <container名稱> --files --new`，自動産出unit file
4. 將産出的unit file放至
	- system service：/etc/systemd/system/unit.service
	- user service：~/.config/systemd/user/unit.service

#### 操作systemd unit file
基本上system service的操作再加上參數`--user`就可操作user service
- system service
	1. `systemctl daemon-reload`，重新讀取unit file
	2. `systemctl start <container名稱>`，啟動service
	3. `systemctl stop <container名稱>`，停止service
	4. `systemctl enable <container名稱>`，設置開機啟用
	5. `systemctl disable <container名稱>`，設置開機不啟用
- user service
	1. `systemctl --user daemon-reload`，重新讀取unit file
	2. `systemctl --user start <container名稱>`，啟動service
	3. `systemctl --user stop <container名稱>`，停止service
	4. `systemctl --user enable <container名稱>`，設置開機啟用
	5. `systemctl --user disable <container名稱>`，設置開機不啟用
	6. 設置user service無資料存取是否啟動<--默認情況下，user service臨資料存取會自動關閉
		- `loginctl enable-linger`，啟動
		- `loginctl disable-linger`，關閉
		- `loginctl show-user <用戶名>`，查看狀態
		```bash
		[mickey@vm104 .config]$ loginctl show-user mickey | grep Linger
		Linger=no
		```

## 🐧lxc
### 前置準備
1. 安裝EPEL(Extra Packages for Enterprise Linux)，用於安裝lxc套件；參考：[EPEL](https://fedoraproject.org/wiki/EPEL)
	```bash
	yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	```
2. `yum list | grep lxc`
3. `yum install lxc.x86_64 lxc-templates.x86_64`
4. 建立網路橋接器，並將網卡接上網路橋接器
	- `nmcli connection add con-name br0 type bridge ifname virbr0 ipv4.address 192.168.56.102/24 ipv4.gateway 192.168.250.254 ipv4.dns 8.8.8.8 ipv4.method manual`，建立網路橋接器
	- `nmcli connection add con-name eth0 type bridge-slave ifname enp0s8 master br0`，將網卡接上網路橋接器
5. `lxc-checkconfig`，確認系統是否滿足容器要求

### 操作container
1. `lxc-create -n CentOS-8 -t download`，建立名為CentOS-8的image，從lxc網站上下載
	- 下載後image位置：/var/lib/lxc/
	- 下載後本機cache：/var/cache/lxc/download/
	```bash
	# 另外指定server
	[mickey@vb101 ~]$ sudo lxc-create -n test -t download -- --keyserver hkp://p80.pool.sks-keyservers.net:80
	(省略)
	Distribution:
	centos
	Release:
	8
	Architecture:
	amd64
	```
2. `lxc-copy -n <被複製container名稱> -N <新container名稱>`，下載過後的container可直接覆製為指定名稱(`-N`)
	```bash
	[root@vb101 ~]# lxc-copy -n test -N test2
	[root@vb101 ~]# lxc-ls -f
	NAME  STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED
	test  STOPPED 0         -      -    -    false
	test2 STOPPED 0         -      -    -    false
	```
4. `lxc-ls -f`，查看當前container狀態
	```bash
	[root@vb101 ~]# lxc-ls -f
	NAME STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED
	test STOPPED 0         -      -    -    false
	```
3. 設置container連線網卡
	- 單個container config：/var/lib/lxc/<container名稱>/config
	- 默認config：/etc/lxc/default.conf
	```bash
	# Network configuration
	lxc.net.0.type = veth
	# lxc.net.0.link = lxcbr0
	lxc.net.0.link = virbr0
	lxc.net.0.flags = up
	lxc.net.0.hwaddr = 00:16:3e:xx:xx:xx
	```
4. `lxc-start -n <container名稱>`，啟動container
	```bash
	[root@vb101 ~]# lxc-start -n test
	[root@vb101 ~]# lxc-ls -f
	NAME STATE   AUTOSTART GROUPS IPV4           IPV6 UNPRIVILEGED
	test RUNNING 0         -      192.168.122.37 -    false
	```
5. `lxc-attach -n <container名稱> passwd`，設置container root密碼，新建的container才需要
	```bash
	[root@vb101 ~]# lxc-attach -n test passwd
	Changing password for user root.
	New password:
	Retype new password:
	passwd: all authentication tokens updated successfully.
	```
6. `lxc-bash -n <container名稱>`，進入指定container bash
	- 資安關系，建議不要sshd.server
	- 離開container操作：按Ctrl + A --> Ctrl、A鍵放開 --> 按Q
	```bash
	[root@vb101 ~]# lxc-bash -n test
	Connected to tty 1
	Type <Ctrl+a q> to exit the bash, <Ctrl+a Ctrl+a> to enter Ctrl+a itself

	CentOS Linux 8
	Kernel 4.18.0-147.el8.x86_64 on an x86_64

	test login: root
	Password:
	[root@test ~]#
	```
7. `lxc-stop -n <container名稱>`，關閉container
	```bash
	[root@vb101 ~]# lxc-stop -n test
	[root@vb101 ~]# lxc-ls -f
	NAME STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED
	test STOPPED 0         -      -    -    false
	```
8.  `lxc-destroy -n <container名稱>`刪除container
	```bash
	[root@vb101 ~]# lxc-destroy -n CentOS=8
	lxc-destroy: CentOS=8: tools/lxc_destroy.c: main: 271 Destroyed container CentOS=8
	```
9. `lxc-top`，查看每個container所占用的資源