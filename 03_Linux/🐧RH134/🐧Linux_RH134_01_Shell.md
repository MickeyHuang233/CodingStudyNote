# Shell
Shell是一個命令行解釋器(直譯式程式設言)，它為用戶提供了一個向Linux內核發送請求以便運行程序的界面系統級程序，用戶可以用Shell來啟動、挂起、停止甚至是編寫一些程序。
參考：[學習 Shell Scripts-鳥哥的 Linux 私房菜](http://linux.vbird.org/linux_basic/0340bashshell-scripts.php)
![[Linux_RH124_19_Shell_01.png]]

## 🐧Hello World
### Shell腳本
腳本`#!/bin/bash`開頭，以指定直譯器，若沒寫Red Hat默默為`#!/bin/sh`
```shell
#!/bin/bash

echo "Hello World~!!"
# 輸出指向stderr
echo "ERROR: A ERRPR MESSAGE." >&2
```

### Shell執行
- 前提條件：必須有執行的權限
- 無法在當前資料夾直接輸入要執行的shell文件，因為無指定絕對路徑或相對路徑時，Linux只會找${PATH}下是否有Shell Script
```bash
[mickey@mickey Documents]$ hallo.sh
bash: hallo.sh: command not found...
[mickey@mickey Documents]$ pwd
/home/mickey/Documents
[mickey@mickey Documents]$ ${PATH}
-bash: /home/mickey/.local/bin:/home/mickey/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```
- 執行shell的方法：
	1. 將shell腳本放至${PATH}中的目錄底下(不建議做法)
	2. 將shell腳本的目錄加入${PATH}中(不建議做法)
	3. 輸入絕對路徑
	```bash
	[mickey@mickey Documents]$ $(pwd)/hallo.sh
	Hello World~!!
	```
	4. 輸入相對路徑
	```bash
	[mickey@mickey Documents]$ ./hallo.sh
	Hello World~!!
	```

### 注釋
- 單行注釋
	```shell
	#單行注釋
	```
- 多行注釋
```shell
:<<!
多行注釋
!
```

### 轉義控制字元
如：$、#…等
1. 使用`\控制字元`
	```bash
	[root@localhost ~]# echo \$HOME = $HOME
	$HOME = /root
	```
2. `'控制字元'`使裡面的文字轉義，但`"控制字元"`無法將裡面的文字轉義
	```bash
	[root@localhost ~]# echo '$HOME' = $HOME
	$HOME = /root
	[root@localhost ~]# echo "$HOME" = $HOME
	/root = /root
	```

## 🐧讀取控制台輸入
`read`
`-p`，指定讀取值時的提示符(print)
`-t`，指定讀取值時等待的時間(秒)，若沒有在時間內輸入就不再等待
```shell
#read
read -p "請輸入num01值" NUM01
echo "剛剛輸入的num01值為：$NUM01"

read -p "請輸入num02、num03值" -t 3 NUM02 NUM03
echo "剛剛輸入的num02、num03值為：$NUM02 $NUM03"
```

```bash
[root@localhost testShell]# ./05_ShellRead
請輸入num01值56
剛剛輸入的num01值為：56
請輸入num02值剛剛輸入的num02值為：14 67
剛剛輸入的num02、num03值為：14 67
```

## 🐧變量
- 變量分為：
	1. 系統變量
	2. shell變量：作用域只在這個Shell(登入~登出也是一個Shell)
- 變量命名規則：
	1. 變量名可由字母、數字和下劃線組成，但不能以數字開頭
	2. 變量名一般習慣為大寫
- `set`，顯示當前shell中所有變量
	```bash
	[root@localhost testShell]# set | more
	ABRT_DEBUG_LOG=/dev/null
	BASH=/bin/bash
	BASHOPTS=checkwinsize:cmdhist:expand_aliases:extglob:extquote:force_fignore:histappend:interactive_comments:login_shell:pr
	ogcomp:promptvars:sourcepath
	```

### 變量調用
- 調用參數雖然可以使用`$變數名`，但建議使用`${變數名}`，以免有非預期錯誤
	```shell
	#!/bin/bash
	#調用系統變量
	echo "PATH=$PATH"
	echo "user=$USER"
	```

### 聲明變量
-	注意=兩邊不可以有空格
	```shell
	#!/bin/bash
	#定義變量
	var=100
	echo "var=$var"
	unset var
	echo "var=$var"
	```

	```bash
	[root@localhost testShell]# ./01_firstShell
	var=100
	var=
	```
- 在.sh的聲明的方式一樣，但需注意Shell變數作用域只在此次Shell中，而用戶登入作業系統說到底也是執行Shell，也就是說用戶登出後Shell變數就無效
	```bash
	[root@localhost ~]# KEY=value
	```

	```bash
	[root@localhost ~]# export TEST_KEY=test value
	[root@localhost ~]# echo ${TEST_KEY}
	test
	```

### 釋放變量
- 靜態變量，靜態變量不可以釋放
	```shell
	#!/bin/bash
	#靜態變量，不可撒銷
	readonly static=200
	echo "static=$static"
	unset static
	echo "static=$static"
	```

	```bash
	[root@localhost testShell]# ./01_firstShell
	static=200
	./01_firstShell: line 19: unset: static: cannot unset: readonly variable
	static=200
	```
- 指令行釋放變量
	```bash
	[mickey@localhost Documents]$ test="string"
	[mickey@localhost Documents]$ echo ${test}
	string
	[mickey@localhost Documents]$ unset test
	[mickey@localhost Documents]$ echo ${test}
	```

### 命令行結果為變量
- `'命令'`，聲明指令變量
- `$(命令)`，聲明指令變量
	```shell
	#!/bin/bash
	#將命令返回值賦給變量
	LS_STR01=`ls -la`
	echo "LS_STR01=$LS_STR01"
	LS_STR02=$(ls -la)
	echo "LS_STR02=$LS_STR02"
	```

	```bash
	[root@localhost testShell]# ./01_firstShell
	LS_STR01=總計 4
	drwxr-xr-x. 2 root root  27  1月 23 10:37 .
	drwxr-xr-x. 4 root root 175  1月 23 09:50 ..
	-rwxr--r--. 1 root root 389  1月 23 10:37 01_firstShell
	LS_STR02=總計 4
	drwxr-xr-x. 2 root root  27  1月 23 10:37 .
	drwxr-xr-x. 4 root root 175  1月 23 09:50 ..
	-rwxr--r--. 1 root root 389  1月 23 10:37 01_firstShell
	```
- `unalias <變量>`，釋放指令變量
	```bash
	[mickey@localhost Documents]$ unalias test
	[mickey@localhost Documents]$ test
	[mickey@localhost Documents]$
	```


### 環境變量
- 查看所有環境變數
	```bash
	[mickey@localhost Documents]$ env
	LANG=en_US_UTF-8
	HISTCONTROL=ignoredups
	(以下省略)
	```
- 環境變量路徑：/etc/profile
	```bash
	[root@localhost testShell]# vim /etc/profile
	```
- `export 變量名=變量值`，設置環境變量
	```shell
	#設置環境變量
	JAVA_HOME=/opt/jdk1.8.212
	export JAVA_HOME
	```

	```bash
	[root@localhost testShell]# echo $JAVA_HOME

	[root@localhost testShell]# source /etc/profile
	[root@localhost testShell]# echo $JAVA_HOME
	/opt/jdk1.8.212
	```

### 位置參數變量
當執行shell腳時，若希望獲取命令行的參數信息，就可以使用到位置參數變量(類似於java中的main參數)，如：`./myShell.sh 100 200`
1.	`$n`，n為數字，`$0`代表命令本身，`$1`~`$9`代表第一到第九個參數，第十以上的參數如`$W(12)`
2.	`$@`，代表命令行中所有的參數，不過`$@`把每個參數區分對待
3.	`$*`，代表命令行中所有的參數
```shell
#位置參數
echo "命令本身=$0"
echo "第一個參數=$1"
echo "顯示所有參數(分開)=$@"
echo "顯示所有參數(一起)=$*"
```

```bash
[root@localhost testShell]# ./01_firstShell 10 20 30
命令本身=./01_firstShell
第一個參數=10
顯示所有參數(分開)=10 20 30
顯示所有參數(一起)=10 20 30
```

### 預定義變量
就是shell設計者事先已經定義好的變量，可以直接在shell腳本中使用
1.	`$$`，當前進程號(PID)
2.	`$!`，後台運行的最後一個進程的進程號(PID)
3.	`$?`，最後一次執行的命令的返回狀態，若這個變量值為0，證明上一個命令正確執行；反正若不為0(具體是哪個數，由命令自己來決定)，則證明上一個命令執行不正確
```shell
#!/bin/bash
#預定義變量
echo "當前進程號=$$"
./01_firstShell 10 20 30 &
echo "最後的進程號=$!"
echo "執行的值=$?"
```

```bash
[root@localhost testShell]# ./02_ShellVar
當前進程號=16753
最後的進程號=16754
執行的值=0
(以下略)
```

## 🐧運算符
```shell
#!/bin/bash
#運算符
RESULT_01=$(((2 + 3) * 4))
echo "result_01=$RESULT_01"

#推薦方式
RESULT_02=$[(2 + 3) * 4]
echo "result_02=$RESULT_02"

TEMP=`expr 2 + 3`
RESULT_03=`expr $TEMP \* 4`
echo "result_03=$RESULT_03"
```

```bash
[root@localhost testShell]# ./02_ShellVar
result_01=20
result_02=20
result_03=20
```

## 🐧條件判斷
- 兩個整數比較
	1.	`=`，字符串比較
	2.	`lt`，小於
	3.	`le`，小於等於
	4.	`eq`，等於
	5.	`gt`，大於
	6.	`ge`，大於等於
	7.	`ne`，不等於
- 按照文件權限進行判斷
	1.	`-r`，有讀的權限
	2.	`-w`，有寫的權限
	3.	`-x`，有執行的權限
- 按照文件類型進行判斷
	1.	`-f`，文件存在并且是一個常規的文件
	2.	`-e`，文件存在
	3.	`-d`，文件存在并且是一個目錄

### 判斷命令執行結果
- `$?`，用於判斷前面的命令執行是否成功，回傳直0~255
	1. 0 --> 成功
	2. !=0 --> 失敗
	```bash
	[mickey@mickey Documents]$ ping -c3 192.168.11.111
	PING 192.168.11.111 (192.168.11.111) 56(84) bytes of data.

	--- 192.168.11.111 ping statistics ---
	3 packets transmitted, 0 received, 100% packet loss, time 65ms

	[mickey@mickey Documents]$ echo $?
	1
	[mickey@mickey Documents]$ ls ./hallo.sh
	./hallo.sh
	[mickey@mickey Documents]$ echo $?
	0
	```

### if ... then ...
`if [ condition ] then fi`
注意：condition前後要有空格
```shell
#!/bin/bash
#if
if [ "ok" = "ok" ]
then
		echo "ok equal ok"
fi

if [ 25 -gt 20  ]
then
		echo "25 > 20"
fi

if [ -e "./03_ShellIf" ]
then
		echo "./03_ShellIf 存在"
fi
```

```bash
[root@localhost testShell]# ./03_ShellIf
ok equal ok
25 > 20
./03_ShellIf 存在
```

### if ... then elif ... then ...
`if [ condition ] then elif [ condition ] then fi`
```shell
#if else
if [ $1 -ge 60 ]
then
        echo "$1 為及格分"
elif [ $1 -lt 60 ]
then
        echo "$1 不及格！"
fi
```

```bash
[root@localhost testShell]# ./03_ShellIf 65
ok equal ok
25 > 20
./03_ShellIf 存在
65 為及格分
```

### case
```shell
#case
case $1 in
"1")
        echo "周一"
;;
"2")
        echo "周二"
;;
"3")
        echo "周三"
;;
*)
        echo "其他"
;;
esac
```

```bash
[root@localhost testShell]# ./03_ShellIf 2
周二
```

## 🐧循環
### for
```shell
#!/bin/bash

#host1 host2 host3
#host{1,2,3}
for VAR in host{1..3}
do
        echo ${VAR}
done
```

- 也可使用`seq`指令結果來循環
	- `seq 1 5`，輸出1~5
	- `seq 2 2 10`，輸出2~10且間隔為2的數字(2、4、6、8、10)
	```shell
	#!/bin/bash

	for EVEN in $(seq 2 2 10)
	do
			echo ${EVEN}
	done
	```

```shell
#!/bin/bash

#for
for i in "$*"
do
        echo "num : $i"
done
echo "=========="

for i in "$@"
do
        echo "num : $i"
done
echo "=========="

#從1加總到100
SUM=0
for((i=1;i<=100;i++))
do
        SUM=$[$SUM + $i]
done
echo "sum = $SUM"
```

可以看出`$*`和`$@`的區別
```bash
[root@localhost testShell]# ./04_ShellFor 10 20 30 40
num : 10 20 30 40
==========
num : 10
num : 20
num : 30
num : 40
==========
sum = 5050
```

### while
```shell
#!/bin/bash

#while
SUM=0
TEMP=0
while [ $TEMP -le $1 ]
do
        SUM=$[$SUM + $TEMP]
        TEMP=$[$TEMP + 1]
done
echo "sum = $SUM"
```

```bash
[root@localhost testShell]# ./04_ShellFor 100
while sum = 5050
```

## 🐧函數
shell和其他編程語言一樣，有系統函數，也可以自定義函數

### 系統函數
- basename，返回文件名
- dirname，返回路徑
```bash
[root@localhost 下載]# basename /root/下載/testJdk.java
testJdk.java
[root@localhost 下載]# basename testJdk.java .java
testJdk
[root@localhost 下載]# dirname /root/下載/testJdk.java
/root/下載
```

### 自定義函數
```shell
#自定義函數
function getSum(){
        SUM=$[$NUM01 + $NUM02]
        echo "總和為 $SUM"
}

read -p "輸入NUM01:" NUM01
read -p "輸入NUM02:" NUM02
#調用function
getSum $NUM02 $NUM02
```

```bash
[root@localhost testShell]# ./06_ShellFunc
輸入NUM01:6
輸入NUM02:18
總和為 24
```
