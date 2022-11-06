#📆2022年 
狀態:: #☑DONE  
完成日期:: 2022-11-03
標籤:: #💻編程/🖼前端 #🗂Overview 
子筆記:: 
教程:: [教程](https://www.bilibili.com/video/BV1Pz4y1S7Uv)
備註:: 

# NPM
- Node.js官網：[https://nodejs.org/en/]()
- npm官網：[https://www.npmjs.com/]()
- Node Package Manager(NPM，node包管理器)，是Node.js默認、以JavaScript編寫的包管理系統

## 📦環境安裝
1. 下載安裝檔，[node.js](https://nodejs.org)
2. 使用默認設定安裝
3. `cmd`確認安裝版本
	```bash
	C:\Users\mickey>node -v
	v18.12.0

	C:\Users\mickey>npm -v
	8.19.2
	
	C:\Users\mickey>npm -h
	npm <command>
	
	Usage:
	
	npm install        install all the dependencies in your project
	npm install <foo>  add the <foo> dependency to your project
	(以下省略)
	```

## 📦基本使用
### 鏡像相關指令
- 查看境像配置
	- `npm config get registry`
	- `npm config get disturl`
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm config get registry
	https://registry.npmjs.org/
	
	D:\_Mickey_temp\20221103_npm-test>npm config get disturl
	undefined
	```
- `npm config set registry <鏡像url> [--global]`，設置npm鏡像
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm config set registry https://registry.npm.taobao.org/
	```
- `npx nrm use <鏡像源>`，使用nrm工具切換鏡像源(使用前需要先安裝)
	```bash
	D:\_Mickey_temp\20221103_npm-test>npx nrm use taobao
	   Registry has been set to: https://registry.npm.taobao.org/
	```
- `npx nrm use npm`，換回官方鏡像源
	```bash
	D:\_Mickey_temp\20221103_npm-test>npx nrm use npm
	   Registry has been set to: https://registry.npmmirror.com/
	```

### 常用指令
- `npm -v`，查看版本
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm -v
	8.19.2
	```
- `npm help`
- `npm install <模塊名>@<版本號>`，安裝模塊，無指定版本號則為最新版
	- `--save`，寫入生産環境依賴，會寫入package.json的`dependencies`節點中
	- `--save-dev`，寫入開發環境依賴，會寫入package.json的`devDependencies`節點中
	- 若專案資料夾中已有package.json，使用`npm install`會根據package.json中的配置下載插件
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm install jquery
	
	added 1 package, and audited 2 packages in 3s
	
	found 0 vulnerabilities
	```
- `npm list -a`，查看所有安裝模塊
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm list -a
	20221103_npm-test@ D:\_Mickey_temp\20221103_npm-test
	-- jquery@3.6.1
	```
- `npm list <模塊名>`，查看指定模塊版本號
- `npm update <模塊名>`，更新指定模塊為最新版
- `npm uninstall <模塊名>`，卸載指定模塊
- `npm init --yes`，初始化包管理配置文件package.json
	```bash
	D:\_Mickey_temp\20221103_npm-test>npm init --yes
	Wrote to D:\_Mickey_temp\20221103_npm-test\package.json:
	
	{
	  "name": "20221103_npm-test",
	  "version": "1.0.0",
	  "description": "",
	  "main": "index.js",
	  "scripts": {
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [],
	  "author": "",
	  "license": "ISC"
	}
	```

## 📦package.json屬性
[package.json官方文檔說明](https://docs.npmjs.com/cli/v8/configuring-npm/package-json)

- name，包名
- version，包的版本號
- description，包描述
- homepage，包的官網url
- author，包的作者姓名
- contributor，包的其他貢獻者姓名
- dependencies，依賴包列表，如果依賴包沒安裝，npm會自動將依賴包安裝在node_module下，版本號表示含義
	- `5.11.2`，表示安裝指定版本
	- `~5.11.3`，表示安裝5.11.X中最新的版本
	- `^5.11.3`，表示安裝5.X.X中最新的版本
- repository，包代碼存放地方類型，git / svn
- main，程序的主入口文件，`requie('moduleName')`就會加載此文件，默認值為index.js
	```javascript
	// index.js
	 
	// 不需要在每個html文件加上<scirpt src="......jquery.js" />
	const $ = require('jquery')
	console.log($.get())
	```
- keywords，關鍵字

## 📦擴展：Yarn包管理工具
- Yarn是由Facebook、Google、Exponent、Tilde聯合推出的JS包管理工具，是為了彌補npm的缺陷而出現的
- npm5以下會出現以下問題
	1. `npm install`速度非常慢
	2. 同一項目多人開發時，由於安裝版本不一致出現bug
- Yarn官網：[https://yarnpkg.com/]()

### 環境安裝
1. 下載安裝檔，[node.js](https://nodejs.org)
2. 使用默認設定安裝
3. `cmd`確認安裝版本
	```bash
	node -v
	npm -v
	npm -h
	```
4. `npm install -g yarm`，使用npm安裝
5. `yarn --version`，查看yarn版本
6. 【非必要】`yarn config set registry <鏡像url>`，設置鏡像源

### 常用指令
操作和npm類似
- `yarn init`，初始化包管理配置文件package.json
- `yarn add <包名>@<版本號> --exact`，安裝包的精確版本
- `yarn add <包名>@<版本號> --tilde`，安裝包的次要版本裡的最新版，如`yarn add test@5.11.2 --tilde`表示安裝5.11.X中最新的版本
- `yarn publish`，發布包
- `yarn remove <包名>`，移除包，會自動更新package.json和yarn.lock
- `yarn upgrade [包名 | 包名@版本號]`，更新指定包版本，若無指定則更新所有作用域包
- `yarn run <腳本名>`，執行在package.json中`script`節點下定義的腳本
- `yarn info <包名>`，查看某個包的信息
- `yarn cache list`，而出已緩存的每個包
- `yarn cache dir`，返回全局緩存位置
- `yarn cache clean`，清除緩存
