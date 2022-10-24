# Ansible Filter
資料轉換器，可以在Google查詢"ansible filter"

## 將資料轉為JSON或YAML
1. `{{ output | to_json }}`
2. `{{ output | to_yaml }}`
3. `{{ output | to_nice_json }}`
4. `{{ output | to_nice_yaml }}`
```yaml
---
- name: test ansible filter
  hosts: all
  tasks:
	- name: print to json
	  # 不易於查看的格式(沒分行)
	  debug:
		msg: "{{ ansible_facts | to_json }}"
	- name: print to yaml
	  # 不易於查看的格式(沒分行)
	  debug:
		msg: "{{ ansible_facts | to_yaml }}"

	- name: print to nice json
	  # 易於查看的格式(有分行)
	  debug:
		msg: "{{ ansible_facts | to_nice_json }}"
	- name: print to nice yaml
	  # 易於查看的格式(有分行)
	  debug:
		msg: "{{ ansible_facts | to_nice_yaml }}"
...
```

## 解析JSON或YAML
1. `{{ output | from_json }}`
2. `{{ output | from_yaml }}`

## 産生密碼雜湊
`{{ output | password_hash('sha512') }}`
```yaml
---
- name: test ansible filter
  hosts: all
  tasks:
	- name: print password hash
	  debug:
		msg: "{{ 'redhat' | password_hash('sha512') }}"
	- name: create a user
	  user:
		state: present
		name: newstudent
		password: "{{ 'redhat' | password_hash('sha512') }}"
		group: student
...
```



