# 常用Module介紹
## 🐧套件安裝相關
### yum
原始指令參考：[[🐧Linux_RH124_17_套件管理]]
```yaml
---
- name: test yum module
  hosts:
    - all
  vars:
    yum_list:
      - httpd
      - mariadb
	  # 安裝group
      - '@Development Tools'
	  # 安裝module
	  #- '@perl:5.26/minimal'

  tasks:
    - name: install software
      yum:
	    # 也可用'*'，代表全部
        name: "{{ yum_list }}"
		# absent刪除
		# latest安裝最新版本(在name指定)
		# present安裝指定版本(在name指定)
        state: present
...
```

### yum_repository
- 注冊Yum Server，/etc/yum.repos.d/\*.repo
	```yaml
	---
	- name: test yum_repository module
	  hosts:
	   - mylocal

	  tasks:
		- name: add repository
		  yum_repository:
			name: epel_test
			description: test_add_repository
			file: external_repos
			baseurl: http://materials.example.com/yum/repository/
			enable: yes
			gpgcheck: no
	...
	```
	産出結果
	```bash
	[mickey@vm100 ansible]$ sudo cat /etc/yum.repos.d/external_repos.repo
	[epel_test]
	baseurl = http://materials.example.com/yum/repository/
	enabled = 1
	gpgcheck = 0
	name = test_add_repository
	```
- 刪除repository
	```yaml
	---
	- name: test yum_repository module
	  hosts:
		- mylocal

	  tasks:
		- name: remove repository
		  yum_repository:
			file: external_repos
			name: epel_test
			state: absent
	...
	```

### rpm_key
`rpm -import gpgkey`
```yaml
# (前略)
  tasks:
    - name: import gpgkey
      rpm_key:
        state: present
        key: http://materials.example.com/yum/repository/RPM-GPG-KEY-example
```

### package_facts
自動取得系統的套件管理工具：
- yum --> Red Hat
- apt --> Ubunto

```yaml
---
- name: get package_facts
  hosts:
    - all

  tasks:
    - name: turn on auto get package_facts
      package_facts:
        manager: auto
    - name: debug ansible_facts.package
      debug:
        var: ansible_facts.packages
...
```

### package
通用的套件管理工具，會自動判斷系統
```ymal
---
- name: test package module
  hosts:
    - all

  tasks:
    - name: intall httpd
      package:
        name:
          - httpd
        state: present
...
```

## 🐧用戶管理相關
### user
- 建立/修改用戶
	```yaml
	---
	- name: test user module playbook
	  hosts:
		- all
	  vars:
		new_user: john
		new_passwd: redhat

	  tasks:
		- name: manage a user
		  user:
			name: "{{ new_user }}"
			# 指定hash加密密碼寫至/etc/passwd，必須這麼指定
			password: "{{ new_passwd | password_hash('sha512') }}"
			# 指定用戶要用的shell
			shell: /bin/bash
			# 主要群組
			group: testTeam
			# 次要群組
			groups:
			  - student
			  - mickey
			# 設置次要群組怎麼加
			# yes，原來次要群組 + 新的次要群組
			# no，原來的次要群組清空後加新的次要群組
			append: yes
	...
	```
- 刪除用戶
	```yaml
	---
	- name: test remove user by user module
	  hosts:
		- all
	  vars_files:
		- ./vars/user_vars

	  tasks:
		- name: do remove user
		  user:
			name: "{{ remove_user }}"
			state: absent
			# 是否刪除用戶家目錄
			remove: yes
	...
	```
- 産生ssh金鑰
	注意：若無此用戶，則會新增用戶
	```yaml
	---
	- name: generate user ssh key by user module
	  hosts:
		- mylocal
	  vars_files:
		- ./vars/user_vars
	  gather_facts: false

	  tasks:
		- name: do gen sshkey
		  user:
			name: "{{ modify_user }}"
			generate_ssh_key: yes
			ssh_key_bits: 2048
			ssh_key_file: .ssh/id_rsa
	...
	```

### group
```yaml
---
- name: test group module
  hosts:
    - all
  vars:
    add_group: test_group
  gather_facts: false

  tasks:
    - name: add group
      group:
        name: "{{ add_group }}"
		# present，新增
		# absent，刪除
        state: present
...
```

### known_hosts
相當於指令：`key-scan <hostname|IP> > /etc/ssh/ssh_know_hosts`
```yaml
- name: tell the host about our servers it might want to ssh to
  known_hosts:
    path: /etc/ssh/ssh_known_hosts
    name: foo.com.invalid
    key: "{{ lookup('file', 'pubkeys/foo.com.invalid') }}"
```

### authorized_key
傳送主機金鑰至被控端主機，用於下次可以用金鑰登入
```yaml
---
- name: test authorized_key module
  hosts:
    - all
  gather_facts: false

  tasks:
    - name: set authorized key
      authorized_key:
	  	# 被控端用戶名
        user: mickey
        state: present
        key: "{{ lookup('file', '/home/mickey/.ssh/id_rsa.pub') }}"
...
```

## 🐧排程
### at
```yaml
---
- name: test at module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a at scheduled
      at:
        command: ls -d / > /dev/null
        count: 20
        # minutes, hours, days, weeks
        units: minutes
		# 如果有一個一樣的排程，是否還需要再建立(默認fasle)
        unique: yes
        # state: absent
...
```

### cron
開機執行的方法：將要執行的shell script寫入/etc/rc.d/rc.local
minute、hour、day、month、weekday，時間表示法和/etc/crontab一樣
```yaml
---
- name: test cron module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a cron scheduled
      cron:
	  	# cron-file注解
        name: "a new batch job"
		# 用誰的身份指定排程
		user: root
        # annually, monthly, weekly, daily, hourly, reboot
        special_time: hourly
        state: present
		# cron執行內容
        job: "ls / > /dev/null"
		# 若無指定cron_file，cron排程為user cron
		# 若有指定cron_file，cron排程為system cron(放在/etc/cron.d/**)
        # cron_file: ansible_cronfile
...
```

## 🐧服務相關
### service
```yaml
---
- name: test service module
  hosts:
    - mylocal
  gather_facts: false
  vars:
    software: httpd
  tasks:
    - name: install software
      yum:
        name: "{{ software }}"
        state: latest
    - name: start software service
      service:
        name: "{{ software }}"
		# reloaded, restarted, started, stopped
        state: started
...
```

### systemd
用於操作systemd
```yaml
---
- name: test systemd module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: reload httpd server
      systemd:
        name: httpd
        state: reloaded
        # systemctl daemon-reload
        daemon_reload: yes
...
```

### reboot
會等到被控端端重啟完後再執行後面的task
```yaml
---
- name: test reboot module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
	- name: do reboot
	  reboot:
	  	# 最多等垼時間
		reboot_timeout: 3600
...
```

## 🐧執行外部指令相關
### shell
用於執行外部命令
```yaml
---
- name: test shell module
  hosts:
    - mylocal
  gather_facts: false
  vars:
    myfile: "/home/mickey/Documents/ansible/ansible.cfg"
  tasks:
    - name: do someting in shell
	  # 用此方法讀取文檔比較沒有問題，如：cat "abc xyz"
      shell: cat {{ myfile|quote }}
      register: result
    - name: print result
      debug:
        var: result
...
```

### command
用於執行外部命令
- 可傳入參數
	```yaml
	---
	- name: test command module
	  hosts:
		- mylocal
	  gather_facts: false
	  vars:
		myfile: "/home/mickey/Documents/ansible/ansible.cfg"
	  tasks:
		- name: do command
		  command: /usr/bin/make_database.sh db_user db_name
		  become: yes
		  become_user: db_owner
		  # 傳入參數
		  args:
			chdir: somedir/
			creates: /path/to/database
	...
	```
- 抓取環境變數：`{{ ansible_env }}`
	抓指定環境變數：`{{ lookup('env','USER','HOME','SHELL') }}`
	```yaml
	---
	- name: test command module
	  hosts:
		- mylocal
	  vars:
		local_shell: "{{ ansible_env }}"
	  tasks:
		- name: print all the environment var in ansible
		  debug:
			var: local_shell
	...
	```

### script
把script丟到被控端執行
```yaml
---
- name: test script module
  hosts: all
  gather_facts: no
  tasks:
    - name: do script
      script: ./script/check_date
      register: result
    - name: print result
      debug:
        var: result
...
```

## 🐧文件相關
### file
- `state: touch`，建立文檔
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_file.yml
	---
	- name: test file module
	  hosts: localhost
	  gather_facts: no
	  vars:
		file_path: /home/mickey/ansible_touch/myfile
	  tasks:
		# 建立新文檔
		- name: touch a file
		  file:
			path: "{{ file_path }}"
			state: touch
			owner: student
			group: student
			mode: 0640
			# SELinux type
			setype: "samba_share_t"
	...
	[mickey@vm104 test_ansible]$ ansible-playbook playbook_file.yml
	[mickey@vm104 test_ansible]$ ls -Z /home/mickey/ansible_touch/myfile
	unconfined_u:object_r:samba_share_t:s0 /home/mickey/ansible_touch/myfile
	```
- `state: absent`，刪除文檔
	```yaml
	---
	- name: test file module
	  hosts: localhost
	  gather_facts: no
	  vars:
		file_path: /home/mickey/passwd

	  tasks:
		# 刪除文檔
		- name: delete a file
		  file:
			path: "{{ file_path }}"
			state: absent
	...
	```
- `state: link`，建立softlink
- 將文件上下文設置成默認(SELinux資料庫)
	```yaml
	---
	- name: test set selinux file context
	- 
	  hosts:
		- mylocal
	  tasks:
		- name: file selinux context set to default
		  file:
			path: /home/mickey/users.txt
			seuser: _default
			serole: _default
			setype: _default
			selevel: _defalut
	...
	```

### copy
從主控端覆製檔案至被控端
```yaml
---
- name: test copy module
  hosts: localhost
  tasks:
    - name: copy a file
      copy:
        src: /home/mickey/ansible/test_ansible/touch_file.txt
        dest: /home/mickey/copyfile
        # 若有目的地有相同名字的文件，是否要覆蓋，默認為yes
        force: no
...
```
- `copy`
	1. `ansible all -m copy -a 'content="Hello World" dest=/home/mickey/helloAnsible.txt'`，將字串寫至遠端機器
	2. `ansible all -m copy -a 'src=/etc/passwd dest=/home/mickey/'`，將主控端資料複製至被控端

### synchronize
檔案由控制端複製至被控端-->底層使用rsync
```yaml
---
- name: test module synchronize
  hosts:
    - all
  gather_facts: false
  
  tasks:
    - name: do synchronize
      synchronize:
        src: /home/mickey/ansible/test_ansible/ansible.cfg
        dest: /home/mickey/ansible_synchronize.cfg
...
```

### fetch
從被控端抓檔案至控制瑞
```yaml
---
- name: test fetch file
  hosts: 192.168.56.105
  gather_facts: no
  vars:
    user: mickey
  tasks:
    - name: pull file
      fetch:
        src: "/home/{{ user }}/fetchfile"
        dest: "/home/mickey/{{  user  }}fetchfile"
		# 複製回來的路徑和被控端中此檔案的路徑是否需要一樣，默認為yes
		# flat: no
...
```
`flat: yes`時，抓回來的檔案結構：
```bash
[mickey@vm104 test_ansible]$ tree /home/mickey/mickeyfetchfile
/home/mickey/mickeyfetchfile
└── 192.168.56.105
    └── home
        └── mickey
            └── fetchfile
```

### stat
用於查看指定檔案的狀態
```yaml
---
- name: test stat module
  hosts: localhost
  gather_facts: no
  tasks:
    - name: verify the checksum of a file
      stat:
        path: /home/mickey/systemcron.txt
        # 可以指定雜湊演算法來計算檔案的雜湊值
        checksum_algorithm: md5
      register: result
    - name: print result
      debug:
        msg: "{{ result }}"
		# var: result
...
```
`stat`産出：
```json
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "stat": {
            "atime": 1621333015.940003,
            "attr_flags": "",
            "attributes": [],
            "block_size": 4096,
            "blocks": 8,
            "charset": "us-ascii",
            "checksum": "b1a93cdc61a6e54e9dceaa6c7df5b3b9",
            "ctime": 1621332901.875405,
            "dev": 64768,
            "device_type": 0,
            "executable": false,
            "exists": true,
            "gid": 1001,
            "gr_name": "student",
            "inode": 1625598,
            "isblk": false,
            "ischr": false,
            "isdir": false,
            "isfifo": false,
            "isgid": false,
            "islnk": false,
            "isreg": true,
            "issock": false,
            "isuid": false,
            "mimetype": "text/plain",
            "mode": "0644",
            "mtime": 1621332901.875405,
            "nlink": 1,
            "path": "/home/mickey/systemcron.txt",
            "pw_name": "mickey",
            "readable": true,
            "rgrp": true,
            "roth": true,
            "rusr": true,
            "size": 1559,
            "uid": 1000,
            "version": "3862059110",
            "wgrp": false,
            "woth": false,
            "writeable": true,
            "wusr": true,
            "xgrp": false,
            "xoth": false,
            "xusr": false
        }
    }
}
```

### lineinfile
- 簡單用法，在指定檔案末端新增/刪除一行
	```yaml
	---
	- name: test lineinfile module
	  hosts: localhost
	  tasks:
		- name:
		  lineinfile:
			path: /home/mickey/lineinfile
			line: 'add a line to the file'
			state: present
	...
	```
- `regexp`，用正規表達式尋找，可用於修變設定檔內容
	```yaml
	---
	- name: test lineinfile module
	  hosts: localhost
	  gather_facts: no
	  tasks:
		- name:
		  lineinfile:
			path: /home/mickey/passwd
			regexp: 'student'
			line: 'lineinfile module add a line under "student"'
			state: present
	...
	```

### blockinfile
- 簡單用法，在指定檔案末端新增多行
	```yaml
	---
	- name: test blockinfile module
	  hosts: localhost
	  gather_facts: no
	  tasks:
		- name: add many line in file
		  blockinfile:
			path: /home/mickey/passwd
			state: present
			block: |
			  "-------------"
			  "add a block in file"
			  "-------------"
	...
	```
- 在符合正則表達式的那行前面/後面加上多行
	```yaml
	---
	- name: test blockinfile module
	  hosts: localhost
	  gather_facts: no
	  tasks:
		- name: add many line in file
		  blockinfile:
			path: /home/mickey/passwd
			state: present
			# 在符合條件的那行"前面"加上多行
			# 要在後面加的話，使用insertafter: '正則表達式'
			insertbefore: 'student'
			block: |
			  "-------------"
			  "add a block in file"
			  "-------------"
	...
	```

### sefcontext
```yaml
---
- name: test sefcontext module
  hosts: localhost

  tasks:
    - name: add SELinux type
      sefcontext:
        target: /home/mickey/ansible/test-ansible
        setype: samba_share_t
        # 新增；刪除用absent
        state: present
...
```

### template
[[🐧Linux_RH294_07_Jinja2 Templates]]-->主要用於布署文字檔

## 🐧管理儲存裝置
ansible_facts取得儲存裝置信息
```bash
[mickey@vm100 ansible]$ ansible mylocal -m setup -a 'filter=ansible_devices'
vm100.mickey.test.com | SUCCESS => {
    "ansible_facts": {
        "ansible_devices": {
            "sda": {
                "holders": [],
                "host": "SATA controller: Intel Corporation 82801HM/HEM (ICH8M/ICH8M-E) SATA Controller [AHCI mode] (rev 02)",
                "links": {
                    "ids": [
                        "ata-VBOX_HARDDISK_VB5a2c0d41-f32a3c25",
                        "scsi-0ATA_VBOX_HARDDISK_VB5a2c0d41-f32a3c25",
                        "scsi-1ATA_VBOX_HARDDISK_VB5a2c0d41-f32a3c25",
                        "scsi-SATA_VBOX_HARDDISK_VB5a2c0d41-f32a3c25"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VBOX HARDDISK",
                "partitions": {
                (省略)
            },
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```

### parted
```yaml
---
- name: test parted module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a new partition
      parted:
        device: /dev/sdb
        number: 1
        state: present
        part_end: 2000MiB
    - name: add an other partition
      parted:
        device: /dev/sdb
        number: 2
        state: present
        part_start: 2000MiB
        part_end: 4000MiB
...
```

### lvg
```yaml
---
- name: test lvg module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a vg
      lvg:
	  	# vg名稱
        vg: vg1
		# 由哪些pv組成
        pvs:
          - /dev/sdb1
          - /dev/sdb2
        pesize: 128k
...
```

### lvol
```yaml
---
- name: test lvol module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a lv
      lvol:
        vg: vg1
        lv: test_lv01
        state: present
        size: 2g
...
```

### filesystem
```yaml
---
- name: test filesystem module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: create filesystem in lv
      filesystem:
	  	# ext2,ext3,ext4,ext4dev,f2fs,lvm,xfs,vfat,swap 
        fstype: ext4
        dev: /dev/mapper/vg1-test_lv01
...
```

### mount
mount信息寫至/etc/fstab
```yaml
---
- name: test mount module
  hosts:
    - mylocal
  gather_facts: false
  vars:
    mount_point: "/lv01"
  tasks:
    - name: create mount file
      file:
        path: "{{ mount_point }}"
        state: directory
    - name: do mount
      mount:
        path: "{{ mount_point }}"
		# src: UUID=5f61754b-ec60-4493-9a90-93dd7080d2a5
        src: /dev/mapper/vg1-test_lv01
        fstype: ext4
		# absent --> 刪除/etc/fstab中的設置
		# mounted --> mount
		# unmounted --> umount
        state: present
    - name: mount all
      command: mount -a
...
```

### 綜合-移除lvm
```yaml
---
- name: return mount status
  hosts:
    - mylocal
  gather_facts: false
  vars:
    mount_point: "/lv01"
    lv_name: "test_lv01"
    vg_name: "vg1"
  tasks:
    - name: do umount
      mount:
        state: unmounted
        path: "{{ mount_point }}"
    - name: remove /etc/fstab
      mount:
        state: absent
        path: "{{ mount_point }}"
    - name: delete mount point
      file:
        state: absent
        path: "{{ mount_point }}"
    - name: remove lv
      lvol:
        vg: "{{ vg_name }}"
        lv: "{{ lv_name }}"
        state: absent
        force: yes
    - name: remove gv
      lvg:
        vg: "{{ vg_name }}"
        state: absent
    - name: remove partition
      parted:
        device: /dev/sdb
        state: absent
        number: "{{ item }}"
      loop:
        - 1
        - 2
...
```

### 綜合-mount swap
```yaml
---
- name: make a swap partition
  hosts:
    - mylocal
  gather_facts: false
  vars:
    device_name: "/dev/sdb"
    partition_num: 1
  tasks:
    - name: make partition
      parted:
        device: "{{ device_name }}"
        number: "{{ partition_num }}"
        state: present
        part_end: 2GiB
    - name: create file system
      filesystem:
        fstype: swap
        dev: "{{ device_name }}{{ partition_num }}"
        force: yes
    - name: turn on swap
      command: "swapon {{ device_name }}{{ partition_num }}"
    - name: write mount info in /etc/fstab
      mount:
        src: "{{ device_name }}{{ partition_num }}"
        path: swap
        fstype: swap
        opts: pri=20
        state: present
...
```

## 🐧網路管理
ansible_fact取得網卡信息
```bash
[mickey@vm100 ansible]$ ansible mylocal -m setup \
> -a 'gather_subset=network filter=ansible_interfaces'
vm100.mickey.test.com | SUCCESS => {
    "ansible_facts": {
        "ansible_interfaces": [
            "lo",
            "enp0s3",
            "virbr0-nic",
            "virbr0",
            "enp0s8"
        ],
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
```

### nmcli
```yaml
---
- name: test nmcli module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: add a connection
      nmcli:
        conn_name: test_connection
        ifname: enp0s8
        type: ethernet
        ip4: "192.168.56.233/24"
        gw4: "192.168.56.255"
        state: present
      register: result
    - name: print result
      debug:
        var: result
...
```

### hostname
```yaml
---
- name: test hostname module
  hosts:
    - mylocal
  gather_facts: false
  tasks:
    - name: change hostname
      hostname:
        name: vm100.mickey.test.com
...
```

### firewalld
```yaml
- name: firewalld permits access to httpd service
  firewalld:
	service: http
	# firewall-cmd --permanent
	permanent: true
	state: enabled
	# firewall-cmd --reload
	immediate: yes
```

## 🐧流程控制
### fail
```yaml
# Example playbook using fail and when together
- fail:
    msg: The system may not be provisioned according to the CMDB status.
  when: cmdb_status != "to-be-staged"
```

### assert
斷言，如果條件成立就中斷
```yaml
- name: do assert
  assert:
    that:
      - my_param <= 100
      - my_param >= 0
    fail_msg: "'my_param' must be between 0 and 100"
    success_msg: "'my_param' is between 0 and 100"
```

## 🐧其他類型
### timezone
```yaml
- name: Set timezone to Asia/Tokyo
  timezone:
    name: Asia/Tokyo
```

### get_url
只能從server端下資料，不能上傳
```yaml
- name: download page
get_url: 
  url: "http://materials.example.com/labs/playbook-review/index.php"
  # 下載目的地
  dest: /var/www/html/index.php
  # 檔案權限
  mode: 0644
```

### uri
http、https的url上傳、下載
- 取得網頁返回內容
	```yaml
	---
	- name: check web page is alive
	  hosts: all
	  gather_facts: no
	  tasks:
		- name: get url response
		  uri:
			url: http://google.com
			return_content: yes
		  register: result
		- name: print result
		  debug:
			var: result
		- name:
	...
	```
- 取得網頁返回狀態
	```yaml
	- name: connect web server
	uri: 
	  url: http://servera.lab.example.com
	  return_content: yes
	  status_code: 200
	```

### debug
- 打印字串
	```yaml
	- name: display free memory
	  debug:
		msg: "Free memory for this system is {{ ansible_facts['memfree_mb'] }}"
	```
- 打印變量
	```yaml
	- name: print result
	  debug:
		var: result
	```
- log産出設置
	```yaml
	- name: display the "result" variable
	  debug:
		var: result
		# 值為0~4，對應於-v、-vv、-vvv、-vvvv
		verbosity: 1
	```
