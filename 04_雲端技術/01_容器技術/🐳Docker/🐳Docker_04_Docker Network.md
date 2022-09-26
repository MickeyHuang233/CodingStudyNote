# Docker Network
- 啟動docker.service時用`ifconfig`會看到docker0的虛擬網橋，它的作用和virbr0差不多
	```bash
	[mickey@vm102 springboot]$ ifconfig
	docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
	        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
	        inet6 fe80::42:a8ff:fe10:716a  prefixlen 64  scopeid 0x20<link>
	        ether 02:42:a8:10:71:6a  txqueuelen 0  (Ethernet)
	        RX packets 14  bytes 2301 (2.2 KiB)
	        RX errors 0  dropped 0  overruns 0  frame 0
	        TX packets 22  bytes 3707 (3.6 KiB)
	        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
	```
- 作用
	1. 容器間的互聯、通信、端口映射
	2. 容器IP變動時，可通過服務名直接用網絡通信

## 🐳基本命令
- `docker network ls`，查看Docker網絡
	```bash
	[mickey@vm102 springboot]$ sudo docker network ls
	NETWORK ID     NAME      DRIVER    SCOPE
	4a5e6b286ebc   bridge    bridge    local
	34f171a402e0   host      host      local
	23ba85c0a143   none      null      local
	```
- `docker network rm <網路名稱>`，刪除網路
- `docker network inspect <網路名稱>`，查看網路源信息
- `docker inspect <containerID | container_name>`，可查看網絡相關信息
	```bash
	[mickey@vm102 springboot]$ sudo docker inspect 00375c990259 | tail -n 20
	            "Networks": {
	                "bridge": {
	                    "IPAMConfig": null,
	                    "Links": null,
	                    "Aliases": null,
	                    "NetworkID": "4a5e6b286ebc4d77baac0f16a7a4cce05207c2314ecdda23f2fda8028beba028",
	                    "EndpointID": "4f65e636ec15eed28668f68579fbbfce0071ead6642cc1bd8e628f7282be5ef0",
	                    "Gateway": "172.17.0.1",
	                    "IPAddress": "172.17.0.2",
	                    "IPPrefixLen": 16,
	                    "IPv6Gateway": "",
	                    "GlobalIPv6Address": "",
	                    "GlobalIPv6PrefixLen": 0,
	                    "MacAddress": "02:42:ac:11:00:02",
	                    "DriverOpts": null
	                }
	            }
	        }
	    }
	]
	```

## 🐳網絡模式種類
### bridge
- 虛擬網橋，默認，為每個容器分配IP，并將容器連接到docker0，讓本機和容器間可以通過網橋相互通信
	![[Docker_04_Docker Network_01_bridge.png]]
- 建立容器時用`--network bridge`指定，默認使用
- 本機端看網卡資訊，`if21`表示接上容器的21號的邏輯網卡
	```bash
	[mickey@vm102 ~]$ ip addr
	(省略)
	22: veth6895fdc@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
	    link/ether ba:31:86:e4:ab:6a brd ff:ff:ff:ff:ff:ff link-netnsid 1
	    inet6 fe80::b831:86ff:fee4:ab6a/64 scope link
	       valid_lft forever preferred_lft forever
	```
- 容器端看網卡資訊，`if21`表示接上本機的22號的邏輯網卡
	```bash
	[root@7a13c95d4a88 /]# ip addr
	(省略)
	21: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	```

### host
- 直接使用主機的IP、端口，需要直接從`localhost:<端口號>`找容器，若端口被占時則遞增
	![[Docker_04_Docker Network_02_host.png]]
- 建立容器時用`--network host`指定，此時指定`-p`端口映射會有警告信息，因為此設置無意義
	```bash
	[mickey@vm102 ~]$ sudo docker run -d --network host -p 6379:6379 redis
	WARNING: Published ports are discarded when using host network mode
	725d8d08f1d1d58e6a6067127933da8811d9d8f4d5dc0469d9ab57bf2deb8c5a
	[mickey@vm102 ~]$ sudo docker ps
	CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
	725d8d08f1d1   redis      "docker-entrypoint.s…"   33 seconds ago   Up 33 seconds                                               brave_bhabha
	[mickey@vm102 ~]$ sudo docker inspect brave_bhabha | tail -n 20
	            "Networks": {
	                "host": {
	                    "IPAMConfig": null,
	                    "Links": null,
	                    "Aliases": null,
	                    "NetworkID": "34f171a402e0e8634c6c3081d4b71f87adfb9c26c71170d4cdbdd17f4e70ebe9",
	                    "EndpointID": "9f9ac09f1024996dd407f48aafb8fc0316beae7e8c5c2d0b624890640ef0db7b",
	                    "Gateway": "",
	                    "IPAddress": "",
	                    "IPPrefixLen": 0,
	                    "IPv6Gateway": "",
	                    "GlobalIPv6Address": "",
	                    "GlobalIPv6PrefixLen": 0,
	                    "MacAddress": "",
	                    "DriverOpts": null
	                }
	            }
	        }
	    }
	]
	```

### none
- 容器有獨立的Network namespace，但沒有任何的網絡設置，如IP、網橋連接…，基本不會使用
- 建立容器時用`--network none`指定，禁用網絡功能，只有lo的配置
	```bash
	[mickey@vm102 ~]$ sudo docker run -it --rm --network none centos
	[root@4d4a1f0a0639 /]# ip addr
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	```

### container
- 創建容器時不會創建自己的網卡、IP，而是指定容器共享的IP、端口范圍…
	![[Docker_04_Docker Network_03_container.png]]
- 建立容器時用`--network container:<container_name>`指定，此時指定`-p`端口映射會有錯誤信息，因為此設置無意義
	1. 啟動容器1
		```bash
		[mickey@vm102 ~]$ sudo docker run -it --rm --name continer_01 centos
		[root@cd076f2a32a0 /]# ip addr
		(省略)
		23: eth0@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
		    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
		    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
		       valid_lft forever preferred_lft forever
		```
	2. 啟動容器2(網絡模式為container)
		```bash
		[mickey@vm102 ~]$ sudo docker run -it --rm --name container_02 --network container:continer_01 centos
		[root@cd076f2a32a0 /]# ip addr
		(省略)
		23: eth0@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
		    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
		    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
		       valid_lft forever preferred_lft forever
		```

## 🐳自定義網絡
1. `docker network create <網路名稱>`，建立自定義網路
	```bash
	[mickey@vm102 ~]$ sudo docker network create mi_net
	b7e3ac62f2577868c50078ed9ef7dfc3d7c9332b0154423bb42e8784ba45799d
	[mickey@vm102 ~]$ sudo docker network ls
	NETWORK ID     NAME      DRIVER    SCOPE
	d42f44a2c1e4   bridge    bridge    local
	34f171a402e0   host      host      local
	b7e3ac62f257   mi_net    bridge    local
	23ba85c0a143   none      null      local
	```
2. 啟動服務，`--network`指定網絡，並確認所有服務是否在同網段
	```bash
	[mickey@vm102 ~]$ sudo docker run -it --name centos2 --network mi_net centos
	[root@97be8403a59d /]# ipaddr
	bash: ipaddr: command not found
	[root@97be8403a59d /]# ip addr
	(省略)
	14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	[mickey@vm102 ~]$ sudo docker run -it --name centos1 --network mi_net centos
	[root@5173d2fc938c /]# ip addr
	(省略)
	       valid_lft forever preferred_lft forever
	16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:12:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.18.0.3/16 brd 172.18.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	```
3. 此時用容器名就可以相互連通，不需要IP
	```bash
	[root@5173d2fc938c /]# ping -c 3 centos2
	PING centos2 (172.18.0.2) 56(84) bytes of data.
	64 bytes from centos2.mi_net (172.18.0.2): icmp_seq=1 ttl=64 time=0.167 ms
	64 bytes from centos2.mi_net (172.18.0.2): icmp_seq=2 ttl=64 time=0.160 ms
	64 bytes from centos2.mi_net (172.18.0.2): icmp_seq=3 ttl=64 time=0.166 ms
	```




