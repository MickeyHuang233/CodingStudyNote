# Git基本操作
## 🧾文件操作
和[[🐧Linux_00_Overview]]操作一致
- `cd <folder>`，進入指定資料夾。
- `cd ..`，回父資料夾。
- `cd /`，回根資料夾。
- `dir`，顯示所有資料(windows cmd)。
- `ls`，顯示所在文件夾中所有資料，不顯示隱藏文件(git GUI)。
- `ls -lA`，顯示所在文件夾中所有資料，顯示隱藏文件(git GUI)。
- `cat <fileName>`，查看文件內容(git GUI)。
- `rm`，刪除文件

## 🧾求助指令
查看常見指令：`git --help`
查看單個指令的說明文檔：`git help <指命名>`

## 🧾Git基本操作
![GIT_01_基本操作_01_工作區常用指令](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_01_%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_01_%E5%B7%A5%E4%BD%9C%E5%8D%80%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4.png?raw=true)

- 狀態查看操作：`git status`
- 將指定文件工作區加到暫存區：`git add <fileName>`
- 將全部文件工作區加到暫存區：`git add .`
- 暫存區退回至工作區(new file)：`git rm --cached <fileName>`
- 暫存區退回至工作區(modify)：`git reset HEAD <fileName>`
- 將指定文件的修改撤銷到最近一次git add或git commit時的內容：`git checkout -- <fileName>`
- 將全部文件的修改撤銷到最近一次git add或git commit時的內容：`git checkout .`
- 提交至本地庫：
	1. `git commit -m "日誌信息（內容隨意）" <fileName>`
	2. `git commit -m "日誌信息（內容隨意）" `
- 提交至遠端庫：
	1. `git push <遠端庫別名> <branchName>`
- 下載遠程庫專案：`git clone <選端庫地址>`
	1. 完整的內容從遠端庫下載到本地
	2. 創建遠端庫別名
	3. 初始化本地庫(`git init`)
- 遠端庫拉取
	- `git pull`，相當於是`git fatch` + `git marge`的操作，所做的修改比較簡單(無沖突)時可使用
	- `git fatch <遠端庫別名> <遠程分支名>`
	- `git merge <遠端庫別名/遠程分支別名>`

## 🧾git ignore
- 忽略版本控制檔案設定檔
- 各語言`.ignore`樣本：[https://github.com/github/gitignore](https://github.com/github/gitignore)

## 🧾其他暫存
- [Git stash 暫存正在修改的內容](https://matthung0807.blogspot.com/2019/11/git-stash.html?m=0)
- [了解 GitHub 的 fork 與 pull request 版控流程](https://ithelp.ithome.com.tw/articles/10140305)
- 將別人的轉案複製至自己的遠端庫：在需複製的專案點擊【fork】按鈕
	pull requests-->create pull request
- SSH免密碼登入：[Windows使用ssh對Github進行操作](https://dotblogs.com.tw/kirkchen/2013/04/23/use_ssh_to_interact_with_github_in_windows)