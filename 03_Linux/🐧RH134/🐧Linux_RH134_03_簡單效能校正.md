# 簡單效能校正
## 🐧turned-adm
一般的效能校正是通過kernel、driver加參數實現的，因為調整的門檻較高，因此Linux有一個簡單的效能校正工具`turned`，相當於是懶人包(或稱為劇本)
- `tuned-adm active`，當前套用劇本
	```bash
	[root@mickey tmpfiles.d]# tuned-adm active
	Current active profile: virtual-guest
	```
- `tuned-adm list`，列出所有劇本
	1. virtual-guest，虛擬機
	2. virtual-host，跑虛擬機的實體機
	```bash
	[root@mickey tmpfiles.d]# tuned-adm list
	Available profiles:
	- balanced                    - General non-specialized tuned profile
	- desktop                     - Optimize for the desktop use-case
	- hpc-compute                 - Optimize for HPC compute workloads
	- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
	- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
	- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
	- powersave                   - Optimize for low power consumption
	- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
	- virtual-guest               - Optimize for running inside a virtual guest
	- virtual-host                - Optimize for running KVM guests
	Current active profile: virtual-guest
	```
- `tuned-adm recommend`，推薦劇本
	```bash
	[root@mickey tmpfiles.d]# tuned-adm recommend
	virtual-guest
	```
- `tuned-adm profile <劇本名>`，修改劇本
	```bash
	[root@mickey tmpfiles.d]# tuned-adm active
	Current active profile: virtual-guest
	[root@mickey tmpfiles.d]# tuned-adm profile desktop
	[root@mickey tmpfiles.d]# tuned-adm active
	Current active profile: desktop
	```
- `tuned-adm off`，關閉劇本
- 使用web console指定劇本
	參考：[Using the web console for selecting performance profiles -red hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/managing_systems_using_the_rhel_7_web_console/configuring-system-settings-in-the-web-console_system-management-using-the-rhel-7-web-console#using-the-web-console-for-selecting-performance-profiles_configuring-system-settings-in-the-web-console)

## 🐧影響執行效能參數
- nice值：靜態優先級
	1. nice值范圍為-20~19，預設為0
	2. nice越小，表示優先級越高，取得CPU資源越多
	3. 可通過`top`或`ps -al`查看NI欄位
	4. root可隨意調整nice值，一般user初使設置只能為正數(1~19)且調整只能將nice值調大
- PR值(priority)：動態優先級
	![nice&priority](https://oscimg.oschina.net/oscnet/1fcc725aec8ebc237009c2e383e25d2b886.png)
	1. 一般只會看nice值，不太看PR
	2. 預設為80
	3. 可通過`top`或`ps -al`查看PRI欄位

### nice值設置
- 不指定nice值，默認為0
	```bash
	[mickey@mickey ~]$ sleep 1000 &
	[1] 2711
	[mickey@mickey ~]$ ps -l 2711
	F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
	0 S  1000  2711  2357  0  80   0 -  1820 hrtime pts/0      0:00 sleep 1000
	```
- 使用`nice`指令，但不指定nice值，默認為10
	```bash
	[mickey@mickey ~]$ nice sleep 1000 &
	[2] 2749
	[mickey@mickey ~]$ ps -l 2749
	F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
	0 S  1000  2749  2357  0  90  10 -  1820 hrtime pts/0      0:00 sleep 1000
	```
- `nice -n <nice值> <指令>`，指定nice值執行指令，一般user只能指定正數
	```bash
	[mickey@mickey ~]$ nice -n -5 sleep 1000 &
	[3] 2778
	[mickey@mickey ~]$ nice: cannot set niceness: Permission denied
	[mickey@mickey ~]$ nice -n 5 sleep 1000 &
	[4] 2779
	[mickey@mickey ~]$ ps -l 2779
	F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
	0 S  1000  2779  2357  0  85   5 -  1820 hrtime pts/0      0:00 sleep 1000
	```
- `renice -n <nice值> <PID>`，修改執行中程式的nice值，一般user只能調大
	```bash
	[mickey@mickey ~]$ ps -l 2779
	F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY        TIME CMD
	0 S  1000  2779  2357  0  85   5 -  1820 hrtime pts/0      0:00 sleep 1000
	[mickey@mickey ~]$ renice -n 0 2779
	renice: failed to set priority for 2779 (process ID): Permission denied
	[mickey@mickey ~]$ renice -n 10 2779
	2779 (process ID) old priority 5, new priority 10
	```
