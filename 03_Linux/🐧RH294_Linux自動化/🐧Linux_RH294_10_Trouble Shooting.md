# Trouble Shooting
## 🐧log産出
默認情況下，ansible執行過程不會有log，若需要log，可設置：
注意：資料夾路徑必須存在，且需要有寫的權限
- [[🐧Linux_RH294_03_ansible設定檔]]
	```ini
	log_path = ./log/ansible.log
	```
- 宣告環境變數
```
export ANSIBLE_LOG_PATH="/home/mickey/ansible/.log/ansible.log"
```

## 🐧debug模組

## 🐧檢查playbook語法
```bash
ansible-playbook <playbook路徑> --syntax-check
```

## 🐧單步執行
`--step`
- (N)o，不執行這一步
- (y)es，執行這一步
- (c)ontinue，直接執行後面的步驟
```bash
[mickey@vm104 test_ansible]$ ansible-playbook playbook_touble_1.yml --step

PLAY [test toubleshooting 1] ***************************************************************************
Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue: y

Perform task: TASK: Gathering Facts (N)o/(y)es/(c)ontinue: *********************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.56.105]
ok: [192.168.56.104]
Perform task: TASK: step 1 -- display free memory (N)o/(y)es/(c)ontinue:
```

## 🐧從指定task執行
`--start-at-task="<taskName>"`
```bash
[mickey@vm104 test_ansible]$ ansible-playbook playbook_touble_1.yml --start-at-task="step 2 -- add a user"
PLAY [test toubleshooting 1] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.56.104]
ok: [192.168.56.105]

TASK [step 2 -- add a user] *********************************************************************************
ok: [192.168.56.105]
ok: [192.168.56.104]

TASK [step 3 -- print result] *********************************************************************************
skipping: [192.168.56.104]
skipping: [192.168.56.105]

PLAY RECAP *********************************************************************************
192.168.56.104             : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  
192.168.56.105             : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  
```

## 🐧摸擬執行
### 基本用法
```bash
ansible-playbook <playbook路徑> -C
```
```bash
ansible-playbook <playbook路徑> --check
```

### task禁止模擬執行
```yaml

---
- name: test toubleshooting 1
  hosts: all
  tasks:
    - name: step 1 -- display free memory
      debug:
        msg: "Free memory for this system is {{ ansible_facts['memfree_mb'] }}"
    - name: step 2 -- delete user
      user:
        state: absent
        name: student
        group: mickey
      # 設置不允許模擬執行，默認為yes
      check_mode: no
    - name: install httpd
      yum:
        name: httpd
        state: latest
...
```

### 顯示修改前後的變化
```
ansible-playbook --check --diff playbook_touble_1.yml
```

## 🐧檢查server
### 檢查網站是否活著
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
	# 判斷網頁是否包含指定字串
    - name: is web page alive
      fail:
        msg: "web page is not alive"
      when: "'google' not in result['content']"
...
```

### 檢查服務是否啟動
主要思路：服務被啟動時會在/var/run中産生*.pid的檔案，因此只要查看檔案是否存在就可以了
```yaml
---
- name: check server is active
  hosts: all
  gather_facts: no
  tasks:
    - name: get file status
      stat:
        path: /var/run/httpd/httpd.pid
      register: result
    - name: print result
      debug:
        var: result
    - name: fail if the httpd is running
      # 其中一個條件，判斷為false，就中斷
      assert:
        that:
          - result.stat.exists
    - name: print sucess message
      debug:
        msg: "check server is active playbook is over."
...
```

