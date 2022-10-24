# Linux_RH294_02_inventory設備檔
- 將設備信息寫死，支援[INI-style](https://zh.wikipedia.org/wiki/INI%E6%96%87%E4%BB%B6)或YAML格式
- 使用程式動態生成設備信息：無論用什麼程式跑，返回值必須為[[🍀JSON]]格式
	1. 可以在github上下載別人已寫好的dynamically inventory程式-->[參考-github](https://github.com/ansible/ansible/tree/stable-2.9/contrib/inventory) 
	2. 若inventory有執行權限，則會視為dynamically inventory執行；反之則為靜態的inventory
	3. 開發dynamically inventory需要可以支援`--list`、`--host`等參數，詳細可參考：[ansible官方文件](https://docs.ansible.com/ansible/latest/dev_guide/index.html)

## 🐧基本寫法
可指定IP(IPv4、IPv6)和電腦名稱
```ini
# []-->代表群組，下邊列出此群組包含的機器名稱/IP
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com

# 沒有指定群組的機器
192.0.2.42

# 相同機器可出現多個群組
[testserver]
web1.example.com
db1.example.com

# [群組名:children]-->群組包含的子群組，下邊列出子群組名稱
[production:children]
webservers
dbservers

[rangeservers]
# IP區間，如下代表101~109
192.168.56.[101:109]
# 電腦名稱區間，如下代表vba~vbe和a~f
vb[a:e].dns.excample.com
2001:db8::[a:f]
```

## 🐧幫主機起別名
可能會忘記給機器起別名的原因而導致不易記憶，因此不建議使用
1. 在host_vars中新增以別名為檔名的檔案，文檔中指定`ansible_host:<主機名|主機IP>`
	```bash
	[mickey@vm104 test_ansible]$ tree
	.
	├── ansible.cfg
	├── ansible_facts
	├── group_vars
	│   └── vault
	└── host_vars
		├── 192.168.56.105
		└──  abc
	[mickey@vm104 test_ansible]$ cat ./host_vars/abc
	ansible_host: 192.168.56.106
	```
2. 在inventory設備檔加入主機別名
	```ini
	[myvm]
	192.168.56.105
	192.168.56.104
	abc
	```
3. 測試
	```bash
	[mickey@vm104 test_ansible]$ ansible abc -m ping
	abc | SUCCESS => {
		"ansible_facts": {
			"discovered_interpreter_python": "/usr/libexec/platform-python"
		},
		"changed": false,
		"ping": "pong"
	}
	```

## 🐧相關指令
### 檢查主機列表
`ansible <主機名|主機IP|群組名> --list-host`，查看指定主機名|主機IP|群組名是否在主機列表中
```bash
[mickey@vb101 ~]$ ansible localhost --list-host
  hosts (1):
    localhost
[mickey@vb101 ~]$ ansible test --list-host -i ~/ansible/test/inventory
  hosts (2):
    192.168.56.102
    192.168.56.103
```
1. `-i <設備檔路徑>`，查看指定設備檔
	- `ansible --version`可查看目前是看哪個設定檔(ansible.cfg)
	- 預設/etc/ansible/ansible.cfg設定的預設inventory為/etc/ansible/hosts
2. 不用宣告，ansible自帶群組
	- all，代表所有的機器
	- nogrouped，代表不屬於任何群組的機器
	```bash
	[mickey@vb101 test]$ ansible all --list-host
	  hosts (2):
		192.168.56.102
		192.168.56.103
	```

### 將設備檔轉為JSON
`ansible-inventory -i <設備檔路徑> --list`
```bash
[mickey@vm104 test_ansible]$ ansible-inventory -i inventory --list
{
    "_meta": {
        "hostvars": {
            "abc": {
                "ansible_host": "192.168.56.105"
            }
        }
    },
    "all": {
        "children": [
            "myvm",
            "ungrouped"
        ]
    },
    "myvm": {
        "hosts": [
            "192.168.56.104",
            "192.168.56.105",
            "abc"
        ]
    }
}
```


