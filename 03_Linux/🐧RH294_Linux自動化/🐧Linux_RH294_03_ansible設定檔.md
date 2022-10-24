# ansible.cfg設定檔
- ansible.cfg設定檔優先順序(由高-->低)：
	`ansible --version`可查看目前是看哪個設定檔
	1. ANSIBLE_CONFIG，宣告環境變數所指定的設定檔
	2. ./ansible.cfg，當前所在目錄下的設定檔
	3. ~/ansible.cfg
	4. /etc/ansible/ansible.cfg
- `ansible <主機名|主機IP|群組名> --list-host -v`，可看設定檔路徑
	```console
	[mickey@vb101 test]$ ansible test --list-host -v
	Using /home/mickey/ansible/test/ansible.cfg as config file
	  hosts (2):
		192.168.56.102
		192.168.56.103
	```
- 注釋語法(Linux通用規則)：`#`、`;`

## 🐧基本設定
- 范例參考：/etc/ansible/ansible.cfg
- `ansible-config dump`，查看默認ansible.cfg設置

### 登入及轉換用戶相關
- SSH設置用帳號+金鑰登入，參考：[[🐧Linux_RH124_13_SSH連線]]
- 設置指定用戶sudo不需要密碼，參考：[[🐧Linux_RH124_07_用戶管理]]
```ini
[defaults]
# inventory設備檔路徑
# 若./inventory為目錄，則./inventory/下所有文檔都為inventory設備檔
inventory = ./inventory
# 被控端登入用戶名
remote_user = mickey
# 被控端登入用戶是否需要輸入密碼
# 若不需要，要確認被控端是否SSH設置用帳號+金鑰登入
ask_pass = false

# 用於身份轉換
[privilege_escalation]
# 登入被控端後是否需要身份轉換
become = true
# 要用什麼方式身分轉換
become_method = sudo
# 要轉換成哪個用戶
become_user = root
# 身份轉換是否需要輸入密碼
# 若不需要，要確認被控端身分轉換設置不需要密碼
become_ask_pass = false
```

### 平行處理主機相關
```ini
# 一次幾台主機平行處理，默認為5
# 使用ansible指令執行可以用參數：-f或--forks
forks = 10
```

## 🐧不使用SSH連線設定
1. 預設ansible連線遠端是用SSH協議，但被控端為localhost則直接在本機以當前用戶執行，若是有身份轉換，也是用當前用戶進行轉換
2. 若需要本機連線用SSH，有兩個方法：
	- 將localhost加入inventory設備檔
	- 在ansible執行目錄建立host_vars目錄，在此目錄建立localhost檔案，檔案內容：
		```bash
		ansible_connection: smart #連線本機用SSH
		ansible_connection: local #連線本機不用SSH
		```

## 🐧其他設置
- 關閉呼叫ansible facts舊表達式寫法([[🐧Linux_RH294_05_Playbook變量]])：在ansible.cfg加上
	```ini
	[default]
	# True表示開啟舊表達式寫法，Falue表示關閉
	inject_facts_as_vars = True
	```
