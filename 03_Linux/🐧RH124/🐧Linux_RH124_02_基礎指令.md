# Linux_RH124_02_基礎指令
## 🐧Commend Line Function Key
- 指標移至行首：Ctrl + A
- 指標移至行尾：Ctrl + E
- 將指標前的字串刪除：Ctrl + U
- 將指標後的字串刪除：Ctrl + K
- 搜索歷史指令：Ctrl + R
```bash
(reverse-i-search)`who': ls; whoami
```

## 🐧sync
`sync`：將內存的數據同步到磁盤，當我們關機/重啟時，都應該先執行sync命令，坐止數據丟失

## 🐧shutdown
- `shutdown -h now`：立即關機
	等同於：`halt`、`poweroff`
- `shutdown -h 1`：1分鐘後關機
- `shutdown -r now`：立即重啟
	等同於：`reboot`

## 🐧logout
`logout`：注銷用戶
`logout`命令在圖形運行級別無效，在運行級別3下有效

## 🐧history
- 已經執行過歷史命令放置路徑：~/.bash_history
- 預設記錄1000筆指令
- 注意：當用戶登出時才會回寫歷史指令

- `history`，查看所有已經執行過歷史命令
	```bash
	[root@localhost 下載]# history 5
	  363  ll
	  364  rm -r linkToMickey
	  365  ll
	  366  history
	  367  history 5
	[root@localhost 下載]#
	```
- `history 5`，顯示最後執行的5個命令
	```bash
	[root@localhost 下載]# history 5
	   10  ! -4
	   11  history
	   12  history -2
	   13  history 9
	   14  history 5
	```
- `!363`，執行第363號的命令
	```bash
	[root@localhost 下載]# !363
	ll
	總計 8
	drwxr-xr-x. 2 root root    6  1月  5 15:54 testCreate
	-rw-r--r--. 1 root root 1038  1月  5 20:12 testTest
	-rw-r--r--. 1 root root  263  1月  5 12:48 testViEditor
	```
- `!-4`，執行倒數第4號的命令
- `!<指令名>`，執行最近一筆的指令(向上數)，如`!history`執行最近一筆的history指令
	```bash
	[root@localhost Documents]# !ls
	ls; whoami
	ifconfig.txt  selectName.txt
	root
	[root@localhost Documents]#
	```

## 🐧echo
`echo`，輸出內容到控制台，可輸出環境變量
```bash
[root@localhost 下載]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@localhost 下載]# echo "Hello World"
Hello World
```
