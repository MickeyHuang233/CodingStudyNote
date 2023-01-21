# SVN
![GIT_00_Overview_01_SVN示意圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_00_Overview_01_SVN%E7%A4%BA%E6%84%8F%E5%9C%96.png?raw=true)

- 優點
	1. 權限管理容易，目錄可個別管理
	2. 指令少、易上手
- 缺點
	1. 本機只有當前版本的內容，若需要查看舊版本，需要從SVN Server下載
	2. 無法離線操作
	3. 建立分支成本太高
	4. 越來越少現代開發工具支援

# GIT
Git 指令說明文檔：https://git-scm.com/docs

## 🧾初次建立懶人操作
- `git init` ，資料初始化
- `git remote add origin <git link>`
- `git push -u origin master`
- `git status` ，檢查狀態，紅色代表還沒上傳集結區，綠色代表已上傳
- `git checkout <fileName>`，文件被修改了，但未执行git add的返回操作
- `git clean -f -d`
- `git add .`，全部加入集結區
- `git reset HEAD .`，全部取消加入集結區
- `git commit -m "<修改訊息>"`
- `git remote add origin <貼上鏈接>`，連結遠端
- `git push origin master`，資料推入雲端

## 🧾指令及操作
- [[🧾GIT_01_基本操作]]
- [[🧾GIT_02_初始化]]
- [[🧾GIT_03_log]]
- [[🧾GIT_04_版本、分支]]
- [[🧾GIT_05_開發模型]]
- [[🧾GIT_06_其他操作]]
- [[🧾GitLab]]

## 🧾Git原理
### Hash算法簡介
[hash是什麼？](https://blockbar.io/blockchain/hash%E6%98%AF%E4%BB%80%E9%BA%BC-what-is-hash/)
- 無論輸入數據的數據量有多大，輸入同的個Hash算法，得到的加密結果長度固定。
- Hash算法、輸入數據確定，輸出數據能夠保證不變。
- Hash算法不可逆。
- Git底層采用SIA-1算法。

### Git數據存儲機制
[Git 原理入門](https://ithelp.ithome.com.tw/articles/10190453)
![GIT_00_Overview_02_Git數據存儲機制](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_00_Overview_02_Git%E6%95%B8%E6%93%9A%E5%AD%98%E5%84%B2%E6%A9%9F%E5%88%B6.png?raw=true)
> Git把數據看作是小型文件系統的一組快照，每次提交更新時Git都會對當前的全部文件制作一個快照并保存這個快照的索引，為了高效，如果文件沒有修改，Git不再重新存儲該文件，而是只保留一個鏈接指向之前存儲的文件，所以Git工作方式可稱為快照流。
- Blob：Git將檔案內容產生hash(SHA1)儲存起來的物件。
- Tree：Git將文件夾下的檔名産生hash(SHA1)儲存起來的物件, 其中可能包含另一個Tree。
- Commit：Git將Tree和Commit相關信息產生hash(SHA1)儲存起來的物件。
- Parent：儲存前一個Commit的hash(SHA1)

### Git分支管理
分支的創建和切換只是HEAD指針指向的Commit不同而已

## 🧾問題記錄
- [git status顯示中文亂碼解決](https://blog.51cto.com/u_15072912/4150615)

