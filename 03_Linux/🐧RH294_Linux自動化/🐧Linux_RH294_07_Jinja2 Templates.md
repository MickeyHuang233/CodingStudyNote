# Jinja2 Templates
template模組，主要用於布署文字檔
- template檔副檔名都為`.j2`

## 🐧語法介紹
1. 變數，`{{ EXPR }}`
2. 流程控制，`{% EXPR %}`
	- 迴圈(`users`為list變數)
	```j2
	{% for user in users %}
		{{ user }}
	{% endfor %}
	```
	- 判斷(`is_finish`為boolean變數)
	```j2
	{% if is_finish %}
	{{ result }}
	{% endif %}
	```
3. 注解，`{# COMMENT #}`
4. 可使用[[🐧Linux_RH294_06_Ansible Filter]]

## 🐧templates范例
### 迴圈
- loop.j2
	```j2
	{# 習慣加上，用於提醒此文檔用ansible編寫 #}
	# {{ date_now['stdout'] }}
	# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

	{# loop #}
	ansible_all_ipv4_addresses:
	{% for address in ansible_facts['all_ipv4_addresses'] %}
	  {# loop.index表示迴圈的第幾圈 #}
	  {{loop.index}}        {{ address }}
	{% endfor %}
	```
- playbook_templates.yml
	```yaml
	---
	- name: test templates
	  hosts: all
	  tasks:
		- name: get date
		  shell:
			cmd: "date +%F"
		  register: date_now
		- name: template reader
		  template:
			src: ./templates/loop.j2
			dest: /home/mickey/test_templates.txt
			owner: mickey
			group: student
			mode: 0644
	...
	```
- 産出
	```bash
	[mickey@vm104 test_ansible]$ ll ~
	total 52
	drwxr-x---. 3 mickey student    26 May 11 00:00 ansible
	-rw-r--r--. 1 mickey student   157 May 18 20:10 test_templates.txt
	[mickey@vm104 test_ansible]$ cat ~/test_templates.txt
	# 2021-05-18
	# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

	ansible_all_ipv4_addresses:
		1   10.0.2.15
		2   192.168.56.104
		3   192.168.122.1
	```

### 判斷
- if.j2
	```j2
	# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

	my users:
	{% for user in users %}
	  {% if not "root"==user %}
		{{ user }}
	  {% endif %}
	{% endfor %}
	```
- playbook_templates_if.yml
	```yaml
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
- 産出
	```bash
	[mickey@vm104 test_ansible]$ cat /home/mickey/test_templates_if.txt
	# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

	my users:
		  mickey
			jack
			  mike
	```

### 産出hosts
- host.j2
	```j2
	{# 使用魔法變數迴圈 #}
	{% for myhost in groups['all'] %}
	{{ hostvars[myhost]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[myhost]['ansible_facts']['fqdn'] }} {{ hostvars[myhost]['ansible_facts']['hostname'] }}
	{% endfor %}
	```
- playbook_template_hosts.yml
	```yaml
	---
	- name: test template host
	  hosts: all
	  tasks:
		- name: template reader
		  template:
			src: ./templates/host.j2
			dest: /home/mickey/test_templates_hosts.txt
			owner: mickey
			group: student
			mode: 0644
	...
	```
- 産出
	```bash
	[mickey@vm104 test_ansible]$ cat /home/mickey/test_templates_hosts.txt
	10.0.2.15 vm105 vm105
	10.0.2.15 vm104 vm104
	```
