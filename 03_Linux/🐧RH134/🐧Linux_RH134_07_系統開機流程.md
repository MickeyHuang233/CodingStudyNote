# Linux_RH134_07_系統開機流程
- 開機步驟可參考[[🗄️BIOS]]
- Linux開機步驟
	1. 開啟電源-->BIOS自檢程序(POST)-->呼叫int13h程序
	2. int13h呼叫寫在硬碟的boot louder(grub2)，boot louder寫在MBR最前面446 Bytes
	3. grub2將initramfs-xxx解壓縮至RAM
		並讀取/boot/loader/entries/\*.conf取得開機選項，用戶可在boot louder menu選擇要使用的kenel
	1. grub2執行vmlinu2-xxx(也就是kenel)
	2. vmlinu2-xxx mount RAM中的根目錄(/)
	3. vmlinu2-xxx執行init，實際上是執行systemd
		過去是init處理開機服務，改systemd後為避免修改kenel，所以折中方法是直接將init改為soft link，可參考：[[🐧Linux_RH124_10_服務管理]]
	7. systemd執行以下步驟：
		- `mount -o rw Root.DEVICE /sysroot`，將/sysroot掛載至硬碟
		- `chroot /sysroot`，變更根目錄至硬碟
		- 執行/usr/lib/systemd/systemd中的服務(開機啟動服務)
	![Linux_RH134_06_系統開機流程_01_Linux開機步驟](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_06_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_01_Linux%E9%96%8B%E6%A9%9F%E6%AD%A5%E9%A9%9F.png?raw=true)
- `poweroff`，關機，相當於`systemctl poweroff`
- `reboot`，重啟，相當於`systemctl reboot`
- `halt`，只會關作業系統，不會關機(很久以前關機用的)，相當於`systemctl halt`

## 🐧Linux運行級別
![Linux_RH134_07_系統開機流程_02_開機步驟](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_07_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_02_%E9%96%8B%E6%A9%9F%E6%AD%A5%E9%A9%9F.png?raw=true)
- [Linux系统的7个运行级别](https://www.jianshu.com/p/635e8480a75e)
- 類似於Windows的安全模式(**現在用其他方式**)
	- 0：關機
	- 1：單用戶(找回丟失密碼)
	- 2：多用戶狀態沒有關絡服務
	- 3：多用戶狀態有網絡服務【常用】
	- 4：系統未使用保留給用戶
	- 5：圖形界面【常用】
	- 6：系統重啟
- 系統的運行級別配置文件：/etc/inittab
- `init [0|1|2|3|5|6]`：切換到指定運行級別
	```bash
	init 3
	```

## 🐧Systemd Target
Systemd Target，開機界面
- 常用target
	1. graphical.target，圖形界面
	2. multi-user.target，文字界面
	3. emergency.target，系統救援用，root登入要password、initramfs已執行，但資料都還在ram(read only)
	4. rescue.target，系統救援用，開機程序基本完成，時間點比emergency.target後面
- 因為是systemd管理，所以文檔也是放在/lib/systemd/system，可參考：[[🐧Linux_RH124_10_服務管理]]
- `systemctl list-units --type=target`，列出所有target
	```bash
	[root@mickey ~]# systemctl list-units --type=target | head -n 5
	UNIT                   LOAD   ACTIVE SUB    DESCRIPTION
	basic.target           loaded active active Basic System
	cryptsetup.target      loaded active active Local Encrypted Volumes
	getty.target           loaded active active Login Prompts
	graphical.target       loaded active active Graphical Interface
	```
- `systemctl list-dependencies <界面名稱>.target | grep target`，查看target相依性
	```bash
	[root@mickey system]# systemctl list-dependencies graphical.target | grep target | head -n 5
	graphical.target
	● └─multi-user.target
	●   ├─basic.target
	●   │ ├─paths.target
	●   │ ├─slices.target
	```

### 開機默認界面
- `systemctl get-default`，取得默認界面
	```bash
	[root@mickey system]# systemctl get-default
	graphical.target
	```
- `systemctl set-default <界面名稱>.target`，設置默認界面
	```bash
	[root@mickey system]# systemctl set-default multi-user.target
	Removed /etc/systemd/system/default.target.
	Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target.
	[root@mickey system]# systemctl get-default
	multi-user.target
	```

### 開機時指定界面
不走默認開機界面，在開機時決定要執行的界面(僅限單次)。
1. 在boot loader menu，選擇第一項按E
	![Linux_RH134_07_系統開機流程_03_bootLoaderMenu](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_07_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_03_bootLoaderMenu.png?raw=true)
2. 在開頭為linux的那行最後加上`systemd.unit=<界面名稱>.target`
	按Ctrl + X確定修改並繼續開機流程
	![Linux_RH134_07_系統開機流程_01_bootLoaderMenu](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_07_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_01_bootLoaderMenu.png?raw=true)

## 🐧忘記root密碼操作流程
前提：initramfs、kenel沒壞掉才可這樣修
1. 在boot loader menu，選擇第一項按E
	![Linux_RH134_07_系統開機流程_03_bootLoaderMenu](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_07_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_03_bootLoaderMenu.png?raw=true)
2. 在開頭為linux的那行最後加上`rd.break`
	按Ctrl + X確定修改並繼續開機流程，此時就不用輸入root密碼就可以登入
	![Linux_RH134_07_系統開機流程_05_bootLoaderMenu](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH134/images/Linux_RH134_07_%E7%B3%BB%E7%B5%B1%E9%96%8B%E6%A9%9F%E6%B5%81%E7%A8%8B_05_bootLoaderMenu.png?raw=true)
3. `mount -o rw,remount /sysroot`，手動mount根目錄至硬碟-->可讀可寫
4. `chroot /sysroot`，變更根目錄至硬碟，使執行的指定用的是硬碟中的指令
5. `passwd`，變更root密碼
6. `touch /.autorelabel`，下次正常啟動時用於修復SELinux context
	- 在rd.break模式下SELinux尚未開啟，修改檔案會破壞SELinux標籤
	- SELinux開啟時會找/.autorelabel，執行`restorecon -RV`來修復SELinux context
	- 檔名存在就可以，內容可以為空

## 🐧開機除錯方式
1. 使用journalctl看log，但需要先設置將log存至硬碟，可參考：[[🐧Linux_RH124_14_Log查看]]
2. 使用debug shell
	- 要先開啟debug shell服務，`systemctl enable debug-shell.service`
	- 用root開啟TTY9(Ctrl + Alt + F9)
3. 使用emergency.target、rescue.target
	記得`mount -o rw,remount /`，才可以讀、寫
4. `systemctl list-jobs`，查看正在啟動的服務，可能有些服務啟動過程中有問題而卡住，以致後面服務無法啟動

## 🐧開機過程出錯程序
- 檔案系統損毀-->系統會等一段時間，若還是有問題則開啟emergency.target
- 找不到設備(/etc/fstab)-->系統會等一段時間，若還是有問題則開啟emergency.target
- 找不到mount point(/etc/fstab)-->直接開啟emergency.target
- mount參數寫錯(/etc/fstab)-->直接開啟emergency.target