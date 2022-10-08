#💻編程/🎢版本控制/GIT
# GIT log、版本及分支
## 🧾查看log
* 可參考：[檢視提交的歷史記錄](https://git-scm.com/book/zh-tw/v2/Git-%E5%9F%BA%E7%A4%8E-%E6%AA%A2%E8%A6%96%E6%8F%90%E4%BA%A4%E7%9A%84%E6%AD%B7%E5%8F%B2%E8%A8%98%E9%8C%84)
* 多屏顯示控制方式：
	* J: 向下看一行
	* 空格：向下翻頁
	* K：向上看一行
	* B：向上翻頁
	* Q：退出

* `git log [文檔路徑]`，可看指定文檔commit log
	```console
	$ git log
	commit ca82a6dff817ec66f44342007202690a93763949
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

		changed the version number

	commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

		removed unnecessary test

	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 10:31:28 2008 -0700

		first commit
	```

* `git log --pretty=oneline [文檔路徑]`
	```console
	$ git log --pretty=oneline
	ca82a6dff817ec66f44342007202690a93763949 changed the version number
	085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
	a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
	```

* `git log --oneline [文檔路徑]`
	```console
	$ git log --oneline
	e12d8ef (HEAD -> master) add database.yml in config folder
	85e7e30 add hello
	657fce7 add container
	abb4f43 update index page
	cef6e40 create index page
	cc797cd init commit
	```

* `git reflog`
	```console
	$ git reflog
	657fce7 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~2
	e12d8ef (origin/master, origin/HEAD, cat) HEAD@{1}: checkout: moving from cat to master
	e12d8ef (origin/master, origin/HEAD, cat) HEAD@{2}: checkout: moving from master to cat
	```

## 🧾版本切換
### 指令
* 基於索引值操作(推薦) ：`git reset --hard <版本局部索引值>`
配合`git reflog`使用
* 使用^符號(版本只能退)
退回上一個版本：`git reset --hard HEAD^ [文檔路徑]`
退回上上個版本：`git reset --hard HEAD^^ [文檔路徑]`
* 使用~符號(版本只能退)：`git reset --hard HEAD~<退後的步數>`

### reset三個參數對比
*  \--soft：僅移動工作區的HEAD指針
*  \--mixed：移動暫存區和工作區的HEAD指針
*  \--hard：移動本地庫、暫存區和工作區的HEAD指針

### 文件比較差異
* 工作區比較暫存區：`git diff <fileName>`
* 工作區比較本地庫歷史版本：`git diff <版本局部索引值> <fileName>`
* `git diff `：所有有差異的文件
    
## 🧾分支操作
### 基本操作
建立分支：`git branch <branchName>`
查看分支：`git branch -v`
切換分支：`git checkout <branchName>`
合并分支`git marge <branchName>`
* 若分支branch1合并至分支master，則需切換至分支maser：`git checkout master`
* 合并分支：git marge branch1

### 解決沖突
1.  刪除沖突文件中的特殊符號要、編輯沖突文件並存檔
2.  編輯沖突文件至暫存區：`git add <fileName>`
3.  `git commit -m "日誌信息（內容隨意）"`，注意：此時commit一定不能帶具體文件名