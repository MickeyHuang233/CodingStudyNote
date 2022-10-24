# Linux_RH294_08_Role
- 類似於API，別人寫好的腳本，只需要給相應參數調用即可
- Role存放目錄(可用`ansible-galaxy list`查看)
	1. <用戶家目錄>/.ansible/roles
	2. /usr/share/ansible/roles
	3. /etc/ansible/roles
	4. 以上路徑都找不到則找./roles
- 取得Role方式：
	1. 自行編寫Role
	2. Red Hat官方Role：rhel-system-roles
	3. 至[galaxy ansible](https://galaxy.ansible.com/)下載
- 在Playbook調用Role
	建議編寫順序：宣告Role變數-->呼叫Role

## 🐧文件架構
```bash
.
├── ansible.cfg
├── ansible_facts
├── group_vars
├── host_vars
├── inventory
├── playbook.yml
└── roles						# 一般都會將Role放在roles/底下
    └── rhel-system-roles.selinux
        ├── README.md			# 一般都會有說明文件
        ├── defaults			# 主要用於宣告默認變數
        │   └── main.yml
        ├── files				# 用於放置要複製至被控端的文件
        ├── handlers			# handler存放目錄
        │   └── main.yml
        ├── meta				# 存放相依的Role，相依Role會先執行，執行完再執行自己
        │   └── main.yml
        ├── tasks				# 放置Role要執行的步驟(相當於playbook的tasks)
        │   └── main.yml
        ├── templates			# 放template.j2
        │   └── templates.j2
        ├── tests				# 測試用(可忽略)
        │   ├── inventory
        │   └── main.yml
        └── vars				# 變數宣告，優先程變比defaults高
            └── main.yml
```

## 🐧呼叫Role
- 變量宣告在`vars`
	```yaml
	---
	- name: test import role
	  hosts: 192.168.56.104
	  vars:
		selinux_booleans:
		- name: 'httpd_enable_homedirs'
		  state: 'on'
		  persistent: 'yes'
	  roles:
		- rhel-system-roles.selinux
		- rhel-system-roles.timesync
	...
	```
- 變量指定在要調用的Role底下
	```yaml
	---
	- name: test import role
	  hosts: 192.168.56.104
	  roles:
		- role: rhel-system-roles.selinux
		  selinux_booleans:
			- name: 'httpd_enable_homedirs'
			  state: 'on'
			  persistent: 'yes'
		- role: rhel-system-roles.timesync
	...
	```
- 變量和Role寫在同一行，不建議使用
	```yaml
	---
	- name: test import role
	  hosts: 192.168.56.104
	  roles:
		- role: myrole1
		- { role: myrole2, var1: myvar1, var2: myvar2 }
	...
	```

## 🐧rhel-system-roles
rhel-system-roles為Red Hat官方提供的Role
- Red Hat Enterprise Linux7.4之後才有此套件
- `yum install rhel-system-roles`，安裝套件
- 套件的role都放在/usr/share/ansible/roles/(可用`rpm -ql rhel-system-roles`查看)
	注意：基本上用soft link為引用role的名稱，這樣更新role就不用改playbook，只要更新soft link就可以了
	```bash
	[mickey@vm104 test_ansible]$ ll /usr/share/ansible/roles/
	total 56
	lrwxrwxrwx.  1 root root   29 Mar 19 04:36 linux-system-roles.certificate -> rhel-system-roles.certificate
	lrwxrwxrwx.  1 root root   33 Mar 19 04:36 linux-system-roles.crypto_policies -> rhel-system-roles.crypto_policies
	(省略)
	drwxr-xr-x. 10 root root 4096 May 19 23:42 rhel-system-roles.certificate
	drwxr-xr-x.  8 root root 4096 May 19 23:42 rhel-system-roles.crypto_policies
	(省略)
	```
- 無論是Red Hat官方或外部下載的Role，都會有README.md說明文件，主要說明相關的參數如何設置
- Red Hat 8所提供的Role及其版本
	1. rhel-system-roles.kdump，完整正式版
	2. rhel-system-roles.network，完整正式版
	3. rhel-system-roles.selinux，完整正式版
	4. rhel-system-roles.timesync，完整正式版
	5. rhel-system-roles.postfix，mail server，Beta版(不建議使用)
	6. rhel-system-roles.firewall，開發中(不建議使用)
	7. rhel-system-roles.tuned，開發中(不建議使用)

### rhel-system-roles.selinux
相關指令設定可參考：[[🐧Linux_RH134_05_SELinux]]
- 作用：
	1. 修改工作模式
	2. 執行`restorecon`
	3. 設置應用程式權限啟用，`setsebool`
	4. 設置文件上下文，`semanage fcontext`
	5. 設置SELinux中的用戶(和Linux中的用戶不一樣)
```yaml
---
- name: test SELinux role
  hosts: 192.168.56.104
  # role變數宣告，可查看README.md
  vars:
    # 修改SELinux boolean資料庫資料
    selinux_booleans:
        # 要修改權限的權限名稱
      - name: 'httpd_enable_homedirs'
        # 修改權限狀態
        state: 'on'
        # 是否回寫資料庫，setsebool -P
        persistent: 'yes'
    # SELinux context資料庫新增資料
    selinux_fcontexts:
      - target: '/var/www(/.*)?'
        setype: 'httpd_sys_content_t'
        state: 'present'
    # 執行restorecon的目錄
    selinux_restore_dirs:
      - /var/www
    # SELinux port資料庫新增資料
    selinux_ports:
      - ports: '82'
        setype: 'http_port_t'
        proto: 'tcp'
        state: 'present'

  tasks:
    - name: apply SELinux role
      block:
        # 把Role當作task用
        # 若有修改SELinux的工作模式，則需要重啟
        - name: include SELinux role
          include_role:
            name: rhel-system-roles.selinux
      rescue:
        # black中的Role執行完後，若需要重新開機，selinux_reboot_required為true
        - name: check for failue for other resons than required reboot
          fail:
          when: not selinux_reboot_required
        # 重啟遠端機器，重啟完成後繼續執行
        - name: restart managed host
          reboot:
        # 再執行一次SELinux Role
        - name: reapply SELinuxrole to complete changes
          include_role:
            name: rhel-system-roles.selinux
...
```

### rhel-system-roles.timesync
timesync只能設置時間，不能設置時區
```yaml
---
- name: test role tkmesync
  hosts: 192.168.56.104
  vars:
    # 設置NTP Server位置
    timesync_ntp_server:
      - hostname: time.stdtime.gov.tw
        iburst: yes
      - hostname: time.stdtime.gov.tw
        iburst: yes
      - hostname: time.stdtime.gov.tw
        iburst: yes
    timezone: "Asia/Taipei"

  roles:
    - rhel-system-roles.timesync

  tasks:
    # 設定時區
    - name: set timezone
      timezone:
        # 時區字串可用tzselect或timedatectl找
        name: "{{ timezone }}"
...
```

### rhel-system-roles.network
```yaml
---
- name: test rhel network role
  hosts:
    - mylocal
  vars:
    network_connections:
      - name: test_connection
        persistent_state: present
        autoconnet: yes
        type: ethernet
        interface_name: enp0s8
        mac: 08:00:27:A2:4F:D8
        ip:
          address:
            - 192.168.56.233/24
  roles:
    - linux-system-roles.network
...
```

## 🐧ansible-galaxy Role
可至網站(圖形界面)下載：[galaxy ansible](https://galaxy.ansible.com/)

### 查詢已下載Role
- `ansible-galaxy list`
	```bash
	[mickey@vm104 test_ansible]$ ansible-galaxy list
	# /home/mickey/.ansible/roles
	# /usr/share/ansible/roles
	- linux-system-roles.certificate, (unknown version)
	- linux-system-roles.crypto_policies, (unknown version)
	(省略)
	- rhel-system-roles.tlog, (unknown version)
	# /etc/ansible/roles
	```
- `ansible-galaxy list -p <Role庫路徑>`
	```bash
	[mickey@vm104 roles]$ ansible-galaxy list -p /home/mickey/ansible/roles
	```

### 線上查找Role
- `ansible-galaxy search '<關鍵字>' --platorms EL`，按關鍵字搜索
	1. `--platorms EL`，表示適用於企業Linux(EL)平台的角色的名稱
	2. `--author`，指定作者
	3. `--galaxy-tags`，指定標籤
	```bash
	[mickey@vm104 ~]$ ansible-galaxy search 'redis' --platforms EL | head -n 10
	Found 235 roles matching your search:
	 Name                                            Description
	 ----                                            -----------
	 0x0i.consul                                     Consul - a service discovery, mesh and configuration control plane and networking tool
	 0x0i.grafana                                    Grafana - an analytics and monitoring observability platform
	 0x5a17ed.ansible_role_netbox                    Installs and configures NetBox, a DCIM suite, in a production setting.
	 1it.sudo                                        Ansible role for managing sudoers
	 adfinis-sygroup.redis                           Ansible role for Redis
	```
- `ansible-galaxy info <Role名>`，查看Role詳細信息
	```bash
	[mickey@vm104 ~]$ ansible-galaxy info adfinis-sygroup.redis
	Role: adfinis-sygroup.redis
			description: Ansible role for Redis
			active: True
			commit: 89c580b8a7ff511300dd6dbf186cad279f6613b0
			commit_message: Merge pull request #4 from in0rdr/fix/owner
	(省略)
	```

### 下載Role
#### 下載單個Role
`ansible-galaxy install <Role名> -p <下載路徑>`，下載Role至指定路徑
- `-p <下載路徑>`，下載至指定路徑
```bash
[mickey@vm104 ~]$ ansible-galaxy install adfinis-sygroup.redis -p ./ansible/test_ansible/roles/
- downloading role 'redis', owned by adfinis-sygroup
- downloading role from https://github.com/adfinis-sygroup/ansible-role-redis/archive/master.tar.gz
- extracting adfinis-sygroup.redis to /home/mickey/ansible/test_ansible/roles/adfinis-sygroup.redis
- adfinis-sygroup.redis (master) was installed successfully
```

#### 批次下載Role
1. 編寫批次檔(.yml)
	- 從Ansible Galaxy下載
	- 用git下載，HTTPS/SSH
	- 本機
	```yaml
	# from Ansible Galaxy, using the latest version
	- src: geerlingguy.redis

	# from Ansible Galaxy, overriding the name and using a specific version
	- src: geerlingguy.redis
	  # 指定版本
	  version: "1.5.0"
	  # 下載的目錄名稱
	  name: redis_prod

	# from any Git-based repository, using HTTPS
	- src: https://github.com/hsoft/ansible-acme-nginx.git
	  # 用git下載role
	  scm: git
	  version: master
	  name: nginx-acme

	# from any Git-based repository, using SSH
	- src: git@github.com:hsoft/ansible-acme-nginx.git
	  scm: git
	  version: master
	  name: nginx-acme-ssh

	# from a role tar ball, given a URL
	# Role已經在本機
	- src: file:///opt/local/roles/myrole.tar
	  name: myrole
	```
1. `ansible-galaxy install -r <批次檔路徑> -p <下載路徑>`
	- `-r`，指定批次檔
		```bash
		[mickey@vm104 roles]$ ansible-galaxy install -r ./requirements.yml -p .
		```

### 移除Role
- `ansible-galaxy remove <Role名> -p <Role庫路徑>`，移除role，相當於rm
	1. `-p`，指定Role庫路徑
	```bash
	[mickey@vm104 roles]$ ls
	geerlingguy.redis  redis_prod  requirements.yml
	[mickey@vm104 roles]$ ansible-galaxy remove geerlingguy.redis -p .
	- successfully removed geerlingguy.redis
	[mickey@vm104 roles]$ ls
	redis_prod  requirements.yml
	```


## 🐧自行編寫Role
### 初始化Role
`ansible-galaxy init <Role路徑>`，建立Role基本架構
```bash
[mickey@vm104 test_ansible]$ ansible-galaxy init ./roles/my_first_role
- Role ./roles/my_first_role was created successfully
[mickey@vm104 test_ansible]$ tree ./roles
./roles
└── my_first_role
	├── defaults
	│   └── main.yml
	├── files
	├── handlers
	│   └── main.yml
	├── meta
	│   └── main.yml
	├── README.md
	├── tasks
	│   └── main.yml
	├── templates
	├── tests
	│   ├── inventory
	│   └── test.yml
	└── vars
		└── main.yml

9 directories, 8 files
```

### template
- tasks/main.yml，\*.j2會自動在templates找
	template.j2編寫，可參考：[[🐧Linux_RH294_07_Jinja2 Templates]]
	```bash
	[mickey@vm104 test_ansible]$ cat ./roles/myrole/tasks/main.yml
	---
	# tasks file for myrole---
	- name: template reader
	  template:
		# 會自動在templates找
		src: host.j2
		dest: /home/mickey/test_templates_hosts.txt
		owner: mickey
		group: student
		mode: 0644
	  register: result
	- name: print result
	  debug:
		var: result
	...
	```
- `roles/myrole/templates/*.j2`
	```bash
	[mickey@vm104 test_ansible]$ cat roles/myrole/templates/host.j2
	{# 使用魔法變數迴圈 #}
	{% for myhost in groups['all'] %}
	{{ hostvars[myhost]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[myhost]['ansible_facts']['fqdn'] }} {{ hostvars[myhost]['ansible_facts']['hostname'] }}
	{% endfor %}
	```
- playbook.yml
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_role_myrole.yml
	---
	- name: test import myrole
	  hosts: all
	  # gather_facts: no
	  roles:
		- myrole
	...
	```

### 相依Role
meta/main.yml，`dependencies`
```bash
[mickey@vm104 test_ansible]$ cat roles/myrole/meta/main.yml
dependencies:
  - role: rhel-system-roles.timesync
    timesync_ntp_server:
      - hostname: time.stdtime.gov.tw
        iburst: yes
      - hostname: time.stdtime.gov.tw
        iburst: yes
      - hostname: time.stdtime.gov.tw
        iburst: yes
    timezone:
        name: "Asia/Taipei"
- role: my_other_role
```

