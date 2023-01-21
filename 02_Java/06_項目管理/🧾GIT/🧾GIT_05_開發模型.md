# GIT_05_開發模型
## 🧾存取模型
和其他開發者共同協作時的存取模型
### 分散式貢獻者模型 Dispersed
![GIT_05_開發模型_01_分散式貢獻者模型](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_01_%E5%88%86%E6%95%A3%E5%BC%8F%E8%B2%A2%E7%8D%BB%E8%80%85%E6%A8%A1%E5%9E%8B.png?raw=true)

- 特點
	- Linux kernel目前使用此模型進行版本推進
	- 使用.diff向owner提供差異檔
	- 可以不使用版本控制軟體
	- 以完整概念開發
- 優點
	- 強制code review
- 缺點
	- 分享成果變得複雜
	- 貢獻者需自行架設版本管理系統

### 共處一地貢獻者模型 Collocated
![GIT_05_開發模型_02_共處一地貢獻者模型](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_02_%E5%85%B1%E8%99%95%E4%B8%80%E5%9C%B0%E8%B2%A2%E7%8D%BB%E8%80%85%E6%A8%A1%E5%9E%8B.png?raw=true)

- 特點
	- 上游專案有完整控制權
	- author在自己的遠端倉庫建立複本
	- author需要在自己的遠端倉庫修改完後，提交請求至上游專案，上游專案owner審核後才可進行更新
	- GitHub為主流
- 優點
	- 強制code review
- 缺點
	- 過多commit造成除錯困難
	- 每個成員都要建立自己的遠端倉庫複本
	- 分享成果變得複雜

### 共享維護模型 Shared
![GIT_05_開發模型_03_共享維護模型](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_03_%E5%85%B1%E4%BA%AB%E7%B6%AD%E8%AD%B7%E6%A8%A1%E5%9E%8B.png?raw=true)

- 特點
	- 每個author都有寫入權限，因此大家都要有互信基礎
- 優點
	- 鼓勵清楚的master分支
- 缺點
	- 無強制code review
	- 每個人都有寫入權限

## 🧾主線、分支開發模型
實際開發時如何管理分支

### 主線開發
- 最容易理解、最簡單
- 將所有開發者在單一分支上開發

### 主線搭配分支開發
- 特點
	- 當有成果或是整合測試後，則會將分支合并至master
	- 主線使用CI/CD交付整合成果
- 分類
	- 依開發者建立分支開發
	- 依功能建立分支開發【常用】
- 優點
	- 主線都可以部署，比較不會發生緊急修正
- 缺點
	- 持續部署頻繁，比較適合用在自動更新情境
	- 若開發完成後沒馬上合并，要花費額外心力維持分支一致性

## 🧾版控流程
### Git flow
![GIT_05_開發模型_04_GitFlow](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_04_GitFlow.png?raw=true)

- master，上線分支，永遠處於可使用狀態，如：正式區
- hotfix，緊急修復分支，系統上線後需緊急修復bug的分支
- release，發佈用分支，發佈前測試用分支，若有bug在此修正，如：UAT
- develop，開發分支，類似於SIT分支
- feature，依功能/開發者建立的分支，完成後合并至develop

### Github flow
相較於Git flow簡單很多，開發完成後只要將分支合并至master，由master發布即可，但會比較要求工程師的代碼品質

![GIT_05_開發模型_05_GithubFlow](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_05_GithubFlow.png?raw=true)

### Gitlab flow
開發完成後只要將分支合并至master，當master測試完成後再合并至pre-production，當pre-production測試完成後再合并至production

![GIT_05_開發模型_06_GitlabFlow](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/06_%E9%A0%85%E7%9B%AE%E7%AE%A1%E7%90%86/%F0%9F%A7%BEGIT/images/GIT_05_%E9%96%8B%E7%99%BC%E6%A8%A1%E5%9E%8B_06_GitlabFlow.png?raw=true)

- master，如：SIT分支
- pre-production，如：UAT分支
- production，如：正式區分支
