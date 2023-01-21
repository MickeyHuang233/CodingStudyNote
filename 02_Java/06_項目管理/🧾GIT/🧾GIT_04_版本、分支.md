# GIT 版本、分支
## 🧾版本切換
### 指令
- 基於索引值操作(推薦) ：`git reset --hard <版本局部索引值>`
	配合`git reflog`使用
- 使用`^`符號(版本只能退)
	1. 退回上一個版本：`git reset --hard HEAD^ [文檔路徑]`
	2. 退回上上個版本：`git reset --hard HEAD^^ [文檔路徑]`
- 使用~符號(版本只能退)：`git reset --hard HEAD~<退後的步數>`

### reset三個參數對比
| 參數      | 遠端倉庫 | 本地倉庫 | 暫存區                     | 工作區                           | HEAD指針                   |
| --------- | -------- | -------- | -------------------------- | -------------------------------- | -------------------------- |
| `--soft`  | 無變更   | 無變更   | 清除                       | 還原至<版本局部索引值>版本       | 切換至<版本局部索引值>版本 |
| `--mixed` | 無變更   | 無變更   | 清除                       | ==原本暫存區的變更退回至工作區== | 切換至<版本局部索引值>版本 |
| `--hard`  | 無變更   | 無變更   | ==從<版本局部索引值>還原== | 無變更                           | 切換至<版本局部索引值>版本 |

## 🧾分支操作
### 基本操作
- `git branch <branchName>`，建立本地分支
	1. `git branch <branchName> <版本局部索引值>`，在指定版本上建立分支
- `git branch -D <branchName> [branchName...]`，刪除本機分支
- `git branch -av`，查看所有分支(遠端/本地)版本
- `git checkout <branchName> [文檔路徑]`，切換分支，也可指定切換指定文檔

### 合并分支
- `git marge <branchName>`，將指定分支合并至當前分支，但合并完成後不會將分支刪除
	1. 若分支branch1合并至分支master，則需切換至分支maser：`git checkout branch1`
	2. 合并分支：`git marge branch1`
- `git merge --abort`，放棄合并並返回沖突前狀態

### 解決沖突
1. 刪除沖突文件中的特殊符號要、編輯沖突文件並存檔
	```bash
	<<<<<<< HEAD
	testEEE
	=======
	testEEE in issue002
	testEEE also in issue001
	so need to merge
	>>>>>>> 	issue002
	```
1. 編輯沖突文件至暫存區：`git add <fileName>`
2. `git commit -m "日誌信息（內容隨意）"`，注意：此時commit一定不能帶具體文件名

### 合并單次提交(小范圍)
使用情境：僅需合并此次提交修正的bug，但用merge會合并到許多不必要的程式

`git cherry-pick <版本局部索引值>`

### 備份工作區
當拉取遠端資料時，遠端和工作區有沖突，可以先狀工作區備份至stash再pull，最後在工作區解決沖突後commit

- `git stash`，備份當前工作區內容
- `git stash pop`，取得最近一次備份內容
- `git stash apply stash@{<備份編號>}`，取得指定編號的備份內谷
- `git stash list`，顯示所有備份
	```bash
	$ git stash list
	stash@{0}: WIP on main: f72b36a add III
	stash@{1}: WIP on main: a199244 merge issue002 into main
	```
- `git stash drop stash@{<備份編號>}`，刪除指定編號的備份內谷
- `git stash clear`，清空所有備份

### 遠端分支操作
- `git push --set-upstream origin <branchName>`，推送分支至遠端
- `git branch --set-upstream-to=origin/<branchName> <branchName>`，將本地分支設定至遠端分支
- `git push origin :<branchName>`，刪除遠端分支
- `git remote prune origin`，刪除已經沒有遠端分支的本機分支