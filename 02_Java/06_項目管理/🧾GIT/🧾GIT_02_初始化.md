# GIT初始化
git資料初始化，生成.git文件夾：`git init`

## 🧾設置簽名
- 項目/倉庫級別：及在當前本地庫范圍內有效，信息保存至./.git/.gitconfig 。
	`git config user.name <yourUserName>`
	`git config user.email <yourEmail@email.com>`
- 系統用戶級別：登錄當前操作系統的用戶范圍，信息保存至~/.gitconfig。
	`git config --global user.name <yourUserName>`
	`git config --global user.email <yourEmail@email.com>`
- 級別優先級：
	1. 就近原則：項目級別>系統用戶級別，二者都有采用項目級別簽名。
	2. 若兩個級別的簽名都沒有則不允許。

## 🧾遠程庫操作
-  查看所有接結的遠端庫：`git remote -v`
-  連結遠端並把此遠端庫別名取為origin：`git remote add origin （貼上鏈接）`
-  第一次推入雲端：`git push origin master`

## 🧾解決與遠程庫的沖突
-  若不是基於遠程庫的最新版所做的修改，不會通過，必須先`git pull`
-  pull後若進入沖突狀態，解決方法和分支解決沖突一致
