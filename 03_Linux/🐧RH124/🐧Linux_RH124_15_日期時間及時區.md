# Linux 日期時間及時區
## 🐧data
`date`，顯示當前日期
```bash
[root@localhost 下載]# date
日  1月  5 20:40:21 CST 2020
[root@localhost 下載]# date +%Y
2020
[root@localhost 下載]# date +%m
01
[root@localhost 下載]# date +%d
05
[root@localhost 下載]# date "+%Y-%m-%d %H:%M:%S"
2020-01-05 20:41:41
[root@localhost 下載]#
```

`date -s "<年月日時分秒>"`，按格式顯示日期時間，參考：[date參數](https://man.linuxde.net/date)
```bash
[root@localhost 下載]# date -s "2020-01-05 20:44:59"
日  1月  5 20:44:59 CST 2020
[root@localhost 下載]# date
日  1月  5 20:45:01 CST 2020
```

## 🐧cal
`cal`，顯示當前日歷
```bash
[root@localhost 下載]# cal
      一月 2020
日 一 二 三 四 五 六
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31
```

`cal <年份>`，顯示當年月歷
```bash
[root@localhost 下載]# cal 2020
                               2020

        一月                   二月                   三月
日 一 二 三 四 五 六   日 一 二 三 四 五 六   日 一 二 三 四 五 六
          1  2  3  4                      1    1  2  3  4  5  6  7
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    8  9 10 11 12 13 14
12 13 14 15 16 17 18    9 10 11 12 13 14 15   15 16 17 18 19 20 21
19 20 21 22 23 24 25   16 17 18 19 20 21 22   22 23 24 25 26 27 28
26 27 28 29 30 31      23 24 25 26 27 28 29   29 30 31

        四月                   五月                   六月
日 一 二 三 四 五 六      日 一 二 三 四 五 六      日 一 二 三 四 五 六
          1  2  3  4                   1  2       1  2  3  4  5  6
 5  6  7  8  9 10 11    3  4  5  6  7  8  9    7  8  9 10 11 12 13
12 13 14 15 16 17 18   10 11 12 13 14 15 16   14 15 16 17 18 19 20
19 20 21 22 23 24 25   17 18 19 20 21 22 23   21 22 23 24 25 26 27
26 27 28 29 30         24 25 26 27 28 29 30   28 29 30
                       31
        七月                   八月                   九月
日 一 二 三 四 五 六      日 一 二 三 四 五 六      日 一 二 三 四 五 六
          1  2  3  4                      1          1  2  3  4  5
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    6  7  8  9 10 11 12
12 13 14 15 16 17 18    9 10 11 12 13 14 15   13 14 15 16 17 18 19
19 20 21 22 23 24 25   16 17 18 19 20 21 22   20 21 22 23 24 25 26
26 27 28 29 30 31      23 24 25 26 27 28 29   27 28 29 30
                       30 31
        十月                  十一月                 十二月
日 一 二 三 四 五 六      日 一 二 三 四 五 六      日 一 二 三 四 五 六
             1  2  3    1  2  3  4  5  6  7          1  2  3  4  5
 4  5  6  7  8  9 10    8  9 10 11 12 13 14    6  7  8  9 10 11 12
11 12 13 14 15 16 17   15 16 17 18 19 20 21   13 14 15 16 17 18 19
18 19 20 21 22 23 24   22 23 24 25 26 27 28   20 21 22 23 24 25 26
25 26 27 28 29 30 31   29 30                  27 28 29 30 31
```

# 時區時間校對
- /etc/localtime，時間設定檔(softlink)
	若要改時區的話，只需將/etc/localtime的soft link指向時區設定檔即可
	```bash
	[mickey@localhost ~]$ ll /etc/localtime
	lrwxrwxrwx. 1 root root 38 Mar  7 20:07 /etc/localtime -> ../usr/share/zoneinfo/America/New_York
	```
- /usr/share/zoneinfo，全球時區設定檔
- 時間會分為系統時間和硬體時間
	1. 系統時間為`date`指令下的時間
	2. 硬體時間為裝在硬體裡的計時器，當電腦啟動時會將系統時間設置和硬體時間一樣

## 🐧date
`date -s <YYYY-mm-dd HH:MM>`設置系統時間(日期和時間可拆開設置)
```bash
[root@localhost ~]$ date '+%Y/%m/%d %H:%M:%S'
2021/04/02 09:05:49
[root@localhost ~]# date -s 2021-04-01
Thu Apr  1 00:00:00 EAT 2021
[root@localhost ~]# date '+%Y/%m/%d %H:%M:%S'
2021/04/01 00:00:35
```

## 🐧時區選擇 timedatectl
此工具只有Red Hat才有
- `timedatectl`，查看時間、時區
	```bash
	[mickey@localhost ~]$ timedatectl
				   Local time: Fri 2021-04-02 00:52:53 EDT
			   Universal time: Fri 2021-04-02 04:52:53 UTC
					 RTC time: Fri 2021-04-02 04:52:53
					Time zone: America/New_York (EDT, -0400)
	System clock synchronized: no
				  NTP service: active
			  RTC in local TZ: no
	```
- `timedatectl list-timezones`，查看時區對照表
	```bash
	[mickey@localhost ~]$ timedatectl list-timezones
	Africa/Abidjan
	Africa/Accra
	Africa/Addis_Ababa
	Africa/Algiers
	Africa/Asmara
	```
- `timedatectl set-timezone <時區>`，時區設定
	```bash
	[mickey@localhost ~]$ timedatectl set-timezone Africa/Asmara
	==== AUTHENTICATING FOR org.freedesktop.timedate1.set-timezone ====
	Authentication is required to set the system timezone.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	```
- `timedatectl set-time <HH:MM:SS>`，設置時間(作業系統、硬體時間)
	```bash
	[mickey@localhost ~]$ date '+%Y/%m/%d %H:%M:%S'
	2021/04/02 08:15:42
	[mickey@localhost ~]$ timedatectl set-time 9:00:00
	==== AUTHENTICATING FOR org.freedesktop.timedate1.set-time ====
	Authentication is required to set the system time.
	Authenticating as: mickey
	Password:
	==== AUTHENTICATION COMPLETE ====
	[mickey@localhost ~]$ date '+%Y/%m/%d %H:%M:%S'
	2021/04/02 09:00:10
	```
- `timedatectl set-ntp <true|false>`，設置是否開啟與NTF Server同步

## 🐧時區選擇 tzselect
`tzselect`，比較傻瓜的方式設置時區
```bash
[root@localhost ~]# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
#? 4
Please select a country whose clocks agree with yours.
 1) Afghanistan           18) Israel                35) Palestine
 2) Armenia               19) Japan                 36) Philippines
 3) Azerbaijan            20) Jordan                37) Qatar
 4) Bahrain               21) Kazakhstan            38) Russia
 5) Bangladesh            22) Korea (North)         39) Saudi Arabia
 6) Bhutan                23) Korea (South)         40) Singapore
 7) Brunei                24) Kuwait                41) Sri Lanka
 8) Cambodia              25) Kyrgyzstan            42) Syria
 9) China                 26) Laos                  43) Taiwan
10) Cyprus                27) Lebanon               44) Tajikistan
11) East Timor            28) Macau                 45) Thailand
12) Georgia               29) Malaysia              46) Turkmenistan
13) Hong Kong             30) Mongolia              47) United Arab Emirates
14) India                 31) Myanmar (Burma)       48) Uzbekistan
15) Indonesia             32) Nepal                 49) Vietnam
16) Iran                  33) Oman                  50) Yemen
17) Iraq                  34) Pakistan
#? 43

The following information has been given:

        Taiwan

Therefore TZ='Asia/Taipei' will be used.
Selected time is now:   Fri Apr  2 21:58:50 CST 2021.
Universal Time is now:  Fri Apr  2 13:58:50 UTC 2021.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
        TZ='Asia/Taipei'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Taipei
```

## 🐧時間伺服器 chronyc
可參考：[chrony-系統校時工具](https://shazi.info/chrony-%E7%B3%BB%E7%B5%B1%E6%A0%A1%E6%99%82%E5%B7%A5%E5%85%B7%EF%BC%8C%E6%8A%8A%E9%81%8E%E6%99%82%E7%9A%84-ntpdate-%E4%B8%9F%E6%8E%89%E5%90%A7/)

硬體時間運行時間久了還是會出現飄移，因此chronyc是通過檢查chronyd 校時來源伺服器來修正時間飄移

- 設定檔路徑：/etc/chrony.conf
	- [台灣慣用公開時間伺服器](https://note.chiatse.com/tai-wan-guan-yong-gong-kai-shi-jian-si-fu-qi-ntp-server/)
		1. Cloudflare - NTP Server  `time.cloudflare.com`
		2. Apple - NTP Server(亞洲)  `time.asia.apple.com`
		3. Google - NTP Server  `time.google.com`
		4. 微軟Microsoft - NTP Server  `time.windows.com`
	- 修改設定檔後需要重新啟動服務，`systemctl restart chronyd`
- `chronyc sources -v`，查看NTP狀態
- 注意：
	1. NTP Server同步必須開啟
	2. 需要在設定檔設置上層時間伺服器，如：`server classroom.example.com iburst`
	3. `^*`，代表已與上層時間伺服器同步完成
	4. 時間誤差越大，同步時間就會越久；因為是慢慢把時間誤差拉近，以避免直接指定時間所帶來時間戳記問題
	```bash
	[root@localhost ~]# chronyc sources -v
	210 Number of sources = 0

	  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
	 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
	| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
	||                                                 .- xxxx [ yyyy ] +/- zzzz
	||      Reachability register (octal) -.           |  xxxx = adjusted offset,
	||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
	||                                \     |          |  zzzz = estimated error.
	||                                 |    |           \
	MS Name/IP address         Stratum Poll Reach LastRx Last sample
	===============================================================================
	^+ 211-22-103-157.HINET-IP.h     2  10   377   892    -70us[  -70us] +/-   37ms
	^* 118-163-81-63.HINET-IP.hi     2  10   377   903   -413us[ -414us] +/-   14ms
	^+ 118-163-81-61.HINET-IP.hi     2   9   377   268   +195us[ +195us] +/-   32ms
	^+ 211-22-103-158.HINET-IP.h     2  10   377   914   +402us[ +401us] +/-   30ms
	^+ 118-163-81-62.HINET-IP.hi     2  10   377   922   -477us[ -478us] +/-   32ms
	```
- `chronyc -c makestep`，立即校時
	```bash
	[root@localhost ~]# chronyc -c makestep
	200 OK
	```
