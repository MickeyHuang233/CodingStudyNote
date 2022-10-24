# Linux_RH294_01_Ansible介紹
## 🐧介紹
- Ansible主要用於操作系統自動化布署和設置環境
- 優點：
	1. 被控端不需另外裝插件
	2. 可跨平台使用：Windows、Linux、UNIX
	3. playbook使用YAML格式管理環境配置，易於觀看
	4. 只要是人為的設定都可以寫成playbook來執行
	5. 可結合git進行版本控制
	6. 使用dynamic inventories可動態取得環境的機器列表
	7. 使用dynamic inventories可容易和其他系統連接
- Ansible管理示意圖
	![Linux_RH294_01_Ansible介紹_01_管理示意圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH294_Linux%E8%87%AA%E5%8B%95%E5%8C%96/images/Linux_RH294_01_Ansible%E4%BB%8B%E7%B4%B9_01_%E7%AE%A1%E7%90%86%E7%A4%BA%E6%84%8F%E5%9C%96.png?raw=true)
- 支援版本
	1. 主控端：Python 3(3.5或更新版本)或Python 2(2.7或更新版本)
		改版記錄可參考：[Red Hat Ansible Engine Life Cycle](https://access.redhat.com/support/policy/updates/ansible-engine)
	2. Linux、UNIX被控端：Python 3(3.5或更新版本)或Python 2(2.6或更新版本)
		若系統為fedora需要另外裝python3-dnf package
	3. Windows被控端：PowerShell 3.0或更新版本以及.NET Framework 4.0或更新版本
		Windows模組列表可參考：[Windows modules](https://docs.ansible.com/ansible/2.9/modules/list_of_windows_modules.html)
		Windows支援版本可參考：[Windows Guides](https://docs.ansible.com/ansible/latest/user_guide/windows.html)
- 支援的網絡設備
	1. Cisco IOS
	2. IOS XR
	3. NX-OS
	4. Juniper Junos
	5. Arista EOS
	6. VyOS-based networking devices
- 安裝ansible
	1. 練習需要先裝[EPEL](https://fedoraproject.org/wiki/EPEL)
		```bash
		yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
		```
	2. `yum install ansible`，安裝ansible套件
	3. `ansible --version`，檢查ansible版本
	```bash
	[mickey@vb101 test]$ ansible --version
	ansible 2.9.20
	  config file = /etc/ansible/ansible.cfg
	  configured module search path = ['/home/mickey/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
	  ansible python module location = /usr/lib/python3.6/site-packages/ansible
	  executable location = /usr/bin/ansible
	  python version = 3.6.8 (default, Oct 11 2019, 15:04:54) [GCC 8.3.1 20190507 (Red Hat 8.3.1-4)]
	```
- 建議使用的架構，易於管理
	```bash
	.
	├── db
	│   ├── ansible.cfg
	│   ├── inventory
	│   └── playbook.yml
	├── service
	│   ├── ansible.cfg
	│   ├── inventory
	│   └── playbook.yml
	└── test
		├── ansible.cfg
		├── inventory
		└── playbook.yml
	```

## 🐧常用標籤
- `name`，playbook中play的命名
- `hosts`，此play要執行的機器(主機名/IP/群組名)，[[🐧Linux_RH294_04_Playbook]]
- `vars`，定義變量，[[🐧Linux_RH294_05_Playbook變量]]
- `vars_files`，[[🐧Linux_RH294_05_Playbook變量]]
- `gather_facts:<yes|no>`，是否進行ansible facts採集，默認為yes，[[🐧Linux_RH294_05_Playbook變量]]
- `force_handler:<yes|no>`，設置就算task執行失敗，在失敗中斷前執行已經被呼叫handler，[[🐧Linux_RH294_04_Playbook]]
- `serial`，主機分批處理，[[🐧Linux_RH294_04_Playbook]]
- `import_playbook`，引入外部playbook，[[🐧Linux_RH294_04_Playbook]]
- `roles`，引入Role，[[🐧Linux_RH294_08_Role]]
- `tasks`，放要執行的行動
	- `name:<task名>`，task命名
	- `<module>`，task所使用的模組名稱，[[🐧Linux_RH294_09_常用Module介紹]]
	- `register:<變量名>`，[[🐧Linux_RH294_05_Playbook變量]]
	- `loop:<list>`，[[🐧Linux_RH294_04_Playbook]]
	- `when:<boolean>`，判斷為true才執行此task，[[🐧Linux_RH294_04_Playbook]]
	- `notify:<handler名>`，呼叫指定handler，[[🐧Linux_RH294_04_Playbook]]
	- `ignore_errors:<yes|no>`，忽略task執行錯誤，默認為no，[[🐧Linux_RH294_04_Playbook]]
	- `failed_when:<boolean>`，用於判斷此task是否執行錯誤，[[🐧Linux_RH294_04_Playbook]]
	- `changed_when:<boolean>`，用於判斷此task是否有執行，[[🐧Linux_RH294_04_Playbook]]
	- `block`、`rescue`、`always`，try...catch...finally，[[🐧Linux_RH294_04_Playbook]]
	- `import_tasks`，引入tasks(playbook一執行就加載)，[[🐧Linux_RH294_04_Playbook]]
	- `include_tasks`，引入tasks(執行到才加載)，[[🐧Linux_RH294_04_Playbook]]
	- `include_role`，將Role當task用
		- `name: <Role名>`，要引入Role名稱
- `handlers`，[[🐧Linux_RH294_04_Playbook]]
	- `name:<handler名>`

# 使用命令行執行 ansible
需要先完成：[[🐧Linux_RH294_02_inventory設備檔]]及[[🐧Linux_RH294_03_ansible設定檔]]設置

## 🐧執行模組
- `ansible -m <模組名> -a '[<參數1>=<值1> <參數2>=<值2> ...]' <機器名|機器IP|群組名>`，在指定機器執行指定模組
	1. `-m <模組名>`，要執行的模組
	2. `-a '[<參數1>=<值1> <參數2>=<值2> ...]'`，傳入模組的參數，無參數可省略，若有多個需要用單引號包起來
		- state=present，建立
		- state=absent，刪除
	3. `<機器名|機器IP|群組名>`，指定的機器/群組必須是inventory設備檔有指定的機器/群組
	```bash
	[root@vb101 test]# ansible -m ping all
	192.168.56.103 | SUCCESS => {
		"ansible_facts": {
			"discovered_interpreter_python": "/usr/libexec/platform-python"
		},
		"changed": false,
		"ping": "pong"
	}
	192.168.56.102 | UNREACHABLE! => {
		"changed": false,
		"msg": "Failed to connect to the host via ssh: mickey@192.168.56.102: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).",
		"unreachable": true
	}
	```
- 執行結果解讀
	1. changed：在執行前會檢查狀態是否符合，不符合才會執行
		- "changed": true，代表有執行
		- "changed": false，代表沒執行

## 🐧執行外部指令
通過模組**會先檢查**當前狀態是否符合需求，若已經符合則不會再執行，而且**會返回執行狀態**；而使用外部指令**不會檢查**就直接執行，而且都是**返回相同的狀態**，因此==不建議使用==
![Linux_RH294_01_Ansible介紹_02_執行外部命令](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH294_Linux%E8%87%AA%E5%8B%95%E5%8C%96/images/Linux_RH294_01_Ansible%E4%BB%8B%E7%B4%B9_02_%E5%9F%B7%E8%A1%8C%E5%A4%96%E9%83%A8%E5%91%BD%E4%BB%A4.png?raw=true)
- `-m command -a '<指令>'`，通過Python直接執行指令，不可使用環境變數
	1. `-o`，輸出結果顯示為一行
	```bash
	[root@vb101 test]# ansible all -m command -a 'hostname'
	192.168.56.103 | CHANGED | rc=0 >>
	vb103
	[root@vb101 test]# ansible all -m command -a 'hostname' -o
	192.168.56.102 | CHANGED | rc=0 | (stdout) vb103
	```
- `-m shell -a '<指令>'`，通過/bin/bash執行指令，所以可使用環境變數
	```bash
	[root@vb101 test]# ansible all -m shell -a 'hostname' -o
	192.168.56.102 | CHANGED | rc=0 | (stdout) vb103
	```
- `-m raw -a '<指令>'`，不通過Python直接執行指令
	```bash
	[root@vb101 test]# ansible all -m raw -a 'hostname' -o
	192.168.56.103 | CHANGED | rc=0 | (stdout) vb103\r\n (stderr) Shared connection to 192.168.56.103 closed.\r\n
	```

## 🐧設定參數
ansible.cfg設定檔的內容可用參數代入，不可複用-->CP值太低-->不建議使用

| ansible.cfg          | command-line option       |
| -------------------- | ------------------------- |
| inventory            | `-i`                      |
| remote_user          | `-u`                      |
| become=true          | `--become`或`-b`          |
| become_method        | `--bacome-method`         |
| become_user          | `--become-user`           |
| become_ask_pass=true | `--ask-become-pass`或`-K` |

```bash
[root@vb101 test]# ansible all -m ping -i /home/mickey/ansible/test/inventory -u mickey --become-method sudo --become-user root
192.168.56.103 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```
