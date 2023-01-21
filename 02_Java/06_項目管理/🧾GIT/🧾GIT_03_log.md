# GIT_03_log
## 🧾查看log
- 可參考：[檢視提交的歷史記錄](https://git-scm.com/book/zh-tw/v2/Git-%E5%9F%BA%E7%A4%8E-%E6%AA%A2%E8%A6%96%E6%8F%90%E4%BA%A4%E7%9A%84%E6%AD%B7%E5%8F%B2%E8%A8%98%E9%8C%84)
- 多屏顯示控制方式：
	- J: 向下看一行
	- 空格：向下翻頁
	- K：向上看一行
	- B：向上翻頁
	- Q：退出

- `git log [文檔路徑]`，可看指定文檔commit log
	- `-n <筆數>`，只顯示最新n筆的commit
	- `-S <查詢字串>`，查詢提交內容有指定字串(區分大小寫)
	- `--since=2023/01/15`
	- `--until=2023/01/16`
	- `--author=<查詢字串>`，查詢提交作者字串(區分大小寫)
	```bash
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

- `git log --pretty=oneline [文檔路徑]`
	```bash
	$ git log --pretty=oneline
	ca82a6dff817ec66f44342007202690a93763949 changed the version number
	085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
	a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
	```

- `git log --oneline [文檔路徑]`
	```bash
	$ git log --oneline
	e12d8ef (HEAD -> master) add database.yml in config folder
	85e7e30 add hello
	657fce7 add container
	abb4f43 update index page
	cef6e40 create index page
	cc797cd init commit
	```

- `git reflog`
	```bash
	$ git reflog
	657fce7 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~2
	e12d8ef (origin/master, origin/HEAD, cat) HEAD@{1}: checkout: moving from cat to master
	e12d8ef (origin/master, origin/HEAD, cat) HEAD@{2}: checkout: moving from master to cat
	```

- `git log --oneline --graph --decorate --all`，查看分支圖
	1. `--oneline`，單行顯示log
	2. `--all`，顯示所有分支log
	```bash
	mickey@MickeyNB MINGW64 /d/_Mickey_temp/20230116_git (issue002)
	$ git log --oneline --graph --decorate --all
	*   2a36907 (HEAD -> main) Merge branch 'issue001' into main
	|\
	| * e47eb66 (origin/issue001, issue001) add FFF
	| * c98a37a add EEE
	* | 0534254 (origin/main) add CCC.txt
	| | * a9e8f37 (origin/issue002, issue002) add EEE
	| | * 126eedd add GGG
	| | * dfd495e add DDD
	| |/
	|/|
	* | c8d3b4c add BBB.txt
	* | 374185c add AAA.txt
	|/
	* b21fe9d first commit
	```

## 🧾查看圖形歷史檢視器
`gitk --all`，呼叫圖形歷史檢視器，查看所有分支

## 🧾查看提交記錄
- 可借此指令取得遺失的版本號，並使用`git reset`還原
- 因為已commit但還未push的版本未被任何分支占用，因此切換至其他分支時從`git log`無法取得

```bash
$ git reflog

a199244 (HEAD -> main) HEAD@{0}: reset: moving to a199244
110eed2 (origin/main) HEAD@{1}: reset: moving to 110eed2
110eed2 (origin/main) HEAD@{2}: reset: moving to 110eed2
a199244 (HEAD -> main) HEAD@{3}: reset: moving to a199244
110eed2 (origin/main) HEAD@{4}: reset: moving to 110eed2
a199244 (HEAD -> main) HEAD@{5}: reset: moving to a199244
a199244 (HEAD -> main) HEAD@{6}: reset: moving to a199244
a199244 (HEAD -> main) HEAD@{7}: reset: moving to HEAD
```

## 🧾查看文檔修改記錄
- `git blame <文檔路徑>`
	```bash
	MickeyHuang@MickeyPC MINGW64 /d/_Mickey_temp/myPrivateWebsite (master)
	$ git blame index.html | head
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   1) <!DOCTYPE html>
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   2) <html lang="en">
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   3)
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   4) <head>
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   5)     <meta charset="UTF-8">
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   6)     <meta name="viewport" content="width=device-width">
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   7)     <title>Michongchong</title>
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   8)     <link rel="stylesheet" href="css/global.css">
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800   9)     <link rel="stylesheet" href="css/index_desktop.css">
	^47fb421 (MiChongZi 2017-04-25 12:00:13 +0800  10)     <link rel="stylesheet" href="css/index_phone.css">
	```
- `git blame <文檔路徑> -L 10,20`，查看10~20行的修改記錄
- `git blame <文檔路徑> -L 10,+20`，查看10行後的20行修改記錄，也就是`-L 10,30`

## 🧾查看文件差異
- 工作區比較本地庫歷史版本：`git diff <版本局部索引值> <fileName>`
- `git diff [fileName]`，查看工作區有差異的文件
- `git diff --cached [fileName]`，查看==暫存區==有差異的文件
- `git diff --name-status`，查看修改、刪除的文件列表，因為新增的檔案並未給git管理，所以要將檔案放入暫存區才會列出
	```bash
	$ git diff --name-status
	M       AAA.txt
	D       FFF.txt
	$ git diff --name-status --cached
	M       AAA.txt
	D       FFF.txt
	A       JJJ.txt
	```