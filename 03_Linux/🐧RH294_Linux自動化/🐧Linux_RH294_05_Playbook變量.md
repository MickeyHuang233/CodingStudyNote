# Linux_RH294_05_Playbook變量
- 變量名由字母、數字、下劃線組成，並以字母開頭
- 變量名重覆時的優先順序(高-->低)
	1. Global scope：command line
	`ansible-playbook <YAML路徑> -e "變量名1=變量值1, 變量名2=變量值2, ..."`
	```bash
	[mickey@vb101 test]$ sudo ansible-playbook testvar.yml -e "myvar='command var', myvar2=testvar"
	(省略)
	ok: [192.168.56.102] => {
		"msg": [
			"myvar = 'command var',",
			"'command var', in first"
		]
	}
	ok: [192.168.56.103] => {
		"msg": [
			"myvar = 'command var',",
			"'command var', in first"
		]
	}
	(省略)
	```
	3. Play scope
	4. Host scope
	5. Group scope

## 🐧變量基本用法
- 變量宣告`vars`
	```yaml
	- hosts: all
	  vars:
	    # 宣告一般變量
		myvar: this is myvar
		# 宣告多層變量
		user:
		  first_name: mickey
		  last_name: huang
		  home_dir: /home/mickey
	```
- 變量引入`vars_files`
	```yaml
	- hosts: all
	  vars_files:
		- vars/users.yml
	```
- 變量呼叫`{{ varname }}`、`"{{ varname }}"`
	```yaml
	---
	- name: test var
	  hosts: localhost
	  vars:
	   myvar: this is myvar
	  tasks:
		- name: debug myvar
		  debug:
			msg:
			  - myvar = {{myvar}}
			  # 若變量在一行的開始使用，需要用""或''包起來
			  - "{{myvar}} in first"
		      # 呼叫多層變量
			  - user first name = {{user.name.first_name}}
			  - user last name = {{user['name']['last_name']}}
	...
	```
	輸出：
	```bash
	ok: [localhost] => {
		"msg": [
			"myvar = this is myvar",
			"this is myvar in first"
		]
	}
	```

## 🐧執行結果傳入變量 register
`register: 變量名`
```yaml
---
- name: test moudel return var
  hosts: localhost
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: latest
      # 執行結果存入變量install_result
      register: install_result

    - debug: var=install_result
...
```
輸出：
```bash
(省略)
TASK [debug] *******************************************************************
ok: [localhost] => {
    "install_result": {
        "changed": true,
        "failed": false,
        "msg": "",
        "rc": 0,
        "results": [
            "Installed: httpd-2.4.37-30.module+el8.3.0+7001+0766b9e7.x86_64",
            "Installed: httpd-filesystem-2.4.37-30.module+el8.3.0+7001+0766b9e7.noarch",
            "Installed: mod_http2-1.15.7-2.module+el8.3.0+7670+8bf57d29.x86_64",
            "Installed: redhat-logos-httpd-81.1-1.el8.noarch",
            "Installed: httpd-tools-2.4.37-30.module+el8.3.0+7001+0766b9e7.x86_64",
            "Installed: apr-1.6.3-11.el8.x86_64",
            "Installed: apr-util-bdb-1.6.1-6.el8.x86_64",
            "Installed: apr-util-1.6.1-6.el8.x86_64",
            "Installed: apr-util-openssl-1.6.1-6.el8.x86_64"
        ]
    }
}
(省略)
```

## 🐧主機及群組變數
### 主機變數 Host scope
- 將變數設定inventory設備檔中，`主機名 變量名=變量值`
	```ini
	[test]
	192.168.56.102 myvar=102user
	192.168.56.103
	```
- 外部宣告主機變數，./host_vars/主機名，`變量名: 變量值`
	```bash
	.
	├── ansible.cfg
	├── host_vars
	│   └── 192.168.56.102
	├── inventory
	└── playbook.yml
	```
	192.168.56.102文檔內容：
	```ini
	myvar: my host var in other file
	```

### 群組變數 Group scope
- 將變數設定inventory設備檔中，`[群組名:vars]`
	```ini
	[test]
	192.168.56.102 myvar=102user
	192.168.56.103

	[test:vars]
	myvar=test vars
	```
	輸出：
	```ini
	ok: [192.168.56.102] => {
		"msg": [
			"myvar = 102user",
			"102user in first"
		]
	}
	ok: [192.168.56.103] => {
		"msg": [
			"myvar = test vars",
			"test vars in first"
		]
	}
	```
- 外部宣告群組變數，./group_vars/群組名/\*.yml，`變量名: 變量值`
	```bash
	.
	├── ansible.cfg
	├── group_vars
	│   └── testgroup
	├── inventory
	└── playbook.yml
	```
	testgroup文檔內容：
	```ini
	myvar: my testgroup var on other file
	```

## 🐧變數加密 ansible-vault
用於宣告較為敏感的變數，如：建立帳戶的帳戶密碼

### ansible-vault基本操作
- vault變數檔中的變數宣告格式和之前一樣
	```yaml
	mypasswd: redhat
	```
- `ansible-vault create <加密檔路徑>`，建立加密檔，使用默認編輯器開啟(vi)
	1. 若需要更改默認編輯器可在環境變數指定：`export EDITOR=nano`
	2. 過去版本使用AES256加密，現在則是用128-bit AES加密
	3. `--vault-password-file <密碼檔路徑>`，可將密碼寫至檔案，要用再讀
		以下關於ansible-vault操作都可以用此參數
	```bash
	[mickey@vm104 test_ansible]$ ansible-vault create secret.yml
	New Vault password:
	Confirm New Vault password:
	```
- `ansible-vault view <加密檔路徑>`，查看加密檔內容
	```bash
	[mickey@vm104 test_ansible]$ ansible-vault view ./secret.yml
	Vault password:
	mypasswd: redhat
	```
- `ansible-vault edit <加密檔路徑>`，編輯加密檔
- `ansible-vault encrypt <文檔路徑> [文檔路徑...]`，加密文檔(可指定多個)
	1. `--output=<另存路徑>`，另存指定路徑
	```bash
	[mickey@vm104 test_ansible]$ ansible-vault encrypt --output=./test1_secret.yml test1.yml
	```
- `ansible-vault decrypt <文檔路徑> [文檔路徑...]`，解密文檔(可指定多個)
	1. `--output=<另存路徑>`，另存指定路徑
	```bash
	[mickey@vm104 test_ansible]$ ansible-vault decrypt test1.yml test2.yml
	```
- `ansible-vault rekey <文檔路徑>`，修改密碼
	```bash
	[mickey@vm104 test_ansible]$ ansible-vault rekey test1_secret.yml
	```

### playbook引用vault變數檔
- playbook引用
	```yaml
	- name: test my playbook
	  hosts:
	  - all

	  # 使用外部引用加密檔
	  vars_files:
	  - ./secret.yml

	  tasks:
	  - name: debug secret passwd
		debug:
		 msg: "{{ mypasswd }} is my password"
	  - name: create a user
		user:
		  state : present
		  name: testuser
		  password : "{{ mypasswd }}"
		  group: student
	...
	```
- `--vault-id @prompt`，提示輸入密碼
	```bash
	[mickey@vm104 test_ansible]$ ansible-playbook --vault-id @prompt playbook.yml
	```
	1. 可加多個密碼都不一樣的加密參數檔；但提示所輸入的密碼沒有前後順序，只要對了就可以了
	```bash
	[mickey@vm104 test_ansible]$ ansible-playbook --vault-id one@prompt \
	> --vault-id two@prompt playbook.yml
	Vault password (one):
	Vault password (two):
	```
- `--vault-password-file <密碼檔路徑>`，讀密碼檔的密碼
	```bash
	[mickey@vm104 test_ansible]$ ansible-playbook --vault-password-file mypasswd playbook.yml
	```
- 建議將vault變數檔另外放在group_vars或host_vars，便於管理
	```bash
	.
	├── ansible.cfg
	├── group_vars
	│   └── vault
	│       └── secret.yml
	├── host_vars
	│   ├── 192.168.56.105
	│   └── vault
	│       └── secret.yml
	├── inventory
	└── playbook.yml
	```

## 🐧ansible facts
在playbook模組前一定會先執行的task，用於收集機器信息
實際上是執行`ansible <主機名|IP> -m setup`，回傳為[[🍀JSON]]
```bash
[mickey@vm104 test_ansible]$ ansible 192.168.56.105 -m setup | head
192.168.56.105 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.56.105",
            "192.168.56.106",
            "192.168.122.1",
            "10.0.2.15"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::157d:5665:55e7:24e2",
```

### 相關設置
- 關閉舊表達式寫法：在ansible.cfg加上
	```ini
	[default]
	# True表示開啟舊表達式寫法，Falue表示關閉
	inject_facts_as_vars = True
	```
- 關閉採集ansible facts：在[[🐧Linux_RH294_04_Playbook]]加上`gather_facts: no`
	```yaml
	- name: test my playbook
	  hosts: all
	  gather_facts: no
	  tasks:
	```

### 呼叫變數
- 基本規則：以`ansible_facts`開頭，後面變量名省略`ansible_`
	1. 新表示法：`{{ ansible_facts['default_ipv4']['address'] }}`
	2. 舊表示法：`{{ ansible_facts.default_ipv4.address }}`
- 呼叫List，從0開始計算
	1. 新表示法：`{{ ansible_facts['device_links']['ids']['sda'][0] }}`
	2. 舊表示法：`{{ ansible_facts.device_links.ids.sda[0] }}`

### 客製facts採集
1. 在被控端主機建立/etc/ansible/facts.d/\*.fact
	- 靜態變量：直接指定鍵值對
	```ini
	[package]
	web_package = httpd
	db_package = maria

	[users]
	user1 = jack
	user2 = vance
	```
	- 變量語script産出：無論用什麼程式，産出格式必須為[[🍀JSON]]
	注意：需要有執行的權限
	```bash
	#! /bin/bash
	echo '{"date":"'$(date +%F)'"}'
	```
	```bash
	[mickey@vm105 facts.d]$ ll
	total 8
	-rw-r--r--. 1 root root 86 May 12 07:53 custom.fact
	-rwx------. 1 root root 45 May 12 08:27 script.fact
	```
2. 在主控端執行`ansible <主機名|IP> -m setup`或執行playbook.yml
	```bash
	[mickey@vm104 test_ansible]$ ansible 192.168.56.105 -m setup
	(省略)
	"ansible_local": {
		"custom": {
			"package": {
				"db_package": "maria",
				"web_package": "httpd"
			},
			"users": {
				"user1": "jack",
				"user2": "vance"
			}
		},
		"script": {
			"date": "2021-05-12"
		}
	},
	(省略)
	```
3. `{{ ansible_facts['ansible_local'] }}`，在playbook引用，==注意這裡的ansible_不可省略==

## 🐧魔法變數 magic variables
除ansible facts外的自帶變數
- hostvars
	```bash
	[mickey@vm104 test_ansible]$ ansible localhost -m debug -a 'var=hostvars'
	localhost | SUCCESS => {
		"hostvars": {
			"192.168.56.105": {
				"ansible_check_mode": false,
				"ansible_diff_mode": false,
				"ansible_facts": {},
				"ansible_forks": 5,
				"ansible_inventory_sources": [
					"/home/mickey/ansible/test_ansible/inventory"
				],
				"ansible_playbook_python": "/usr/bin/python3.6",
				"ansible_verbosity": 0,
	(省略)
	```
- group_names
	```bash
	[mickey@vm104 test_ansible]$ ansible all -m debug -a 'var=group_names'
	192.168.56.105 | SUCCESS => {
		"group_names": [
			"myvm"
		]
	}
	```
- groups，inventory設備檔中的群組名稱
	```bash
	[mickey@vm104 test_ansible]$ ansible all -m debug -a 'var=groups'
	192.168.56.105 | SUCCESS => {
		"groups": {
			"all": [
				"192.168.56.105"
			],
			"myvm": [
				"192.168.56.105"
			],
			"ungrouped": []
		}
	}
	```
- inventory_hostname
	```bash
	[mickey@vm104 test_ansible]$ ansible all -m debug -a 'var=inventory_hostname'
	192.168.56.105 | SUCCESS => {
		"inventory_hostname": "192.168.56.105"
	}
	```