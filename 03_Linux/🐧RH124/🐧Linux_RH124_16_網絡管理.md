# Linux_RH124_16_網絡管理
## 🐧簡介
- 網絡相關知識可參考：[[🌐網路架構_00_Overview]]
- VM網絡配置示意圖
	![Linux_RH124_13_網路配置_01](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_13_%E7%B6%B2%E8%B7%AF%E9%85%8D%E7%BD%AE_01.png?raw=true)
	
### 網絡介面命名規則
1. 第一、二個字元代表網絡類型
	- en，ethernet，乙太網路
	- wl，WLAN，無線區域網絡
	- ww，WWAN，無線廣域網絡
2. 後面的字元代表網路設備類型及編號(以下顯示M或N)
	- oN，設備焊在主機板上(onboard)；eno1-->第一個焊在主機板上的乙太網絡設備
	- sN，可熱插拔網絡設備；ens3-->插在3號插槽(slot number)熱插拔乙太網絡設備
	- pMsN，額外插上的網絡設備；wlp4s0-->插在第四個PCI擴充插槽的0號插槽(slot)的無線網路設備
		![PCI device](https://images2017.cnblogs.com/blog/1094457/201709/1094457-20170904094834804-389798155.jpg)

### 網絡介面改回舊的命名規則
1. `vi /etc/default/grub`
2. 在"GRUB_CMDLINE_LINUX=..."加上net.ifnames=0
	```bash
	# 原本設置
	GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
	# 修改後設置
	GRUB_CMDLINE_LINUX="crashkernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet net.ifnames=0"
	```
1. `grub2-mkconfig > /boot/grub2/grub.cfg`
2. 電腦重啟

### IP介紹
- 參考：[[🌐網路架構_02_TCP IP]]
- 取得IP的方法：
	1. 手動指定IP
	2. 自動取得IP

## 🐧查看網絡配置 
- openwrt
- openswitch，軟件的switch

### 查看網卡、路由信息 ip
#### 實體層的網卡資訊 ip link
- `ip link show [網卡名稱]`，查看實體層的網卡資訊(也可查看指定網卡)
	1. state UP mode，代表網卡啟動；state DOWN mode，代表網卡關閉
	2. link/ether，代表MAC地址
	```bash
	[mickey@localhost ~]$ ip link show
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
		link/ether 08:00:27:0c:17:a0 brd ff:ff:ff:ff:ff:ff
	3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
		link/ether 08:00:27:7b:99:57 brd ff:ff:ff:ff:ff:ff
	```
- `ip link set <網卡名稱> <up|down>`, 打開/關閉網卡
	```bash
	[mickey@localhost ~]$ ip link show enp0s3
	1: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
		link/ether 08:00:27:0c:17:a0 brd ff:ff:ff:ff:ff:ff
	[root@localhost ~]# ip link set enp0s3 down
	[root@localhost ~]# ip link show enp0s3
	2: enp0s3: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
		link/ether 08:00:27:0c:17:a0 brd ff:ff:ff:ff:ff:ff
	```
- `ip -s link show [網卡名稱]`，查看網卡接收、發送封包數(也可查看指定網卡)
	1. RX，received，已接收封包數
	2. TX，transmitted，已傳送封包數
	```bash
	[root@localhost ~]# ip -s link show enp0s3
	2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
		link/ether 08:00:27:0c:17:a0 brd ff:ff:ff:ff:ff:ff
		RX: bytes  packets  errors  dropped overrun mcast
		0          0        0       0       0       0
		TX: bytes  packets  errors  dropped carrier collsns
		882        8        0       0       0       0
	```

#### 邏輯層的網卡資訊 ip addr
- `ip address show [網卡名稱]`，查看邏輯層的網卡資訊(也可查看指定網卡)
	1. UP，表示網卡狀態為啟動
	2. link/ether：MAC地址
	3. inet：IPV4
	4. inet6：IPV6
	```bash
	[root@localhost ~]# ip addr show enp0s10
	1: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5f:82:e8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s10
       valid_lft 303sec preferred_lft 303sec
    inet6 fe80::54b1:8386:d40a:729d/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
	```
- `ip addr add <IP> dev <網卡名稱>`，設置指定IP，暫時性，開機後無效
	```bash
	[root@localhost ~]# ip addr show enp0s10
	5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
		link/ether 08:00:27:5f:82:e8 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s10
		   valid_lft 377sec preferred_lft 377sec
		inet6 fe80::54b1:8386:d40a:729d/64 scope link noprefixroute
		   valid_lft forever preferred_lft forever
	[root@localhost ~]# ip addr add 192.168.10.10 dev enp0s10
	[root@localhost ~]# ip addr show enp0s10
	5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
		link/ether 08:00:27:5f:82:e8 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s10
		   valid_lft 305sec preferred_lft 305sec
		inet 192.168.10.10/32 scope global enp0s10
		   valid_lft forever preferred_lft forever
		inet6 fe80::54b1:8386:d40a:729d/64 scope link noprefixroute
		   valid_lft forever preferred_lft forever
	```
- `ip addr del <ip> dev <網卡名稱>`，取消指定IP，暫時性，開機後無效
	```bash
	[root@localhost ~]# ip addr show enp0s10
	5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
		link/ether 08:00:27:5f:82:e8 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s10
		   valid_lft 481sec preferred_lft 481sec
		inet 192.168.10.10/32 scope global enp0s10
		   valid_lft forever preferred_lft forever
		inet6 fe80::54b1:8386:d40a:729d/64 scope link noprefixroute
		   valid_lft forever preferred_lft forever
	[root@localhost ~]# ip addr del 192.168.10.10 dev enp0s10
	Warning: Executing wildcard deletion to stay compatible with old scripts.
			 Explicitly specify the prefix length (192.168.10.10/32) to avoid this warning.
			 This special behaviour is likely to disappear in further releases,
			 fix your scripts!
	[root@localhost ~]# ip addr show enp0s10
	5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
		link/ether 08:00:27:5f:82:e8 brd ff:ff:ff:ff:ff:ff
		inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s10
		   valid_lft 434sec preferred_lft 434sec
		inet6 fe80::54b1:8386:d40a:729d/64 scope link noprefixroute
		   valid_lft forever preferred_lft forever
	```

#### 路由表資訊 ip route
- `ip route show`，查看IPV4路由表資訊
	```bash
	[root@localhost ~]# ip route show
	192.168.10.10 dev enp0s10 proto kernel scope link src 192.168.10.10 metric 100
	192.168.56.0/24 dev enp0s10 proto kernel scope link src 192.168.56.102 metric 100
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
	```
- `ip -6 route show`，查看IPV6路由表資訊
	```bash
	[root@localhost ~]# ip -6 rout show
	::1 dev lo proto kernel metric 256 pref medium
	fe80::/64 dev enp0s10 proto kernel metric 100 pref medium
	```
- `ip route add <IP> via <IP>`，新增路由表，暫時性，開機後無效
	```bash
	[root@localhost ~]# ip route show
	192.168.10.10 dev enp0s10 proto kernel scope link src 192.168.10.10 metric 100
	192.168.56.0/24 dev enp0s10 proto kernel scope link src 192.168.56.102 metric 100
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
	[root@localhost ~]# ip route add 192.168.20.20 via 192.168.20.20
	[root@localhost ~]# ip route show
	192.168.10.10 dev enp0s10 proto kernel scope link src 192.168.10.10 metric 100
	192.168.20.20 via 192.168.20.20 dev enp0s3
	192.168.56.0/24 dev enp0s10 proto kernel scope link src 192.168.56.102 metric 100
	192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
	```

### 確認連線 ping
- `ping -c <次數> <IPV4地址>`，ping IPV4指定次數(沒指定則一直ping)
	```bash
	[root@localhost ~]# ping -c 3 192.168.20.20
	PING 192.168.20.20 (192.168.20.20) 56(84) bytes of data.
	64 bytes from 192.168.20.20: icmp_seq=1 ttl=64 time=0.031 ms
	64 bytes from 192.168.20.20: icmp_seq=2 ttl=64 time=0.047 ms
	64 bytes from 192.168.20.20: icmp_seq=3 ttl=64 time=0.044 ms

	--- 192.168.20.20 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 80ms
	rtt min/avg/max/mdev = 0.031/0.040/0.047/0.010 ms
	```
- `ping6 -c <次數> <IPV6地址>`，ping IPV6指定次數(沒指定則一直ping)
	```bash
	[root@localhost ~]# ping6 -c 3 fe80::54b1:8386:d40a:729d
	PING fe80==54b1:8386:d40a:729d(fe80==54b1:8386:d40a:729d) 56 data bytes
	64 bytes from fe80::54b1:8386:d40a:729d%enp0s10: icmp_seq=1 ttl=64 time=0.041 ms
	64 bytes from fe80::54b1:8386:d40a:729d%enp0s10: icmp_seq=2 ttl=64 time=0.043 ms
	64 bytes from fe80::54b1:8386:d40a:729d%enp0s10: icmp_seq=3 ttl=64 time=0.167 ms

	--- fe80::54b1:8386:d40a:729d ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 38ms
	rtt min/avg/max/mdev = 0.041/0.083/0.167/0.059 ms
	```

### 路由追蹤
看本機和指定IP要經過幾個router
- `mtr <IP>`，動態追蹤每個router的速度
	```bash
	[mickey@localhost ~]$ mtr 216.58.200.228
												   My traceroute  [v0.92]
	localhost.localdomain (10.0.2.15)                                                           2021-04-03T23:14:13+0800
	Keys:  Help   Display mode   Restart statistics   Order of fields   quit
																				Packets               Pings
	 Host                                                                     Loss%   Snt   Last   Avg  Best  Wrst StDev
	 1. _gateway                                                               0.0%    11    0.5   0.5   0.3   0.7   0.1
	 2. 192.168.0.1                                                            0.0%    11    2.6   8.4   1.9  23.9   8.2
	 3. 172.16.0.254                                                           0.0%    11    6.3   7.7   1.8  39.0  10.8
	 4. h254.s98.ts.hinet.net                                                  0.0%    11   16.6  38.8   4.4 240.6  71.0
	 5. tpe4-3301.hinet.net                                                    0.0%    11    6.4  34.1   4.3 157.0  50.0
	 6. ???
	 7. 220-128-12-189.HINET-IP.hinet.net                                      0.0%    11    4.7  13.7   4.7  53.7  13.9
	 8. 72.14.202.178                                                          0.0%    11   36.0  15.0   7.4  36.0   9.2
	 9. 172.253.64.123                                                         0.0%    11   22.5  16.4   5.1  46.7  11.3
	10. 72.14.237.231                                                          0.0%    10    6.4  11.8   4.5  44.4  12.2
	11. tsa03s01-in-f228.1e100.net                                             0.0%    10    7.3  20.7   5.4 126.4  37.3
	```
- `tracepath -n <IP|hostname>`，IPV4；`tracepath6`為IPV6
	1. -n，打印IP地址
	```bash
	[root@localhost yum.repos.d]# tracepath -n www.google.com
	 1?: [LOCALHOST]                      pmtu 1500
	 1:  10.0.2.2                                              0.508ms
	 1:  10.0.2.2                                              0.329ms
	 2:  no reply
	 3:  no reply
	 4:  10.0.2.2                                              0.597ms !N
		 Resume: pmtu 1500
	```
- `traceroute <IP|hostname>`，用法和`tracepath`一樣，但`traceroute`不是自帶的套件
	1. `-6`，IPV6；默認為IPV4
	2. 默認是使用UDP方式打包封包
	3. `-I`，以ICMP方式打包封包
	4. `-T`，以TCP方式打包封包

### 查看Socket信息
- `ss`，比較推薦使用
	1. `-n`，顯示IP位址
	2. `-t`，顯示TCP Socket連線
	3. `-u`，顯示UDP Socket連線
	4. `-l`，只顯示正在監聽的Socket連線
	5. `-a`，顯示所有(啟用/沒啟用)的Socket連線
	6. `-4`，顯示IPV4
	7. `-6`，顯示IPV6
	8. `-p`，顯示進程PID
	```bash
	[root@localhost yum.repos.d]# ss -ta
	State        Recv-Q        Send-Q                  Local Address:Port                   Peer Address:Port
	LISTEN       0             128                           0.0.0.0:sunrpc                      0.0.0.0:*
	LISTEN       0             32                      192.168.122.1:domain                      0.0.0.0:*
	LISTEN       0             128                           0.0.0.0:ssh                         0.0.0.0:*
	LISTEN       0             5                           127.0.0.1:ipp                         0.0.0.0:*
	ESTAB        0             64                     192.168.56.104:ssh                    192.168.56.1:idig_mux
	LISTEN       0             128                              [==]:sunrpc                         [==]:*
	LISTEN       0             128                              [==]:ssh                            [==]:*
	LISTEN       0             5                               [==1]:ipp                            [==]:*
	```
- `netstat`
	```bash
	[root@localhost yum.repos.d]# netstat
	Active Internet connections (w/o servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State
	tcp        0     64 localhost.localdoma:ssh 192.168.56.1:idig_mux   ESTABLISHED
	Active UNIX domain sockets (w/o servers)
	Proto RefCnt Flags       Type       State         I-Node   Path
	unix  2      [ ]         DGRAM                    30516    /run/user/42/systemd/notify
	unix  2      [ ]         DGRAM                    22870    /var/run/chrony/chronyd.sock
	unix  3      [ ]         DGRAM                    11172    /run/systemd/notify
	unix  2      [ ]         DGRAM                    11174    /run/systemd/cgroups-agent
	unix  8      [ ]         DGRAM                    11194    /run/systemd/journal/socket
	unix  21     [ ]         DGRAM                    11213    /run/systemd/journal/dev-log
	unix  2      [ ]         DGRAM                    51710    /run/user/1000/systemd/notify
	```

## 🐧網絡設置 nmcli
- 網絡設定主要通過兩個方面設置：
	1. device，網路卡
	2. connection，設定檔

### 查看網路主態
- `nmcli device status`，從網卡角度看網絡主態
	`nmcli device show [網卡名稱]`，查看網卡詳細信息(可指定網卡)
	```bash
	[root@localhost yum.repos.d]# nmcli device status
	DEVICE      TYPE      STATE      CONNECTION
	enp0s3      ethernet  connected  enp0s3
	enp0s8      ethernet  connected  Wired connection 2
	virbr0      bridge    connected  virbr0
	lo          loopback  unmanaged  --
	virbr0-nic  tun       unmanaged  --
	```
- `nmcli connection show`，從設定檔角度看網絡主態
	1. `--active`，只顯示啟用中
	```bash
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              4cb86086-9bbd-40c1-839b-235f290e3425  bridge    virbr0
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	```
- `nmcli connection show <設定檔名>`，查看指定設定檔詳細信息
	1. connection.autoconnect，電腦啟動後，是否自動啟動此網卡
	2. ipv4.method，auto-->動態IP；manual-->固定IP
	3. ipv4.dns
	4. ipv4.addresses
	5. ipv4.gateway
	6. ipv6.method
	7. ipv6.dns
	8. ipv6.addresses
	9. ipv6.gateway
	```bash
	[root@localhost yum.repos.d]# nmcli connection show enp0s3
	connection.id:                          enp0s3
	connection.uuid:                        5d864ee0-68de-46c9-87e1-52f27d329d42
	connection.stable-id:                   --
	connection.type:                        802-3-ethernet
	connection.interface-name:              enp0s3
	connection.autoconnect:                 yes
	connection.autoconnect-priority:        0
	connection.autoconnect-retries:         -1 (default)
	connection.multi-connect:               0 (default)
	connection.auth-retries:                -1
	connection.timestamp:                   1617471596
	connection.read-only:                   no
	connection.permissions:                 --
	connection.zone:                        --
	connection.master:                      --
	connection.slave-type:                  --
	connection.autoconnect-slaves:          -1 (default)
	connection.secondaries:                 --
	connection.gateway-ping-timeout:        0
	connection.metered:                     unknown
	connection.lldp:                        default
	connection.mdns:                        -1 (default)
	connection.llmnr:                       -1 (default)
	connection.wait-device-timeout:         -1
	802-3-ethernet.port:                    --
	802-3-ethernet.speed:                   0
	802-3-ethernet.duplex:                  --
	802-3-ethernet.auto-negotiate:          no
	802-3-ethernet.mac-address:             --
	802-3-ethernet.cloned-mac-address:      --
	802-3-ethernet.generate-mac-address-mask:--
	802-3-ethernet.mac-address-blacklist:   --
	802-3-ethernet.mtu:                     auto
	802-3-ethernet.s390-subchannels:        --
	802-3-ethernet.s390-nettype:            --
	802-3-ethernet.s390-options:            --
	802-3-ethernet.wake-on-lan:             default
	802-3-ethernet.wake-on-lan-password:    --
	ipv4.method:                            auto
	ipv4.dns:                               --
	ipv4.dns-search:                        --
	ipv4.dns-options:                       --
	ipv4.dns-priority:                      0
	ipv4.addresses:                         --
	ipv4.gateway:                           --
	ipv4.routes:                            --
	ipv4.route-metric:                      -1
	ipv4.route-table:                       0 (unspec)
	ipv4.routing-rules:                     --
	ipv4.ignore-auto-routes:                no
	ipv4.ignore-auto-dns:                   no
	ipv4.dhcp-client-id:                    --
	ipv4.dhcp-timeout:                      0 (default)
	ipv4.dhcp-send-hostname:                yes
	ipv4.dhcp-hostname:                     --
	ipv4.dhcp-fqdn:                         --
	ipv4.never-default:                     no
	ipv4.may-fail:                          yes
	ipv4.dad-timeout:                       -1 (default)
	ipv6.method:                            auto
	ipv6.dns:                               --
	ipv6.dns-search:                        --
	ipv6.dns-options:                       --
	ipv6.dns-priority:                      0
	ipv6.addresses:                         --
	ipv6.gateway:                           --
	ipv6.routes:                            --
	ipv6.route-metric:                      -1
	ipv6.route-table:                       0 (unspec)
	ipv6.routing-rules:                     --
	ipv6.ignore-auto-routes:                no
	ipv6.ignore-auto-dns:                   no
	ipv6.never-default:                     no
	ipv6.may-fail:                          yes
	ipv6.ip6-privacy:                       -1 (unknown)
	ipv6.addr-gen-mode:                     stable-privacy
	ipv6.dhcp-duid:                         --
	ipv6.dhcp-send-hostname:                yes
	ipv6.dhcp-hostname:                     --
	ipv6.token:                             --
	proxy.method:                           none
	proxy.browser-only:                     no
	proxy.pac-url:                          --
	proxy.pac-script:                       --
	GENERAL.NAME:                           enp0s3
	GENERAL.UUID:                           5d864ee0-68de-46c9-87e1-52f27d329d42
	GENERAL.DEVICES:                        enp0s3
	GENERAL.STATE:                          activated
	GENERAL.DEFAULT:                        yes
	GENERAL.DEFAULT6:                       no
	GENERAL.SPEC-OBJECT:                    --
	GENERAL.VPN:                            no
	GENERAL.DBUS-PATH:                      /org/freedesktop/NetworkManager/ActiveConnection/1
	GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/Settings/1
	GENERAL.ZONE:                           --
	GENERAL.MASTER-PATH:                    --
	IP4.ADDRESS[1]:                         10.0.2.15/24
	IP4.GATEWAY:                            10.0.2.2
	IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 10.0.2.2, mt = 100
	IP4.ROUTE[2]:                           dst = 10.0.2.0/24, nh = 0.0.0.0, mt = 100
	IP4.DNS[1]:                             192.168.0.1
	DHCP4.OPTION[1]:                        dhcp_lease_time = 86400
	DHCP4.OPTION[2]:                        dhcp_rebinding_time = 75600
	DHCP4.OPTION[3]:                        dhcp_renewal_time = 43200
	DHCP4.OPTION[4]:                        dhcp_server_identifier = 10.0.2.2
	DHCP4.OPTION[5]:                        domain_name_servers = 192.168.0.1
	DHCP4.OPTION[6]:                        expiry = 1617548696
	DHCP4.OPTION[7]:                        ip_address = 10.0.2.15
	DHCP4.OPTION[8]:                        requested_broadcast_address = 1
	DHCP4.OPTION[9]:                        requested_dhcp_server_identifier = 1
	DHCP4.OPTION[10]:                       requested_domain_name = 1
	DHCP4.OPTION[11]:                       requested_domain_name_servers = 1
	DHCP4.OPTION[12]:                       requested_domain_search = 1
	DHCP4.OPTION[13]:                       requested_host_name = 1
	DHCP4.OPTION[14]:                       requested_interface_mtu = 1
	DHCP4.OPTION[15]:                       requested_ms_classless_static_routes = 1
	DHCP4.OPTION[16]:                       requested_nis_domain = 1
	DHCP4.OPTION[17]:                       requested_nis_servers = 1
	DHCP4.OPTION[18]:                       requested_ntp_servers = 1
	DHCP4.OPTION[19]:                       requested_rfc3442_classless_static_routes = 1
	DHCP4.OPTION[20]:                       requested_root_path = 1
	DHCP4.OPTION[21]:                       requested_routers = 1
	DHCP4.OPTION[22]:                       requested_static_routes = 1
	DHCP4.OPTION[23]:                       requested_subnet_mask = 1
	DHCP4.OPTION[24]:                       requested_time_offset = 1
	DHCP4.OPTION[25]:                       requested_wpad = 1
	DHCP4.OPTION[26]:                       routers = 10.0.2.2
	DHCP4.OPTION[27]:                       subnet_mask = 255.255.255.0
	IP6.ADDRESS[1]:                         fe80::b1c6:6b7f:7a28:ccbf/64
	IP6.GATEWAY:                            --
	IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
	IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, table=255
	```

### 新增網絡設定
- `nmcli connection add con-name <設定檔名> type <網路類型> ifname <網卡名>`，IPV4、IPV6都自動取得IP(無指定)
	```bash
	[root@localhost yum.repos.d]# nmcli connection add con-name home type ethernet ifname enp0s3
	Connection 'home' (d857c29c-08c8-4d96-9d81-77ce7e73266f) successfully added.
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              4cb86086-9bbd-40c1-839b-235f290e3425  bridge    virbr0
	home                d857c29c-08c8-4d96-9d81-77ce7e73266f  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	```
- `nmcli connection add con-name <設定檔名> type <網路類型> ifname <網卡名> ipv4.addresses <IPV4> ipv6.addresses <IPV6>`，IPV4、IPV6都為固定IP
	```bash
	[root@localhost yum.repos.d]# nmcli connection add con-name office type ethernet ifname enp0s3 ipv4.addresses 192.168.10.10 ipv6.addresses 2001:db8:0:1::1
	Connection 'office' (c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55) successfully added.
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              4cb86086-9bbd-40c1-839b-235f290e3425  bridge    virbr0
	home                d857c29c-08c8-4d96-9d81-77ce7e73266f  ethernet  --
	office              c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	```

### 網絡啟動/關閉
- `nmcli device <connect|disconnect> <網卡名>`，啟動/關閉網卡
	```bash
	[root@localhost yum.repos.d]# nmcli device connect enp0s3
	Device 'enp0s3' successfully activated with '5d864ee0-68de-46c9-87e1-52f27d329d42'.
	[root@localhost yum.repos.d]# nmcli device disconnect virbr0
	Device 'virbr0' successfully disconnected.
	```
- `nmcli connection <up|down> <網卡名>`，啟動/關閉設定檔
	```bash
	[root@localhost yum.repos.d]# nmcli connection up home
	Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	home                d857c29c-08c8-4d96-9d81-77ce7e73266f  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              ce8f2454-8b88-43af-9aed-d2e90fe9d365  bridge    virbr0
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  --
	office              c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              ce8f2454-8b88-43af-9aed-d2e90fe9d365  bridge    virbr0
	home                d857c29c-08c8-4d96-9d81-77ce7e73266f  ethernet  --
	office              c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	```

### 修改網絡設定
- `nmcli connection modify <設定檔名> [<欄位名> <欄位值> ...]`，修改設定檔，可一直設置多個欄位(中間以空白隔開)
	1. 欄位名可用`nmcli connection show <設定檔名>`查看
	2. 默認是override，若要附加其中一筆用`+`、刪除其中一筆用`-`
	```bash
	[root@localhost yum.repos.d]# nmcli connection modify home ipv4.addresses 192.168.1.1 ipv4.gateway 255.255.255.0
	[root@localhost yum.repos.d]# nmcli connection show home |grep ipv4.address
	ipv4.addresses:                         192.168.1.1/32
	[root@localhost yum.repos.d]# nmcli connection show home |grep ipv4.gateway
	ipv4.gateway:                           255.255.255.0
	```
- `nmcli connection reload`，重新讀取設定檔

### 刪除網絡設定
- `nmcli connection delete <設定檔名>`
	```bash
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              ce8f2454-8b88-43af-9aed-d2e90fe9d365  bridge    virbr0
	home                d857c29c-08c8-4d96-9d81-77ce7e73266f  ethernet  --
	office              c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	[root@localhost yum.repos.d]# nmcli connection delete home
	Connection 'home' (d857c29c-08c8-4d96-9d81-77ce7e73266f) successfully deleted.
	[root@localhost yum.repos.d]# nmcli connection show
	NAME                UUID                                  TYPE      DEVICE
	enp0s3              5d864ee0-68de-46c9-87e1-52f27d329d42  ethernet  enp0s3
	Wired connection 2  52ac1999-d6a8-4032-a1ed-1bf3d1715abe  ethernet  enp0s8
	virbr0              ce8f2454-8b88-43af-9aed-d2e90fe9d365  bridge    virbr0
	office              c1610dd9-f9e5-42c2-b8dc-545cf1bb2d55  ethernet  --
	Wired connection 1  a678ca64-452a-47b9-a98d-603393bde734  ethernet  --
	```

### 從設定檔修改網絡設置
- 網卡設定檔路徑：/etc/sysconfig/network-scripts/ifcfg-<設定檔名>
	```bash
	[root@localhost yum.repos.d]# ll /etc/sysconfig/network-scripts/
	total 16
	-rw-r--r--. 1 root root 282 Apr  3 21:13 ifcfg-enp0s3
	-rw-r--r--. 1 root root 342 Apr  4 01:49 ifcfg-office
	-rw-r--r--. 1 root root 297 Mar  8 09:38 ifcfg-Wired_connection_1
	-rw-r--r--. 1 root root 296 Mar  8 09:39 ifcfg-Wired_connection_2
	```
- 不建議直接修改設定檔，因為很多和指令的值會不一樣，詳細可參考Red Hat官方文件：[CHAPTER 11. NETWORK INTERFACES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/ch-network_interfaces#s1-networkscripts-files)
- 修改設定檔後需要執行的命令，以讀取最信的設定
	```bash
	[root@localhost yum.repos.d]# nmcli con reload
	[root@localhost yum.repos.d]# nmcli con down "enp0s3"
	Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/8)
	[root@localhost yum.repos.d]# nmcli con up "enp0s3"
	Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/10)
	```

## 🐧電腦名稱及解析(DNS)
DNS相關介紹可參考：[[🌐網路架構_02_TCP IP]]

### 理論介紹
- 固定IP-->建議設置電腦名稱，并注冊至DNS Server
- 動態IP-->因為IP每次開機都會不一樣，所以不建議設置電腦名稱
	![Linux_RH124_16_網絡管理_01_動態IP取hostname過程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_16_%E7%B6%B2%E7%B5%A1%E7%AE%A1%E7%90%86_01_%E5%8B%95%E6%85%8BIP%E5%8F%96hostname%E9%81%8E%E7%A8%8B.png?raw=true)

### 查看及修改hostname
- `hostname`，查看電腦名稱，是讀取/etc/hostname
	```bash
	[root@localhost yum.repos.d]# hostname
	localhost.localdomain
	```
- `hostnamectl status`，查看電腦名稱
	```bash
	[root@localhost yum.repos.d]# hostnamectl status
	   Static hostname: localhost.localdomain
			 Icon name: computer-vm
			   Chassis: vm
			Machine ID: 25584f8132cc49d4ac22b8d243d95c8c
			   Boot ID: ad05b59e2e5b47838c2e1770c92d5a12
		Virtualization: oracle
	  Operating System: Red Hat Enterprise Linux 8.1 (Ootpa)
		   CPE OS Name: cpe:/o:redhat:enterprise_linux:8.1:GA
				Kernel: Linux 4.18.0-147.el8.x86_64
		  Architecture: x86-64
	```
- `hostnamectl set-hostname <name>`，設置電腦名稱
	```bash
	[root@localhost yum.repos.d]# hostnamectl set-hostname mickey.try
	[root@localhost yum.repos.d]# hostname
	mickey.try
	```

### hostname名稱解析
![Linux_RH124_16_網絡管理_02_名稱解析](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_16_%E7%B6%B2%E7%B5%A1%E7%AE%A1%E7%90%86_02_%E5%90%8D%E7%A8%B1%E8%A7%A3%E6%9E%90.png?raw=true)
- /etc/hosts，相當於local DNS設置
	```
	<IP> <full hostname> <short hostname>
	```
- /etc/nsswitch.conf中設置若hostname重覆時，IP是以/etc/hosts為主還是以DNS Server為主
- 正向名稱解析，查看指定域名IP
	1. `host <域名>`
	```bash
	[root@localhost yum.repos.d]# host www.google.com
	www.google.com has address 216.58.200.36
	www.google.com has IPv6 address 2404:6800:4012:1::2004
	```
	2. `nslookup <域名>`
	```bash
	[root@localhost yum.repos.d]# nslookup www.google.com
	Server:         192.168.0.1
	Address:        192.168.0.1#53

	Non-authoritative answer:
	Name:   www.google.com
	Address: 216.58.200.36
	Name:   www.google.com
	Address: 2404:6800:4012::2004
	```
	3. `dig <域名>`
	```bash
	[root@localhost yum.repos.d]# dig www.google.com

	; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el8 <<>> www.google.com
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63769
	;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; udp: 1232
	; COOKIE: 0976d92f98c0c354e75039b66068a376680aed88ae67ca44 (good)
	;; QUESTION SECTION:
	;www.google.com.                        IN      A

	;; ANSWER SECTION:
	www.google.com.         93      IN      A       216.58.200.36

	;; Query time: 13 msec
	;; SERVER: 192.168.0.1#53(192.168.0.1)
	;; WHEN: Sun Apr 04 01:18:49 CST 2021
	;; MSG SIZE  rcvd: 87
	```
	4. `getent hosts <hostname>`，查看/etc/hosts
		```bash
		[root@localhost yum.repos.d]# getent hosts localhost
		::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
		[root@localhost yum.repos.d]# cat /etc/hosts
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		```
- /etc/resolv.conf中記錄DNS Server IP，當設定檔啟用時，會被網卡中設置DNS覆蓋
	```bash
	[root@localhost yum.repos.d]# cat /etc/resolv.conf
	# Generated by NetworkManager
	search try
	nameserver 192.168.0.1
	```
