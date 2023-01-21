# GIT_06_其他操作
## 🧾垃圾清理
清理不必要的檔案並優化本地儲存庫：`git gc`
`git gc --force`，即使可能有另一个git gc实例在此存储库上运行，也强制运行。
删除在本地有但在遠程库中已經不存在的分支：`git prune `

## 🧾git bisect
利用二元搜尋的方式協助使用者找到錯誤的 Commit，可參考：[使用 Git Bisect 快速找到第一個有問題的 Commit](https://www.gss.com.tw/blog/%E4%BD%BF%E7%94%A8-git-bisect-%E5%BF%AB%E9%80%9F%E6%89%BE%E5%88%B0%E7%AC%AC%E4%B8%80%E5%80%8B%E6%9C%89%E5%95%8F%E9%A1%8C%E7%9A%84-commit)
```bash
git bisect start # 開始搜尋
git bisect start HEAD 51dfd5c # 開始搜尋，並標記 HEAD 為壞的，51dfd5c 為好的
git bisect bad # 標記目前版本為壞的
git bisect good # 標記目前版本為壞的
git bisect reset # 停止搜尋 
```

## 🧾Git在eclipse操作(了解就好)
- 初始化本地庫：[參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=44)
- 設置本地庫簽名：[參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=45)
- 基本操作：
	- [Eclipse使用Git/GitHub](https://dotblogs.com.tw/cylcode/2019/01/08/111851)
	- [本地庫操作參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=49)
	- [推送遠端庫操作參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=50)
- git marge：
	- [Eclipse的Git插件Egit： merge合併衝突具體解決方法](https://read01.com/zh-tw/Jmzo76.html#.XgQXK0czZPY)
	- [參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=53)
- 分支操作：[參考視屏(bilibili)](http://www.bilibili.com/video/av67967014?p=56)
- git-extras：[git-extras - GitHub](https://www.google.com.tw/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiuoKa9yM38AhUDAYgKHU28BiEQFnoECAkQAQ&url=https%3A%2F%2Fgithub.com%2Ftj%2Fgit-extras&usg=AOvVaw0DFbrBwGrjm17oJwrQonhu)
