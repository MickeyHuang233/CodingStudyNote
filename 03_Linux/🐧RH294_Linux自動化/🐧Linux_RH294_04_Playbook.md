# Ansible Playbook
## 🐧vi前期工作
為了方便編寫時的格式檢查，需要先在做以下步驟：
1. `yum install vim-enhanced`，安裝套件
2. `vi ~/.vimrc`建立檔案並輸入：
	```bash
	# 編輯.yaml或.yml文件時，按tab鍵代表按兩個空白鍵
	autocmd FileType yaml setlocal ai ts=2 sw=2 et
	# 顯示垂直線(必用)
	set cursorcolumn
	# 顯示水平線(可不用)
	#set cursorline
	```

## 🐧playbook介紹
- [[🍀YAML]]格式，副檔名為.yml或.yaml都可以
- 以`---`開頭，以`...`結尾
- 以**兩個空白**分階層
- `#注解`
- 簡單的Playbook范例
	```yaml
	---
	# 此playbook執行內容描述
	- name: my first playbook
	  # 主機列表
	  hosts:
		- 192.168.56.102
		- 192.168.56.103
	  # 任務列表
	  tasks:
		# 任務描述
		- name: create new user "newbie" with UID 4000
		  # 使用模組
		  user:
			name: newbie
			uid: 4000
			state: present #代表建立
	...
	```
- 一個playbook可以有多個play(如下圖左邊)，但這樣會不好管理，所以不建議這麼做
	比較好的做法(如下圖右邊)是分不同模塊分playbook，再由一個playbook整合
	![[Linux_RH294_04_Playbook_01_import.png]]

## 🐧playbook執行順序
![[Linux_RH294_04_Playbook_03_playbook執行順序.png]]
1. pre_tasks
2. pre_tasks呼叫的handler
	- 呼叫幾次handler，就會執行幾次
3. role_tasks
4. playbook_tasks
5. role_handler
6. playbook_handler
	- playbook_tasks呼叫多次handler，只會執行一次
	- handler的執行順序為handler宣告順序，和呼叫順序無關
	- 同名handler只執行第一個宣告的handler
	- 若與pre_tasks呼叫相同handler，還是會再執行一次此handler
7. post_tasks
8. post_tasks呼叫的handler
	- 呼叫幾次handler，就會執行幾次
	- 若與pre_tasks、playbook_tasks呼叫相同handler，還是會再執行一次此handler

## 🐧模組參考文件
- 可參考官方文件：[Ansible Documentation](https://docs.ansible.com/)
- 常用模組可參考：[[🐧Linux_RH294_09_常用Module介紹]]
- `ansible-doc -l`，查看所有模組
- `ansible-doc <模組名>`，查看指定模組說明
- `ansible-doc -s <模組名>`，只看指定模組參數說明

### 模組說明查看
```bash
[mickey@vb101 ~]$ ansible-doc yum
(省略)
          status:
          - stableinterface
          supported_by: core
(省略)
```
- status
	1. stableinterface，表示此模組參數名不會再改變，可安心使用此模組
	2. preview，表示此模組的參數名可能會改變(Beta版)，若之後參數名改變後則需要修改playbook才可正常使用
	3. deprecated，表示此模組即將被棄用
	4. removed，表示此模組已經被棄用，後面一般會寫可替代模組
- supported_by，此模組的開發者
	1. core，表示此模組由ansible小組開發
	2. curated，表示此模此由第三方廠商開發
	3. community，表示此模組由社區開發，再斟酌使用

### 引入本機模組
可在ansible.cfg設定檔中指定本機模組路徑：
```ini
library = /user/share/my_mymodules
```

## 🐧執行Playbook
- `ansible-playbook --syntax-check <YAML路徑>`，檢查YAML格式
	```bash
	[mickey@vb101 test]$ ansible-playbook --syntax-check playbook.yml

	playbook: playbook.yml
	```
- `ansible-playbook -C <YAML路徑>`，摸擬執行playbook，用於檢查指令參數
	```bash
	[mickey@vb101 test]$ sudo ansible-playbook -C playbook.yml
	[sudo] password for mickey:

	PLAY [my first playbook] **************************************************************************

	TASK [Gathering Facts] **************************************************************************
	ok: [192.168.56.102]
	ok: [192.168.56.103]

	TASK [create new user "newbie" with UID 4000] **************************************************************************
	ok: [192.168.56.103]
	ok: [192.168.56.102]

	PLAY RECAP **************************************************************************
	192.168.56.102             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
	192.168.56.103             : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
	```
- `ansible-playbook -i inventory2 ./playbook.yml`，`-i`，指定設備檔
- `ansible-playbook <YAML路徑>`，執行playbook
	1. 所有playbook真正執行前都會先執行TASK [Gathering Facts]，可參考：[[🐧Linux_RH294_05_Playbook變量]]-ansible facts
	2. 查看執行過程的信息：越簡單`-v`-->`-vv`-->`-vvv`-->`-vvvv`越詳細
	3. 執行結果
		- ok，狀態已符合，故不執行
		- failed，執行失敗
		- changed，執行成功
		- skipped，條件不成立而跳過不執行
	```bash
	[mickey@vb101 test]$ sudo ansible-playbook playbook.yml
	PLAY [my first playbook] ************************************************************

	TASK [Gathering Facts] **************************************************************
	ok: [192.168.56.102]
	ok: [192.168.56.103]

	TASK [create new user "newbie" with UID 4000] ***************************************
	fatal: [192.168.56.103]: FAILED! => {"changed": false, "msg": "useradd: UID 4000 is not unique\n", "name": "newbie", "rc": 4}
	changed: [192.168.56.102]

	PLAY RECAP **************************************************************************
	192.168.56.102             : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
	192.168.56.103             : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
	```

## 🐧YAML格式
### String
- 一般用法
	```yaml
	# 以下三種都可以
	this is string
	'this is string'
	"this is string"
	```
- `|`表示字串多行顯示(斷行位置和變量一樣)
	```yaml
	newlink : |
	  this is string 
	  in yaml
	```
- `>`表示字串轉為一行顯示
	```ymal
	aml
	newlink : >
	  this is string 
	  in yaml
	```

### Map
- 分行顯示
	```yaml
	name: test map
	svcserveice: httpd
	svcport: 80
	```
- 單行顯示(不建議)
	```yaml
	{name: test map, svcserveice: httpd, svcport: 80}
	```

### List
- 分行顯示
	```yaml
	host:
	  - servera
	  - serverb
	  - serverc
	```
- 單行顯示(不建議)
	```yaml
	host: [servera, servera, serverc]
	```

## 🐧hosts
`hosts`列出這個腳本(play)所應用的主機名/主機IP/群組
- 基本用法
	```yaml
	hosts: 
	  # 主機名、主機IP、群組可混用
	  - server
	  - db
	  - myvm
	  - 196.168.56.105
	```
- 萬用字元，`*`，用`''`包起來
	```yaml
	# 表示all
	hosts: '*'
	```

	```yaml
	hosts: 
	  # 可能用於主機名、主機IP或群組名，只要在inventory設備檔中有此字串就符合
	  - '192.168.2.*'
	  - 'datacenter*'
	```
- 排除，`!`
	```yaml
	# 除test1.example.com外的dev群組
	# 以下兩種方法都可以使用
	hosts: 
	  - !test1.example.com,dev
	  - dev,!test.example.com
	```
- 兩群組取交集
	```yaml
	# 以下兩種方法都可以使用
	host: 
	  - lab,&dev
	  - &dev,lab
	```


## 🐧流程控制
### 基本流程
#### 迴圈 loop
- `loop`，目前的迴圈用法
	```yaml
	- name: test for loop
	  hosts:
		- localhost
	  vars:
		service_list:
		  - httpd
		  - postfix

	  tasks:
		- name: install service
		  yum:
			name: "{{ service_list }}"
			state: latest
		- name: do for loop
		  service:
			# 迴圈值
			name: "{{ item }}"
			state: started
		  # 看list中有幾個元素就迴圈幾次
		  loop: "{{ service_list }}"
	```
	執行
	```bash
	[mickey@vm104 test_ansible]$ ansible-playbook playbook_loop.yml
	(省略)
	TASK [do for loop] **********************************************************
	ok: [localhost] => (item=httpd)
	changed: [localhost] => (item=postfix)
	```
- `with_items`，早期的用法(了解就好)
	相關用法可參考：[Migrating from with_X to loop](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#migrating-from-with-x-to-loop)
	```yaml
	#(省略)
	- name: do for loop
	  service:
		# 迴圈值
		name: "{{ item }}"
		state: started
	  # 看list中有幾個元素就迴圈幾次
	  with_items: "{{ service_list }}"
	```

#### 判斷 when
- `when`，可判斷string、number、boolean，判斷為true就執行此模組
	```yaml
	- name: test when control
	  hosts: 192.168.56.105
	  vars:
		myuser: "{{ ansible_facts['ansible_local']['custom']['users']['user1'] }}"
	  tasks:
		- name: do when
		  debug:
			msg: "my user is {{ myuser }}"
		  # 此時變量引用不需要加"{{ }}"
		  when : myuser == "jack"
	```
- 判斷類型
	1. `myuser == "jack"`，字串相等
	2. `max_memory == 512`，數字相等
	3. `max_memory < 512`
	4. `max_memory <= 512`
	5. `max_memory > 512`
	6. `max_memory >= 512`
	7. `max_memory != 512`
	8. `max_memory is defined`，變數有宣告
	9. `max_memory is not defined`，變數無宣告
	10. 變量值為`true`、`True`、`1`、`yes`都為true
	11. `testuser in user_list`，判斷值是否在List中，注意區分大小寫
- 邏輯運算
	1. `<判斷 or 判斷>`，或
	2. `<判斷 and 判斷>`，且
	3. `when`可指定List，且
	4. `not memory_available`，非
		```console
		not memory_available
		```

#### 判斷 + 迴圈
先判斷為true後才執行迴圈
```yaml
- name: test when control
  hosts: 192.168.56.105
  vars:
    check_name: jack
    myusers:
      - mickey
      - jack
      - mark
      - max
  tasks:
    - name: do when
      debug:
        msg: "my user is {{ item }}"
      loop: "{{ myusers }}"
      when : check_name in myusers
```

### 忽略錯誤 ignore_errors
![[Linux_RH294_04_Playbook_02_流程控制.png]]
- playbook是以被控端為單位執行，也就是說被控端_01執行失敗不會影響被控端_02的執行
- task在默認情況下如果執行失敗，就會停止整個playbook運行；如果想要忽略此task的錯誤，則在模組裡加上`ignore_errors: yes`
- 執行回傳參數中`rc=0`代表執行成功，若`rc!=0`代表執行失敗
```yaml
---
- name: reset vg0
  hosts:
    - localhost
  vars:
    - vg_size:
  tasks:
    - name: create partition
      parted:
        device: /dev/sdb
        number: 1
        state: present
        part_end: "{{ vg_size }}GiB"
      # 設置忽略錯誤
      ignore_errors: true
    - name: create vg
      lvg:
        vg: vg0
        state: present
        pesize: 32
        pvs:
          - /dev/sdb1
      when: vg_size is not defined
...

```

### Handler
- handler的概念相當於函數，當task執行結果為change時呼叫指定handler
- handler執行順序
	1. handler沒被呼叫則不會被執行
	2. 執行順序與handler的宣告順序一致(由上而下)，和被呼叫的順序無關
	3. 執行task所有內容後才開始執行handlers的內容
	4. 如果handler中有名稱重覆，只會執行第一個被宣告的handler
	5. 若同一個handler在tasks中被呼叫多次，呼叫的handler只會執行一次；而pre_tasks和post_tasks中呼叫handler不在此限制

#### 基本用法
```yaml
- name: test handler
  hosts: localhost
  tasks:
    - name: task change
      lineinfile:
        path : /home/mickey/test.txt
        line: 'add something in file'
        state: present
      # 執行結果為change(有執行)呼叫handler
      notify:
        - print sucess

  handlers:
      # 事件名稱
    - name: print sucess
      # 事件執行套件
      debug:
        msg: "add something in file sucess"
...
```

#### 強行執行設置 force_handlers
`force_handlers: yes`，設置就算task執行失敗，在失敗中斷前執行已經被呼叫handler
- playbook_handler.yml
	```yaml
	---
	- name: test handler
	  hosts: localhost
	  gather_facts: no
	  # 設置就算task執行失敗，在失敗中斷前執行已經被呼叫handler
	  force_handlers: yes

	  tasks:
		- name: create a file
		  file:
			path: /home/mickey/ansible/test_ansible/touch_file.txt
			state: touch
			mode: u=rw,g=r,o=r
		  notify:
			- print touch file sucess

		- name: task change
		  lineinfile:
			path : /home/mickey/other_file.txt
			line: "add something in file"
			state: present
		  notify:
			- print message

	  handlers:
		- name: print touch file sucess
		  debug:
			msg: "touch file sucess"

		- name: print message
		  debug:
			msg: "add something in file fail"
	...
	```
- 執行結果：
	```bash
	[mickey@vm104 test_ansible]$ ansible-playbook playbook_handler.yml

	PLAY [test handler] *************************************************************************

	TASK [create a file] ************************************************************************
	changed: [localhost]

	TASK [task change] **************************************************************************
	fatal: [localhost]: FAILED! => {"changed": false, "msg": "Destination /home/mickey/other_file.txt does not exist !", "rc": 257}

	RUNNING HANDLER [print touch file sucess] ***************************************************
	ok: [localhost] => {
		"msg": "touch file sucess"
	}

	PLAY RECAP **********************************************************************************
	localhost                  : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
	```

### 自行判斷狀態
#### fail
- `failed_when`，符合條件，執行結果才為fail
	```yaml
	---
	- name: run shell in playbook
	  hosts: localhost
	  gather_facts: no

	  tasks:
		# 執行外部命令，結果永遠為change(已執行)
		- name: do something shell
		  shell: "date +%F"
		  register: mydate
		  failed_when: "'2021' in mydate['stdout']"
	...
	```
- `fail`模組
	```yaml
	---
	- name: run shell in playbook
	  hosts: localhost
	  gather_facts: no

	  tasks:
		- name: do something shell
		  shell: "date +%F"
		  register: mydate

		- name: fail condition
		  fail:
			msg: "the shell is fail"
		  when: "'2021' in date['stdout']"
	...
	```

#### change
`changed_when`，符合條件，執行結果才為change
```yaml
---
- name: run shell in playbook
  hosts: localhost
  gather_facts: no

  tasks:
    - name: do something shell
      shell:
        cmd: "date +%F"
      register: result
      changed_when: "'2020' in result['stdout']"
      notify:
       - print message

  handlers:
    - name: print message
      debug:
        msg: "result status is changed"
...
```

### try catch finally
- 若`block`中的模組執行失敗，則執行`rescue`
- 無論`block`執行是否失敗，都會執行`always`
- 相當於`try...catch...finally`
```yaml
---
- name: test block...rescue...always
  hosts: localhost
  gather_facts: no

  tasks:
    - name: do something
      block:
        - name: do block
          shell:
            cmd: "date +%F"
          register: result

        - name: fail condition
          fail:
            msg: "block have fail"
          when: "'2021' in result['stdout']"

      rescue:
        - name: do rescue
          debug:
            msg: "something wrong in block"

      always:
        - name: do always
          debug:
            msg: "always do print"
...
```

## 🐧管理大型playbook
### 分批處理主機
```yaml
- name: test serial
  host: '*'
  # 2台2台分批處理，也可使用百分比，如：一次處理20%的主機
  serial: 2
  tasks:
    (省略)
```

### import_playbook
`import_playbook: <yaml路徑>`，一開始會先把playbook所有內容引入
1. 被引入的yaml檔，注意：需要注意縮排格式
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_templates_if.yml
	---
	- name: test template if
	  hosts: myvm
	  vars:
		users:
		  - mickey
		  - jack
		  - root
		  - mike
	  tasks:
		- name: template reader
		  template:
			src: ./templates/if.j2
			dest: /home/mickey/test_templates_if.txt
			owner: mickey
			group: student
			mode: 0644
	...
	```
2. 引入playbook
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_import_playbook.yml
	---
	- name: do someting else
	  hosts: localhost
	  tasks:
		- name: print something
		  debug:
			msg: "this is only print playbook"

	- name: test import playbook
	  import_playbook: ./playbook_templates_if.yml
	- name: prepare the web server
	  import_playbook: ./web.yml
	- name: prepare the db server
	  import_playbook: ./db.yml
	...
	```

### import_tasks
`import_tasks: <yaml路徑>`，playbook一執行時就將所有的task加載至記憶體，主要抽取重用的部分
1. 將要重覆使用的部分獨立成一個YAML檔，注意：需要注意縮排格式
	```bash
	[mickey@vm104 test_ansible]$ cat enviroment.yml
	---
	  - name: Install the {{ package }} package
		yum:
		  name: "{{ package }}"
		  state: latest
	  - name: Start the {{ service }} service
		service:
		  name: "{{ service }}"
		  enabled: true
		  state: started
	...
	```
2. 引入task
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_import_tasks.yml
	---
	- name: test import tasks
	  hosts: localhost
	  vars:
		package: httpd
		service: httpd
		is_do: true
		import_tasks_filename: ./enviroment.yml
	  tasks:
		- name: install web server
		  # 可以使用變數引入
		  import_tasks: "{{ import_tasks_filename }}"
		  # 可以使用when判斷，但無法使用loop執行
		  when: "{{ is_do }}"
	...
	```

### include_tasks
`include_tasks: <yaml路徑>`，執行到此task時才會加載到記憶體，如果對記憶體要求不大，基本上用import_tasks即可
1. 將要重覆使用的部分獨立成一個YAML檔，注意：需要注意縮排格式
2. 引入task
	```bash
	[mickey@vm104 test_ansible]$ cat playbook_include_tasks.yml
	---
	- name: test import tasks
	  hosts: localhost
	  vars:
		package: httpd
		service: httpd
		is_do: true
		import_tasks_filename: ./enviroment.yml
	  tasks:
		- name: install web server
		  # 可以使用變數引入
		  include_tasks: "{{ import_tasks_filename }}"
		  # 可以使用when判斷
		  when: "{{ is_do }}"
	...
	```
3. 其他注意事項：
	- `ansible-playbook <playbook路徑> --list-tasks`，可以列茨所有task name，但不會包含include中的task
		```bash
		[mickey@vm104 test_ansible]$ ansible-playbook playbook_include_tasks.yml --list-tasks

		playbook: playbook_include_tasks.yml

		  play #1 (localhost): test import tasks        TAGS: []
			tasks:
			  install web server        TAGS: []
		```
	- `ansible-playbook <playbook路徑> --start-at-task`，可以指定從第幾個task開始執行
	- `include_tasks`不可呼叫handler

### pre_tasks、post_tasks
- `pre_tasks`，執行tasks之前會先執行
- `post_tasks`，執行完所有tasks後才會執行
```yaml
---
- name: test pre_tasks and post_tasks
  hosts: 192.168.56.104
  gather_facts: no
  vars:
    is_change: true

  pre_tasks:
    - name: print pre_tasks message
      debug:
        msg: "do pre_tasks"
      changed_when: "{{ is_change }}"
      notify: print handler message

  tasks:
    - name: print tasks message
      debug:
        msg: "do tasks"
      changed_when: "{{ is_change }}"
      notify: print handler message

  post_tasks:
    - name: print post_tasks message
      debug:
        msg: "do post_tasks"
      changed_when: "{{ is_change }}"
      notify: print handler message

  handlers:
    - name: print handler message
      debug:
        msg: "do handler"
...
```


