# Linux_RH124_13_SSH連線
## 🐧簡介
SSH加密和[[🔐密碼學]]有關，SSH的加密方式主要分為兩個版本：V1和V2
- SSH Server
	1. SSH V1(Red Hat 7開始不支援V1的協定)：RSA
	2. SSH V2：RSA、DSA、ECDSA
- SSH Client
	- Host key：Disk
		1. SSH V1：RSA
		2. SSH V2：RSA、DSA、ECDSA
	- Service key：寫在RAM中，key有1024bit，預設1小時會更新一次

### SSH 加密傳輸流程
![Linux_RH124_14_SSH連線_01_SSH加密流程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_14_SSH%E9%80%A3%E7%B7%9A_01_SSH%E5%8A%A0%E5%AF%86%E6%B5%81%E7%A8%8B.png?raw=true)
- SSH V1做法：
	![Linux_RH124_14_SSH連線_02_SSH V1加密流程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_14_SSH%E9%80%A3%E7%B7%9A_02_SSH%20V1%E5%8A%A0%E5%AF%86%E6%B5%81%E7%A8%8B.png?raw=true)
	1. SSH Server回傳public key和private key給Client
	2. Server會將session-key用public key進行非對稱加密，並傳給Client
	3. 後面的通信都是用session-key進行對稱加密
- SSH V2做法：
	![Linux_RH124_14_SSH連線_03_SSH V2加密流程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_14_SSH%E9%80%A3%E7%B7%9A_03_SSH%20V2%E5%8A%A0%E5%AF%86%E6%B5%81%E7%A8%8B.png?raw=true)
	1. 使用`df`指令在Client和Server兩端生成session-key
	2. 但是SSH Server還是會丟public key和private key給SSH Client

### 圖形化界面遠端
- [x2go](https://wiki.x2go.org/doku.php)
- [xrdp](http://xrdp.org/)

## 🐧ssh遠端流程
### server産生金鑰
SSH Server啟動時，會檢查/etc/ssh下有沒有金鑰，如果沒有的話會自動産生
public key命名原則：ssh_host_<加密方式>.pub
private key命名原則：ssh_host_<加密方式>
```bash
[root@localhost ssh]# ll /etc/ssh
total 600
-rw-r--r--. 1 root root     577388 Jul 25  2019 moduli
-rw-r--r--. 1 root root       1716 Jul 25  2019 ssh_config
drwxr-xr-x. 2 root root         28 Mar  7 20:02 ssh_config.d
-rw-------. 1 root root       4425 Jul 25  2019 sshd_config
-rw-r-----. 1 root ssh_keys    480 Mar  7 20:12 ssh_host_ecdsa_key
-rw-r--r--. 1 root root        162 Mar  7 20:12 ssh_host_ecdsa_key.pub
-rw-r-----. 1 root ssh_keys    387 Mar  7 20:12 ssh_host_ed25519_key
-rw-r--r--. 1 root root         82 Mar  7 20:12 ssh_host_ed25519_key.pub
-rw-r-----. 1 root ssh_keys   2578 Mar  7 20:12 ssh_host_rsa_key
-rw-r--r--. 1 root root        554 Mar  7 20:12 ssh_host_rsa_key.pub
```

為了確保遠端連線至自己的目標機器，需要用`ssh-keygen -l`對這些公鑰進行雜湊，並自行保存雜湊值，雜湊值介紹可參考：[[🔐密碼學]]-消息摘要算法
`-t`，可指定演算方式，默認為SHA256
```bash
[root@localhost ssh]# ssh-keygen -l
Enter file in which the key is (/root/.ssh/id_rsa): ssh_host_rsa_key.pub
3072 SHA256:S0ppqs1CuYuuseJGUKaxsne1bqo48r1WGcHKrS2tla0 no comment (RSA)
[root@localhost ssh]# ssh-keygen -l
Enter file in which the key is (/root/.ssh/id_rsa): ssh_host_ed25519_key.pub
256 SHA256:659Kev0WmEs1fhtRhbhuKuVi3Wrfmd5EwN7z7/thUGQ no comment (ED25519)
[root@localhost ssh]# ssh-keygen -l
Enter file in which the key is (/root/.ssh/id_rsa): ssh_host_ecdsa_key.pub
256 SHA256:QsLxggmRYrtugyyvFE7b0m1fnXI0REVj/WMJPZxQrs8 no comment (ECDSA)
```

### ssh連線
連線：`ssh [-i <金鑰路徑>] <遠端用戶名>@<遠端電腦名|IP>`
登出：`exit`或Ctrl + D
注意：在第一次登入時，會詢問是否要對此機器進行遠端連線，並會顯示此機器公開金鑰的雜湊方式和雜湊值(SHA256:QsLx...JPZxQrs8)，需要核對和之前記錄的目標機器的雜湊值是否一致，一致代表現在連線的機器就是你要的機器
```bash
[root@localhost .ssh]# ssh mickey@localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:QsLxggmRYrtugyyvFE7b0m1fnXI0REVj/WMJPZxQrs8.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

在確認完成後Linux會將已確認機器公鑰存至檔案，下次連線就不會再詢問：
- globe setting：/etc/ssh/ssh_known_hosts
- local setting：~/.ssh/known_hosts
```bash
[root@localhost .ssh]# cat /root/.ssh/known_hosts
localhost ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCj9qFgGJ5DCEYAtGTh2Cnslgq33Hdx9n+7S8xCB+PBGyAXJx/T/fOJxo3mPRNuC/U/AfFy7BKWn6AVOS2TV2+g=
```

## 🐧使用金鑰登入
- 對於自動化處理([[🐧Linux_RH294_01_Ansible介紹]])很重要，因為不可能讓腳本輸入密碼
	![Linux_RH124_14_SSH連線_04_自動化使用情境](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_14_SSH%E9%80%A3%E7%B7%9A_04_%E8%87%AA%E5%8B%95%E5%8C%96%E4%BD%BF%E7%94%A8%E6%83%85%E5%A2%83.png?raw=true)
- SSH驗證方式：
	1. 帳號 + 密碼
	2. 金鑰驗證：`ssh-keygen -t <指定演算方式>`
	3. kerberos驗證
- 步驟：
	1. 産生servera金鑰
	2. 將servera金鑰放至serverb中要金鑰登入用戶~/.ssh/authorized_keys
	3. 設置/etc/ssh/sshd_config登入選項

### 産生金鑰 ssh-keygen
- `-f <公鑰路徑>`，指定金鑰産出路徑

```bash
[root@localhost .ssh]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OeK4VBeA/lfmn0BPa/M1bMlVudi7KCGnlEgPTzzm2iM root@localhost.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|     ..         .|
|    .  .       ..|
|   .    o     o o|
|    .  o B+ .. o.|
|     .+ S=oo .o +|
|     +.+.Ooo=  B.|
|    o ..+ ++.+o.o|
|   . . E + .o... |
|    .   . . .    |
+----[SHA256]-----+
[root@localhost .ssh]# ll
total 8
-rw-------. 1 root root 2610 Mar 29 10:26 id_rsa
-rw-r-----. 1 root root  580 Mar 29 10:26 id_rsa.pub
```

![Linux_RH124_14_SSH連線_05_sshkeygen說明](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/03_Linux/%F0%9F%90%A7RH124/images/Linux_RH124_14_SSH%E9%80%A3%E7%B7%9A_05_sshkeygen%E8%AA%AA%E6%98%8E.png?raw=true)

### 公鑰登入目標主機 ssh-copy-id
將公鑰送至需登入的主機後使用`ssh-copy-id [-i <公鑰路徑>] <用戶名>@<IP|hostname>`
```bash
[root@localhost .ssh]# ssh-copy-id -i ./id_rsa.pub mickey@localhost
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "./id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
mickey@localhost's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'mickey@localhost'"
and check to make sure that only the key(s) you wanted were added.
[root@localhost .ssh]# cd /home/mickey/.ssh/
[root@localhost .ssh]# ll
total 8
-rw-------. 1 mickey student 580 Mar 29 10:36 authorized_keys
-rw-r--r--. 1 mickey student 342 Mar 23 10:20 known_hosts
```

### 登入選項設置
- SSH登入的設定檔：==/etc/ssh/sshd_config==
- 修改設定sshd.server需要重讀設定檔，`systemctl reload sshd`
```bash
# root可以用帳號 + 密碼登入
PermitRootLogin yes
# root不可登入
PermitRootLogin no
# root只能用金鑰登入
PermitRootLogin without-password

# 一般用戶設置，參數和root一樣
PasswordAuthentication yes
```