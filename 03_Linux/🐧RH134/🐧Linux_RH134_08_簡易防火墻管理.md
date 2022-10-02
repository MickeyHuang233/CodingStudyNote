# Linux_RH134_08_簡易防火墻管理
- 防火墻的價值在於連線檢測，而不是單純的開關port而已，因此以下內容只是簡易的防火墻管理
- 防火墻底層是通過iptables或nft進行管理，主要看電腦的防火墻是哪種類型的(netfilter或nftable)
	![Linux_RH134_08_簡易防火墻管理_01_防火墻管理工具](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_08_%E7%B0%A1%E6%98%93%E9%98%B2%E7%81%AB%E5%A2%BB%E7%AE%A1%E7%90%86_01_%E9%98%B2%E7%81%AB%E5%A2%BB%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7.png?raw=true)
- zone相當於防火墻的劇本，符合條件的request封包會按照裡面的設置關開port
	1. Runable，修改記憶體中zone設置，立即生效但重啟後失效
	2. Permanent，修改硬碟中資料庫zone設置，不會立即生效，重啟生效或需要"Reload Firewalld"才會生效

## 🐧Web Console管理
1. 安裝web console管理系統
	```bash
	[root@vm104 ~]# yum install cockpit
	[root@vm104 ~]# systemctl enable --now cockpit.socket
	Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/systemd/system/cockpit.socket.
	[root@vm104 ~]# firewall-cmd --add-service=cockpit --permanent
	Warning: ALREADY_ENABLED: cockpit
	success
	[root@vm104 ~]# firewall-cmd --reload
	success
	```
2. web console管理網址：https://servername:9090
3. 防方墻管理入口：網路作業-->防火墻
	![Linux_RH134_08_簡易防火墻管理_02_管理入口](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_08_%E7%B0%A1%E6%98%93%E9%98%B2%E7%81%AB%E5%A2%BB%E7%AE%A1%E7%90%86_02_%E7%AE%A1%E7%90%86%E5%85%A5%E5%8F%A3.png?raw=true)

## 🐧firewall-cmd
### default zone
- `firewall-cmd --get-zones`，查看所有zone名稱
	```bash
	[mickey@vb101 ~]$ firewall-cmd --get-zones
	block dmz drop external home internal libvirt public trusted work
	```
- `firewall-cmd --get-default-zone`，取得默認zone
	```bash
	[mickey@vb101 ~]$ firewall-cmd --get-default-zone
	public
	```
- `firewall-cmd --set-default-zone=<zone名稱>`，設置默認zone
	```bash
	[root@vb101 ~]# firewall-cmd --set-default-zone=work
	success
	[root@vb101 ~]# firewall-cmd --get-default-zone
	work
	```

### 查看zone的使用規則
- `firewall-cmd --get-active-zones`，查看zone所應用的網卡
	```bash
	[root@vb101 ~]# firewall-cmd --get-active-zones
	libvirt
	  interfaces: virbr0
	public
	  interfaces: enp0s10 enp0s8
	```
- `firewall-cmd {--zone=<zone名稱>} --list-all`，若無指定zone則查看全部
	```bash
	[root@vb101 ~]# firewall-cmd --list-all
	public (active)
	  target: default
	  icmp-block-inversion: no
	  interfaces: enp0s10 enp0s8 enp0s9
	  sources: 192.168.123.123
	  services: cockpit dhcpv6-client ssh
	  ports:
	  protocols:
	  masquerade: no
	  forward-ports:
	  source-ports:
	  icmp-blocks:
	  rich rules:
	```

### 設置zone的使用規則
當符合條件時才會(來源IP、接收端網卡)走指定的zone，若都不符合則走default zone
![Linux_RH134_08_簡易防火墻管理_03_zone](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_08_%E7%B0%A1%E6%98%93%E9%98%B2%E7%81%AB%E5%A2%BB%E7%AE%A1%E7%90%86_03_zone.png?raw=true)
- `--permanent`，表示將修改內容寫到硬碟的資料庫中，需要執行`firewall-cmd --reload`才會生效
	沒加此參數代表寫到記憶體中，重新開機就失效
	```bash
	[root@vb101 ~]# firewall-cmd --permanent --add-service=http --zone=public
	success
	[root@vb101 ~]# firewall-cmd --zone=public --list-all
	public (active)
	  target: default
	  icmp-block-inversion: no
	  interfaces: enp0s10 enp0s8
	  sources:
	  services: cockpit dhcpv6-client ssh
	  (以下省略)

	[root@vb101 ~]# firewall-cmd --reload
	success
	[root@vb101 ~]# firewall-cmd --zone=public --list-all
	public (active)
	  target: default
	  icmp-block-inversion: no
	  interfaces: enp0s10 enp0s8
	  sources:
	  services: cockpit dhcpv6-client http ssh
	  (以下省略)
	```
- `firewall-cmd --reload`，將硬碟資料庫的設置讀至記憶體

#### 來源IP比對 -source
- `firewall-cmd --add-source=<來源IP> --zone=<zone名稱>`，新增zone所符合的來源IP
	```bash
	[root@vb101 ~]# firewall-cmd --add-source=192.168.123.123 --zone=public
	success
	```
- `firewall-cmd --remove-source=<來源IP> --zone=<zone名稱>`，刪除zone所符合的來源IP

#### 網卡比對 -interface
- `firewall-cmd --add-interface=<網卡名稱> --zone=<zone名稱>`，刪除zone所符合的網卡
	```bash
	[root@vb101 ~]# firewall-cmd --add-interface=enp0s8 --zone=work
	success
	```
- `firewall-cmd --remove-interface=<網卡名稱> --zone=<zone名稱>`，新增zone所符合的網卡
	```bash
	[root@vb101 ~]# firewall-cmd --remove-interface=enp0s8 --zone=public
	success
	```

#### 允許端口 -port
- `firewall-cmd --add-port=<port/tcp或udp> --zone=<zone名稱>`，**打開**指定zone的指定port
	```bash
	[root@vb101 ~]# firewall-cmd --add-port=80/tcp --zone=public
	success
	```
- `firewall-cmd --remove-port=<port/tcp或udp> --zone=<zone名稱>`，**關閉**指定zone的指定port
	```bash
	[root@vb101 ~]# firewall-cmd --remove-port=80/tcp --zone=public
	success
	```

#### 允許服務 -service
- `firewall-cmd --add-service=<服務名> --zone=<zone名稱>`，**打開**指定zone的指定服務
	```bash
	[root@vb101 ~]# firewall-cmd --add-service=http --zone=public
	success
	```
- `firewall-cmd --remove-service=<服務名> --zone=<zone名稱>`，**關閉**指定zone的指定服務
	```bash
	[root@vb101 ~]# firewall-cmd --remove-service=http --zone=public
	success
	```