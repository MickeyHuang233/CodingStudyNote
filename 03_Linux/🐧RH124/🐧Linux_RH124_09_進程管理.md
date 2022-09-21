# Linux進程管理
|系統|進程方式|
|---|---|
|dos系統|單人單工|
|Windows|單人多工|
|Linux|多人多工|
|x86 cpu|單工-->假多工(每個process只執行1/10秒)|

## 🐧基本介紹
- 執行進程(process)的方法
	1. exec：執行Shell
	2. fork：呼叫子進程，在執行子進程時，父進程就沒有控制權
		![[Linux_RH124_10_進程管理_01_進程生命周期.png]]
- 進程號：PID
- 父進程號：PPID

## 🐧進程狀態
![[Linux_RH124_10_進程管理_02_進程狀態.png]]
參考：[Linux進程狀態 -知乎](https://zhuanlan.zhihu.com/p/344725512)
- Running：R
- Sleeping：S(在waiting queue中)、D、K、I
- Stopped：T
- zomble：Z、X
- n：表示進程擁有比普通優先級更低的優先級

## 🐧監控進程
其他進程監聽工具：htop、System Monitor

### ps
- `ps`，只看此Terminal的線程信息
	```bash
	[mickey@localhost ~]$ ps | head -10
	  PID TTY          TIME CMD
	 2387 pts/0    00:00:00 bash
	 3618 pts/0    00:00:00 ps
	 3619 pts/0    00:00:00 head
	```
- `ps -aux`，查看進程信息(瞬間)
	1. `-u`，user list，以用戶的格式顯示進程信息
	2. `-a`，顯示所有Terminal的線程
	3. `-x`，顯示後台進程運行(TTY=?)
	```bash
	[mickey@localhost ~]$ ps -aux|head -10
	USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
	root         1  0.0  0.3 179296 14016 ?        Ss   06:52   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	root         2  0.0  0.0      0     0 ?        S    06:52   0:00 [kthreadd]
	root         3  0.0  0.0      0     0 ?        I<   06:52   0:00 [rcu_gp]
	root         4  0.0  0.0      0     0 ?        I<   06:52   0:00 [rcu_par_gp]
	root         6  0.0  0.0      0     0 ?        I<   06:52   0:00 [kworker/0:0H-kblockd]
	root         8  0.0  0.0      0     0 ?        I<   06:52   0:00 [mm_percpu_wq]
	root         9  0.0  0.0      0     0 ?        S    06:52   0:00 [ksoftirqd/0]
	root        10  0.0  0.0      0     0 ?        I    06:52   0:00 [rcu_sched]
	root        11  0.0  0.0      0     0 ?        S    06:52   0:00 [migration/0]
	```
- `ps -lax`
	1. `-l`，long format
	```bash
	[mickey@localhost ~]$ ps -lax|head -10
	F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
	4     0     1     0  20   0 179296 14020 -      Ss   ?          0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	1     0     2     0  20   0      0     0 -      S    ?          0:00 [kthreadd]
	1     0     3     2   0 -20      0     0 -      I<   ?          0:00 [rcu_gp]
	1     0     4     2   0 -20      0     0 -      I<   ?          0:00 [rcu_par_gp]
	1     0     6     2   0 -20      0     0 -      I<   ?          0:00 [kworker/0:0H-kblockd]
	1     0     8     2   0 -20      0     0 -      I<   ?          0:00 [mm_percpu_wq]
	1     0     9     2  20   0      0     0 -      S    ?          0:00 [ksoftirqd/0]
	1     0    10     2  20   0      0     0 -      R    ?          0:00 [rcu_sched]
	1     0    11     2 -100  -      0     0 -      S    ?          0:00 [migration/0]
	```
- `ps -ef`
	```bash
	[mickey@localhost ~]$ ps -ef|head -10
	UID        PID  PPID  C STIME TTY          TIME CMD
	root         1     0  0 06:52 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	root         2     0  0 06:52 ?        00:00:00 [kthreadd]
	root         3     2  0 06:52 ?        00:00:00 [rcu_gp]
	root         4     2  0 06:52 ?        00:00:00 [rcu_par_gp]
	root         6     2  0 06:52 ?        00:00:00 [kworker/0:0H-kblockd]
	root         8     2  0 06:52 ?        00:00:00 [mm_percpu_wq]
	root         9     2  0 06:52 ?        00:00:00 [ksoftirqd/0]
	root        10     2  0 06:52 ?        00:00:00 [rcu_sched]
	root        11     2  0 06:52 ?        00:00:00 [migration/0]
	```
- `p -o <欄位名(多個以,隔開)>`，查看指定欄位
	```bash
	[mickey@localhost ~]$ ps -o user,pid,command
	USER       PID COMMAND
	mickey    2387 -bash
	mickey    3755 ps -o user,pid,command
	```
- `--sort <欄位名>`，顯示特定欄位排序
	```bash
	[mickey@localhost ~]$ ps -aux --sort pid|head -10
	USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
	root         1  0.0  0.3 179296 14020 ?        Ss   06:52   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
	root         2  0.0  0.0      0     0 ?        S    06:52   0:00 [kthreadd]
	root         3  0.0  0.0      0     0 ?        I<   06:52   0:00 [rcu_gp]
	root         4  0.0  0.0      0     0 ?        I<   06:52   0:00 [rcu_par_gp]
	root         6  0.0  0.0      0     0 ?        I<   06:52   0:00 [kworker/0:0H-kblockd]
	root         8  0.0  0.0      0     0 ?        I<   06:52   0:00 [mm_percpu_wq]
	root         9  0.0  0.0      0     0 ?        S    06:52   0:00 [ksoftirqd/0]
	root        10  0.0  0.0      0     0 ?        R    06:52   0:00 [rcu_sched]
	root        11  0.0  0.0      0     0 ?        S    06:52   0:00 [migration/0]
	```
- 欄位說明
	1. PID，進程編號
	1. PPID，父進程編號
	1. TTY，終端機號
	1. TIME，此進程占用CPU時間
	1. CMD，正在執行命令或進程名
	1. USER，用戶名
	1. PID，進程id
	1. %CPU，占用的cpu
	1. %MEM，占用內存
	1. VSZ，使用的虛擬內存
	1. RSS，使用物理內存情況
	1. TTY，使用的終端
	1. STAT，進程的狀態
	1. START，啟動時間
	1. TIME，占用cpu總時間
	1. COMMAND，進程執行時的命令行

## 🐧前景及背景作業
### 查看後台進程 jobs
- `jobs`
```bash
[mickey@localhost ~]$ jobs
[1]+  Running                 sleep 10000 &
```

### 後台執行
- `<指令>&`，指令直接在後台執行
```bash
[mickey@localhost ~]$ sleep 10000&
[1] 4334
```

### 後台-->前台
`fg %<jobID>`，將指定任務叫到前台運行
```bash
[mickey@localhost ~]$ jobs
[1]+  Running                 sleep 10000 &
[mickey@localhost ~]$ fg %1
sleep 10000
```

### 前台-->後台
1. 按Ctrl + Z，執行在前台的任務轉為後台，但狀態為變為stopped
	```bash
	[mickey@localhost ~]$ sleep 10000
	^Z
	[2]+  Stopped                 sleep 10000
	```
2. `bg %<jobID>`，改變指定後台任務狀態，stopped-->running
	```bash
	[mickey@localhost ~]$ jobs
	[1]-  Stopped                 sleep 10000
	[2]+  Stopped                 sleep 10000
	[mickey@localhost ~]$ bg %1
	[1]- sleep 10000 &
	[mickey@localhost ~]$ jobs
	[1]-  Running                 sleep 10000 &
	[2]+  Stopped                 sleep 10000
	```

## 🐧結束進程
### 查詢進程號 pidof
`pidof <指令>`，查詢某指令的PID
```bash
[mickey@localhost ~]$ pidof sleep
5039 4434 4334
```

### 中斷信號 kill signal
|signal|name|description|
|---|---|---|
|1|hangup|掛起|
|2|keyboard interrupt|中斷執行中的線程，相當於Ctrl+C|
|3|keyboard quit|將線程的記憶體內容dump至文件，相當於Ctrl+\|
|9|kill|線程停止，程式不正常停止|
|15|terminate(default)|線程以正常流程停止|
|18|continue|線程繼續執行|
|19|stop|線程暫停|
|20|keyboard stop|後台執行線程，相當於Ctrl+Z|

`kill -l`，查看所有signal，中斷信號可以指定號碼，也可用單字指定
```bash
[mickey@localhost ~]$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

### kill
- `kill [中斷信號] <進程號>`，停止進程，中斷信號默認為15
	```bash
	[root@localhost ~]# ps -ef | grep "sshd"
	root      1450     1  0 20:40 ?        00:00:00 /usr/sbin/sshd -D
	root      2112  1450  0 20:41 ?        00:00:00 sshd: root@pts/0
	root      2200  1450  0 20:42 ?        00:00:00 sshd: mickey [priv]
	mickey    2214  2200  0 20:43 ?        00:00:00 sshd: mickey@pts/1
	root      2290  2118  0 20:44 pts/0    00:00:00 grep --color=auto sshd
	[root@localhost ~]# kill 2214
	[root@localhost ~]# ps -ef | grep "sshd"
	root      1450     1  0 20:40 ?        00:00:00 /usr/sbin/sshd -D
	root      2112  1450  0 20:41 ?        00:00:00 sshd: root@pts/0
	root      2309  2118  0 20:45 pts/0    00:00:00 grep --color=auto sshd
	```
- `kill [中斷信號] %<jobID>`，停止進程，中斷信號默認為15
	```bash
	[mickey@localhost ~]$ jobs
	[1]-  Running                 sleep 10000 &
	[2]+  Running                 sleep 20000 &
	[mickey@localhost ~]$ kill -9 %1
	[mickey@localhost ~]$ jobs
	[1]-  Killed                  sleep 10000
	[2]+  Running                 sleep 20000 &
	[mickey@localhost ~]$ jobs
	[2]+  Running                 sleep 20000 &
	```

### pkill
`pkill [中斷信號] <進程名稱>`，通過進程名停止==執行中==的進程，中斷信號默認為15
- 如果結束父進程，相應的子進程也會一并結束
```bash
[mickey@localhost ~]$ ps -aux|grep sleep
mickey    8582  0.0  0.0   7280   708 pts/0    S    11:53   0:00 sleep 1000
mickey    8583  0.0  0.0   7280   812 pts/0    S    11:53   0:00 sleep 2000
root      8592  0.0  0.0   7280   808 ?        S    11:53   0:00 sleep 60
mickey    8594  0.0  0.0  12108  1096 pts/0    R+   11:53   0:00 grep --color=auto sleep
[mickey@localhost ~]$ pkill -9 sleep
pkill: killing pid 8592 failed: Operation not permitted
[1]-  Killed                  sleep 1000
[2]+  Killed                  sleep 2000
[mickey@localhost ~]$ ps -aux|grep sleep
root      8592  0.0  0.0   7280   808 ?        S    11:53   0:00 sleep 60
mickey    8598  0.0  0.0  12108  1076 pts/0    R+   11:54   0:00 grep --color=auto sleep
```

### killall
`killall [中斷信號] <進程名稱>`，通過進程名停止進程，支持通配符，中斷信號默認為15
```bash
[root@localhost ~]# ps -ef | grep "gedit"
mickey    3272  3175  3 20:48 pts/1    00:00:00 gedit
root      3287  2118  0 20:48 pts/0    00:00:00 grep --color=auto gedit
[root@localhost ~]# killall gedit
[root@localhost ~]# ps -ef | grep "gedit"
root      3340  2118  0 20:49 pts/0    00:00:00 grep --color=auto gedit
```

## 🐧用戶進程管理
### 查看當前在線用戶 w
- `w`，查看當前在線用戶
	1. `-h`，不顯示header
	2. `-u <用戶名>`，查詢指定用戶
	```bash
	[root@localhost ~]# w
	 11:57:56 up  5:05,  2 users,  load average: 0.32, 0.07, 0.02
	USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
	mickey   pts/0    192.168.56.1     06:53   27.00s  0.06s  0.06s -bash
	root     pts/1    192.168.56.1     11:57    4.00s  0.01s  0.00s w
	```
- 欄位說明
	1. LOGIN@，登入時間
	2. IDLE，閒置時間
	3. JCPU，指該TTY終端連接的所有進程占用的時間，不包含後台作業時間
	4. PCPU，當前進程所占用的時間
	5. WHAT，目使使用的指令

### 查看指定用戶進程 pgrep
`pgrep -l -u <用戶名>`，查看指定用戶的所有程序
```bash
[root@localhost ~]# pgrep -l -u mickey
2361 systemd
2368 (sd-pam)
2379 pulseaudio
2380 sshd
2387 bash
2434 dbus-daemon
```

### 樹狀顯示進程 pstree
`pstree`，以樹狀形式顯示進程
* `-p`，顯示進程ID
* `-u <用戶名>`，查詢指定用戶
```bash
[root@localhost ~]# pstree -up mickey
sshd(9060)───bash(9067)

systemd(9041)─┬─(sd-pam)(9049)
              ├─dbus-daemon(9114)───{dbus-daemon}(9115)
              └─pulseaudio(9059)───{pulseaudio}(9113)
```

### 停止指定用戶進程 pkill
- `pkill [中斷信號] -u <用戶名>`，停止指定用戶所有的程序
```bash
[root@localhost ~]# pgrep -l -u mickey
2361 systemd
2368 (sd-pam)
2379 pulseaudio
2380 sshd
2387 bash
2434 dbus-daemon
8957 top
[root@localhost ~]# pkill -9 -u mickey
[root@localhost ~]# pgrep -l -u mickey
```
- `pkill [中斷信號] -t <終端名>`，發送中中斷信號至指定Terminal執行的所有進程
```bash
[root@localhost ~]# w
 12:15:56 up  5:23,  2 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
mickey   pts/0    192.168.56.1     12:08   42.00s  0.01s  0.01s -bash
root     pts/1    192.168.56.1     11:57    0.00s  0.02s  0.00s w
[root@localhost ~]# pkill -t pts/0
```

## 🐧進程資源監控
### 平均負載 Load Average Calculation
- load average，平均負載，指在特定時間間隔內運行隊列中的平均進程數，一取都會出現三個數值，分別代表最近1、5、15分鐘的系統平均負載
- `uptime`，查看系統負載
	```bash
	[mickey@localhost ~]$ uptime
	 07:09:45 up 2 min,  1 user,  load average: 0.05, 0.07, 0.02
	```
	1. 07:09:45，系統當前時間
	2. up 2 min，主機已運行時間
	3. 1 user，用戶總**連接**數
	4. load average: 0.05, 0.07, 0.02，系統最近1、5、15分鐘所有CPU的系統平均負載，若單個CPU負載大於1則代表過載
- `lscpu`，查看此電腦的cpu信息
	```bash
	[mickey@localhost ~]$ lscpu
	Architecture:        x86_64
	CPU op-mode(s):      32-bit, 64-bit
	Byte Order:          Little Endian
	CPU(s):              1
	On-line CPU(s) list: 0
	Thread(s) per core:  1
	Core(s) per socket:  1
	Socket(s):           1
	NUMA node(s):        1
	Vendor ID:           GenuineIntel
	CPU family:          6
	Model:               126
	Model name:          Intel(R) Core(TM) i5-1035G1 CPU @ 1.00GHz
	Stepping:            5
	CPU MHz:             1190.401
	BogoMIPS:            2380.80
	Hypervisor vendor:   KVM
	Virtualization type: full
	L1d cache:           48K
	L1i cache:           32K
	L2 cache:            512K
	L3 cache:            6144K
	NUMA node0 CPU(s):   0
	Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq monitor ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single fsgsbase avx2 invpcid rdseed clflushopt md_clear flush_l1d arch_capabilities
	```
	1. Architecture，處理器架構
	2. CPU(s)，CPU數量
	3. On-line CPU(s) list，在線的CPU數量，有時會為了省電或電腦過熱，某些CPU會停止運行
	4. Thread(s) per core，每個核心的線程數
	5. Core(s) per socket，每個插槽上有幾個核心座
	6. Vendor ID，廠商ID
	7. CPU family，CPU系列
	8. Model，型號
	9. Model name，型號名稱
	10. Stepping，步進，可簡單理解為版本號
	11. BogoMIPS，MIPS是每秒百萬條指令，Bogo是偽，總的意思是估算MIPS值
	12. L1d cache，一級高速緩存dcache，用來存儲數據
	13. L1i cache，一級高速緩存icache，用來存儲指令
	14. Flags是標識了一些cpu的特征

### 即時進程監控 top
`top`，動態監控進程，參考：[top命令](https://man.linuxde.net/top)
- `-u`，指定用户名
- `-k`，輸入要結束的進程ID號
- `top -d 10`，指定每10秒更新(默認為3秒
- `-P`：根据CPU使用百分比大小进行排序；

![[Linux_RH124_10_進程管理_03_top.png]]

- 欄位說明
	1. PID，進程號
	2. USER，進程所有者
	3. VIRT，virtual memory虛擬空間大小，是真實使用的內存+映射進程自己使用的內存
	4. RES，resident memory，常駐內存
		![[Linux_RH124_10_進程管理_04_內存劃分.png]]
	5. SHR，shared memory，共享內存

|控制版面命令|說明|
|---|---|
|?<br/>h|顯示幫助畫面|
|l<br/>t<br/>m|切換平均負載(load)信息<br/>切換進程(threads)信息<br/>切換內存(memory)信息|
|1|切換單個CPU和所有CPU總和的信息|
|s|改變兩次刷新的間隔秒數，可為小數。輸入0代表不斷刷新，默認5s|
|b<br/>Shift + b|高亮顯示正在運行的進程，默認是粗體<br/>加粗顯示正在運行的進程，默認打開|
|k|終止進程|
|r|重新安排一個進程的優先級別|
|q|離開|
|Shift + w|write(save)版面存檔|
|Shift + h|切換顯示總線程或單一線程|
|u<br/>Shift + u|過濾用戶|
|Shift + m|按內存使用狀況對進程排序顯示|
|Shift + p|按CPU占用狀況對進程排序顯示|
