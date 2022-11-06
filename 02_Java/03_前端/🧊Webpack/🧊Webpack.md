###### tags: `📆2022年` `💻編程/🖼前端` `🗂Overview`

# Webpack
- webpack提供前端模塊化開發支持、代碼壓縮、處理瀏覽器JavaScript兼容性、性能優化…功能
- 目前Vue、React…前端項目，基本上是基於Webpack進行工程化開發

## 🧊前端工程化
### 基本概念
- 模塊化：JS模塊化、CSS模塊化、資源模塊化
- 組件化：複用現在的UI結構、樣式、行為
- 規范化：目錄結構的劃分、編碼規范化、接口規范化、Git分支管理
- 自動化：自動化構建、自動部署、自動化測試

### 解決方案
- 早期解決方案
	- [Grunt](https://gruntjs.com/)
	- [Gulp](https://gulpjs.com/)
- 目前常用解決方案
	- [Webpack](https://webpack.js.org/)
	- [Parcel](https://parceljs.org/)

## 🧊環境安裝
### npm執行環境安裝
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

### HelloWorld程式準備
1. 建立內容為空的資料夾
2. `npm init -y`，初始化包管理配置文件package.json
```bash
D:\_Mickey_temp\webpack_test>npm init -y
Wrote to D:\_Mickey_temp\webpack_test\package.json:

{
  "name": "webpack_test",
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
3. 新建src源代碼目錄
4. `npm install jquery --save`，安裝JQuery，此時package.json的dependencies中就會有相應的設置
	- `--save`、`-S`，表示開發、生産環境都需要用到
	```bash
	D:\_Mickey_temp\webpack_test>npm install jquery -S
	
	added 1 package, and audited 2 packages in 1s
	
	found 0 vulnerabilities
	```
5. T005_HelloWorld.html
	```html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta http-equiv="X-UA-Compatible" content="IE=edge">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <title>T005_HelloWorld</title>
	    <script src="./index.js"></script>
	</head>
	<body>
	    <ul>
	        <li>this is num 1</li>
	        <li>this is num 2</li>
	        <li>this is num 3</li>
	        <li>this is num 4</li>
	        <li>this is num 5</li>
	        <li>this is num 6</li>
	        <li>this is num 7</li>
	        <li>this is num 8</li>
	        <li>this is num 9</li>
	    </ul>
	</body>
	</html>
	```
6. index.js
	```javascript
	// 使用ES6語法導入JQuery
	import $ from 'jquery'
	
	// 定義JQuery入口函數
	$(function(){
	    $('li:odd').css('background-color', yellow)
	    $('li:even').css('background-color', orange)
	})
	```

### Webpack安裝
參考：[Webpack本地安装](https://webpack.docschina.org/guides/installation/#local-installation)
搜索安裝包網站：[npmjs.com](https://www.npmjs.com/)

1. `npm install --save-dev webpack`，本地安裝最新版Webpack
	- `--save-dev`、`-D`，表示僅在開發環境下用到
2. `npm install --save-dev webpack-cli`，使用webpack 4+，需要另外安裝
	```bash
	D:\01_MickeyHuang\02_codeProject\20221031_webpack_test>npm install --save-dev webpack webpack-cli
	
	added 117 packages, and audited 119 packages in 17s
	
	15 packages are looking for funding
	  run `npm fund` for details
	
	found 0 vulnerabilities
	```

### Webpack配置
1. 項目根目錄建立webpack.config.js配置文件，參考：[webpack 設定檔案](https://webpack.docschina.org/configuration/)
    - development，表示産出的main.js文件無壓縮
    - production，表示産出的main.js文件會壓縮
	```javascript
	// 使用Node.js，向外導出webpack的配置對象
	module.exports = {
	    // webpack的運行模式(開發階段)，可選值為development、production
	    mode: 'development'
	}
	```
2. 在package.json中的`script`節點下，新增dev腳本
	```json
	(省略)
	// scripts中設置的腳本可以通過npm run執行
	  "scripts": {
	    "dev": "webpack" // npm run dev
	  },
	(省略)
	```
3. cmd執行`npm run dev`，啟動webpack執行項目打包、構建
	```bash
	D:\_Mickey_temp\20221031_webpack_test>npm run dev
	
	> 20221031_webpack_test@1.0.0 dev
	> webpack
	
	asset main.js 325 KiB [emitted] (name: main)
	runtime modules 937 bytes 4 modules
	cacheable modules 283 KiB
	  ./src/index.js 207 bytes [built] [code generated]
	  ./node_modules/jquery/dist/jquery.js 283 KiB [built] [code generated]
	webpack 5.74.0 compiled successfully in 612 ms
	```
4. html引入的js路徑應該為dist/底下的.js檔
	```html
    <!-- <script src="./index.js"></script> -->
    <script src="../dist/main.js"></script>
	```
5. 打包成功後的目錄結構
	![Webpack_01_webpack目錄結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_01_webpack%E7%9B%AE%E9%8C%84%E7%B5%90%E6%A7%8B.png?raw=true)

### 【非必要】指定Webpack輸入、輸出文件路徑
webpack.config.js

- `entry`，指定輸入文件，需要指定絕對路徑，默認打包的輸入文件為src/index.js
	```json
	// 導入path模塊
	const path = require('path')
	
	module.exports = {
	    // entry，指定輸入文件，需要為絕對路徑
	    // __dirname，表示此文件的目錄，也是開發專案目錄
	    // 單入口文件
	    entry: path.join(__dirname, "./src/T005_HelloWorld.js"),
	    
	    // 多入口文件，方式一
	    // 多個輸入文件，只會有一個輸出文件
	    entry: [
		    path.join(__dirname, "./src/test_01.js"),
		    path.join(__dirname, "./src/test_01.js"),
		    path.join(__dirname, "./src/test_03.js"),
		],
		
	    // 多入口文件，方式二
	    // 有幾個入口文件，就有幾個輸出文件
	    entry: {
		    testOne: path.join(__dirname, "./src/test_01.js"),
		    testTwo: path.join(__dirname, "./src/test_02.js"),
		    testThree: path.join(__dirname, "./src/test_03.js"),
	    },
	    output: {
	        path: path.join(__dirname, "dist"),
	        filename: '[name].js' // 使用entry的key命名輸出文件
	    },
	    
		// 特殊用法，混合使用
		entry: [
			mixOne: ["./src/test_01.js", "./src/test_02.js"],
			mixTwo: "./src/test_03.js"
		],
	}
	```
- `output`，輸出文件路徑，默認打包的輸出文件為dist/main.js
	```json
	// 導入path模塊
	const path = require('path')
	
	module.exports = {
	    // output，指定輸出文件，需要指定絕對路徑
	    output: {
	        // 存放的目錄
	        path: path.join(__dirname, "dist"),
	        // 生成的文件名
	        filename: 'T07_Output.js'
	    }
	}
	```

## 🧊Webpack核心概念
- entry：輸入，指示webpack以哪個文件作為起點打包、分析構建內部依賴圖；默認打包的輸入文件為src/index.js
- output：輸出，指示webpack打包後資源輸出到哪，以及輸出文件名稱；默認打包的輸出文件為dist/main.js
- loader：加載器，用於協助Webpack打包趨理特定的文件模塊，處理成webpack可以識別的資源
- plugins：插件，用於執行范圍更廠的任務
- mode：模式，指示webpack使用相應模式的配置打包
	- development，開發環境
	- production，生産環境(正式環境)

## 🧊常用插件 Plugin
### webpack-dev-server
使用express的Http服務器監聽項目源代碼是否變化，當修改源代碼，webpack會自動進行編譯、打包、構建

1. `npm install webpack-dev-server -D`，安裝插件
	```bash
	D:\_Mickey_temp\20221031_webpack_test>npm install webpack-dev-server -D
	
	added 187 packages, and audited 306 packages in 15s
	
	37 packages are looking for funding
	  run `npm fund` for details
	
	found 0 vulnerabilities
	```
2. 在package.json中的`script`節點下，修改腳本
	 ```json
	"scripts": {
		"//_dev": "webpack",
		"//": "使用webpack-dev-server插件，實現實時打包",
		"dev": "webpack server"
	},
	```
3. 在webpack.config.js設置webpack-dev-server服務端路徑
	```javascript
	module.exports = {
		// 設置webpack-dev-server服務端路徑
		devServer: {
			static:{
				directory: path.join(__dirname, "/"),
			},
		},
	}
	```
4. cmd執行`npm run dev`，啟動webpack執行項目打包、構建、插件
	- 注意：webpack-dev-server會啟動一個實時打包的http服務器
```bash
D:\_Mickey_temp\20221031_webpack_test>npm run dev
> 20221031_webpack_test@1.0.0 dev
> webpack server

<i> [webpack-dev-server] Project is running at:
<i> [webpack-dev-server] Loopback: http://localhost:8080/
<i> [webpack-dev-server] On Your Network (IPv4): http://192.168.202.222:8080/
<i> [webpack-dev-server] Content not from webpack is served from 'D:\_Mickey_temp\20221031_webpack_test\' directory
asset T07_Output.js 564 KiB [emitted] (name: main)
runtime modules 27.3 KiB 12 modules
modules by path ./node_modules/ 443 KiB
  modules by path ./node_modules/webpack-dev-server/client/ 55.8 KiB 12 modules
  modules by path ./node_modules/webpack/hot/*.js 4.3 KiB
	./node_modules/webpack/hot/dev-server.js 1.59 KiB [built] [code generated]
	./node_modules/webpack/hot/log.js 1.34 KiB [built] [code generated]
	+ 2 modules
  modules by path ./node_modules/html-entities/lib/*.js 81.3 KiB
	./node_modules/html-entities/lib/index.js 7.74 KiB [built] [code generated]
	./node_modules/html-entities/lib/named-references.js 72.7 KiB [built] [code generated]
	+ 2 modules
  ./node_modules/jquery/dist/jquery.js 283 KiB [built] [code generated]
  ./node_modules/ansi-html-community/index.js 4.16 KiB [built] [code generated]
  ./node_modules/events/events.js 14.5 KiB [built] [code generated]
./src/T005_HelloWorld.js 206 bytes [built] [code generated]
webpack 5.74.0 compiled successfully in 12579 ms
```
5. [http://localhost:8080/]()可查看結果，但為了避免多次在磁盤讀寫，webpack-dev-server會將産出的.js文件放至內存，可在[http://localhost:8080/T07_Output.js]()查看
	![Webpack_02_webpack-dev-server結果](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_02_webpack-dev-server%E7%B5%90%E6%9E%9C.png?raw=true)

### html-webpack-plugin
將指定文件複製一份至根目錄，並自動注入webpack-dev-server生成的內存js腳本，比較方便測試，不需到指定路徑下才可以看到頁面
```html
<!-- 注入的腳本 -->
<script defer="" src="T07_Output.js"></script>
```

1. `npm install html-webpack-plugin -D`，安裝插件
	```bash
	D:\_Mickey_temp\20221031_webpack_test>npm install html-webpack-plugin -D
	
	added 31 packages, and audited 337 packages in 9s
	
	47 packages are looking for funding
	  run `npm fund` for details
	
	found 0 vulnerabilities
	```
2. 在webpack.config.js配置html-webpack-plugin
	```javascript
	// 導入HTML插件，得到html-webpack-plugin構造函數
	const HtmlPlugin = require('html-webpack-plugin')

	// 有幾個html要壓縮就要新建幾個HtmlPlugin
	const htmlPlugin = new HtmlPlugin({
	    template: './src/T005_HelloWorld.html', // 指定原文件路徑
	    filename: './index.html', // 指定文件生成路徑
	    minify:{ // 壓縮html代碼
		    collapseWhitespace: true, // 移除空格
		    removeConnents: true, // 移除注釋
	    },
	    chunks: ['test01', 'test02'], // 引入的webpack産出.js名稱，如果沒指定，則會引入所有産出js檔
	})
	
	module.exports = {
	    mode: 'development',
	    plugins: [htmlPlugin], // webpack運行時，會加載數組內的插件
	}
	```
3. 插件會將文件生成至內存，因此只需訪問[http://localhost:8080/]()

### mini-css-extract-plugin
使用[[🧊Webpack#🧊常用加載器 Loader#css-loader]]時，會將樣式插入DOM中，放入`<head>`-->`<style>`，可以再配合使用min-css-extract-plugin插件提取成單獨css文件引入
1. `npm install mini-css-extract-plugin css-loader -D`，安裝插件
2. 在webpack.config.js配置mini-css-extract-plugin
	```javascript
	// 導入mini-css-extract-plugin
	const MiniCssExtractPlugin = require('mini-css-extract-plugin')
	
	module.exports = {
	    // webpack運行時，會加載數組內的插件
	    plugins: [
	        new MiniCssExtractPlugin({
	            filename: 'T014_CssLoader.css' // 指定輸出.css文件名，默認為main.css
	        }),
	    ], 
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader']},
	            {test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader']},
	            {test: /\.sass$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']},
	        ]
	    },
	}
	```
3. 啟動webpack執行項目打包後，css會用`<link>`的方式引入
	```html
	<link href="T014_CssLoader.css" rel="stylesheet">
	```

### css-minimizer-webpack-plugin
用於壓縮css的插件，可以配合[[🧊Webpack#🧊常用插件 Plugin#mini-css-extract-plugin]]使用，參考：[Webpack官方文檔](https://webpack.docschina.org/plugins/css-minimizer-webpack-plugin/)

1. `npm install css-minimizer-webpack-plugin -D`，安裝插件
2. 在webpack.config.js配置css-minimizer-webpack-plugin
	```javascript
	// 導入mini-css-extract-plugin
	const MiniCssExtractPlugin = require('mini-css-extract-plugin')
	// 導入css-minimizer-webpack-plugin
	const CssMinimizerWebpackPlugin = require("css-minimizer-webpack-plugin")
	
	// webpack運行時，會加載數組內的插件
    plugins: [
        htmlPlugin,
        new CleanWebpackPlugin(),
        new MiniCssExtractPlugin({
            filename: 'T014_CssLoader.css'
        }),
        new CssMinimizerWebpackPlugin(),
    ], 
    
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.css$/, use: ['style-loader', 'css-loader', 'postcss-loader']},
	            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader', 'postcss-loader']},
	            {test: /\.sass$/, use: ['style-loader', 'css-loader', 'sass-loader', 'postcss-loader']},
	        ]
	    }

	    optimization: {
	        // 開啟僅在生産環境下壓縮css
	        minimize: true,
	    },
	}  
	```

### 其他方便配置
- 初次打包完成，自動開啟瀏覽器
	```javascript
	// 配置html-webpack-plugin實現
	module.exports = {
	    devServer: {
	        open: true, // 初次打包完成，自動開啟瀏覽器
	        host: '127.0.0.1', // 實時打包的主機地址
	        port: 8081, // 實時打包的端口號
	    },
	}
	```

## 🧊常用加載器 Loader
- Webpack默認只能處理.js的模塊，而加載器用於協助Webpack打包趨理特定的文件模塊：
	- css-loader，用於打包.css相關文件
	- less-loader，用於打包.less相關文件
	- url-loader，用於打包url路徑相關文件
	- babel-loader，用於打包Webpack無法處理的高級JavaScript語法
- 如果沒有安裝相應的Loader，webpack會因為無法而報錯
	```bash
	ERROR in ./src/css/T014_CssLoader.css 1:2
	Module parse failed: Unexpected token (1:2)
	You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
	> li{
	|     list-style: none;
	| }
	 @ ./src/T005_HelloWorld.js 4:0-33
	
	webpack 5.74.0 compiled with 1 error in 56 ms
	```

### css-loader
1. `npm install style-loader css-loader -D`，安裝Loader
2. 在src/index.js中道入css文件
	```javascript
	// 導入css文件，在webpack中都為模塊
	import './css/T014_CssLoader.css'
	// require('./css/T014_CssLoader.css')
	```
3. 在webpack.config.js的`module`->`rules`節點中配置Loader，當Webpack無法處理時就會查找module.rules是否有對應加載器
	- 注意：Webpack會把符合匹配規則的文件交由後往前交付處理，也就是先給css-loader，處理結果再轉交給style-loader，Webpack會將處理好的結果一并合并至/dist/index.js打包
	- css-loader是處理css中的`@import`和url的外部資源
	- style-loader是將樣式插入DOM中，放入`<head>`-->`<style>`
	```javascript
	module.exports = {
		// 所有第三方文件的匹配規則
		module: {
		    rules: [ // 文件後綴名的匹配規則
		        {test: /\.css$/, use: ['style-loader', 'css-loader']},
		    ]
		}
	}
	```

### less-loader
1. `npm install less-loader less -D`，安裝Loader
2. 在src/index.js中導入less文件
	```javascript
	// 導入less文件
	import './less/T015_LessLoader.less'
	```
3. 在webpack.config.js的`module`->`rules`節點中配置Loader
	- 注意：less-loader內部依賴於less，因此在配置時僅需要配置less-loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.css$/, use: ['style-loader', 'css-loader']},
	            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']},
	        ]
	    }
	}
	```

### sass-loader
1. `npm install node-sass sass-loader -D`，安裝Loader
2. 在src/index.js中導入sass文件
	```javascript
	import './less/T015_SassLoader.less'
	import './css/T014_CssLoader.css'
	```
3. 在webpack.config.js的`module`->`rules`節點中配置Loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.css$/, use: ['style-loader', 'css-loader']},
	            {test: /\.sass$/, use: ['style-loader', 'css-loader', 'sass-loader']},
	        ]
	    }
	}
	```

### postcss-loader
用於處理css兼容性，參考：[尋覓 webpack - 22 - 真實世界的 webpack - 使用 Style](https://ithelp.ithome.com.tw/articles/10250227)

1. `npm install postcss-loader postcss-preset-env -D`，安裝Loader
2. 在src/index.js中導入less文件
	```javascript
	// 導入sass文件
	import './less/T015_SassLoader.less'
	```
3. 在webpack.config.js的`module`->`rules`節點中配置Loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.css$/, use: ['style-loader', 'css-loader', 'postcss-loader']},
	            {test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader', 'postcss-loader']},
				{test: /\.sass$/, use: ['style-loader', 'css-loader', 'sass-loader', 'postcss-loader']},
	        ]
	    }
	}
	```
4. 在postcss.config.js配置postcss-preset-env
	```javascript
	module.exports = {
	    plugins: [
	        require('postcss-preset-env'),
	    ],
	}
	```
5. 在package.json配置兼容瀏覽器范圍，參數官方說明：[Browserslist](https://github.com/browserslist/browserslist#full-list)
	```javascript
	{
	  "browerslist":[
	    "> 0.2%",
	    "last 2 version", 
	    "not dead"
	  ]
	}
	```

### url-loader
瀏覽器在取得html文件時，先取得標籤，真正讀取才會再發請求取得`src`中的資源，但這樣會發送過多請求而降低效能，最好的方式是將小圖轉為Base64
```html
<img src="./images/T017_Image.jpg" alt="" class="">
```

1. 在html新增`img`標籤
	```html
	<img src="" alt="" class="box">
	```
2. 在src/index.js中導入圖片文件，並給`img`標籤`src`賦值
	```javascript
	// 導入圖片，取得圖片文件
	import logo from './images/T017_Image.jpg'
	// 給img標簽動態賦值
	$('.box').attr('src', logo)
	```
3. `npm install url-loader file-loader -D`，安裝Loader
4. 在webpack.config.js的`module`->`rules`節點中配置Loader
	- 注意：url-loader內部依賴於file-loader，因此在配置時僅需要配置url-loader
	- `url-loader?<參數>=<參數值>`，設置loader的參數
	- `limit=222299`，指定圖片字節大小，只有容量在limit以內的圖片才會被轉為Base64，否則就為url路徑
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.jpg|png|gif$/, use: ['url-loader?limit=222299']},
	        ]
	    }
	}
	```

### babel-loader
參考：[babel-loader官方文檔](https://babel.docschina.org/docs/en/next/babel-plugin-proposal-decorators/)

1. 使用webpack無法處理的高級JS語法
	```javascript
	// 定義info裝飾器
	function info(target){
	    // 新增靜態屬性
	    target.info = 'Person info'
	}
	
	// 應用info裝飾器
	@info
	class Person{}
	
	// 打印靜態屬性
	console.log(Person.info)
	```
2. `npm install babel-loader @babel/core @babel/plugin-proposal-decorators -D`，安裝Loader
3. 在webpack.config.js的`module`->`rules`節點中配置Loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            // babel-loader必須定義排除項，因為node_modules目錄下的第三方包不需要打包，只需打包自己開發的包即可
	            {test: /\.js$/, use: ['babel-loader'], exclude: /node_modules/},
	        ]
	    }
	}
	```
4. 在項目根目錄下，建立babel.config.js，並定義babel配置信息
	```javascript
	{
	    // 聲明babel可用插件, webpack調用label-loader時會先加載plugins中的插件使用
	    "plugins": [
	        ["@babel/plugin-proposal-decorators", { "legacy": true }]
	    ]
	}
	```
5. 項目目錄結構
	![Webpack_03_babel-loader目錄結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_03_babel-loader%E7%9B%AE%E9%8C%84%E7%B5%90%E6%A7%8B.png?raw=true)

### eslint-loader
eslint為開源JavaScript檢查工具，主要用於開發團隊內編碼風格一致
配置eslint-loader參考：[https://ithelp.ithome.com.tw/articles/10215259]()

1. `npm install eslint-loader eslint eslint-config-airbnb-base eslint-plugin-import -D`，安裝Loader
2. 在webpack.config.js的`module`->`rules`節點中配置Loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {
	                // eslint只檢查js語法
	                test: /\.js$/,
	                // 排除檢查第三方庫的代碼
	                exclude: /node_modules/,
	                loader: 'eslint-loader',
	                options: {
	                    fix: true, // 在不影響業務邏輯的情況下修復
	                },
	            },
	        ]
	    }
	}
	```
3. 在package.json配置代碼風格規則，[airbnb-base github](https://github.com/airbnb/javascript)
	```json
	{
	  "eslintConfig": {
		"//": "繼承airbnb-base的代碼風格",
	    "extends": "airbnb-base"
	  }
	}
	```
4. cmd執行`npm run dev`，啟動webpack執行項目打包時就會對不符合代碼風格報錯
	```bash
	ERROR in ./src/T005_HelloWorld.js
	Module Error (from ./node_modules/eslint-loader/dist/cjs.js):
	
	D:\01_MickeyHuang\02_codeProject\20221031_webpack-test\src\T005_HelloWorld.js
	  29:1  error  Parsing error: Unexpected character '@'
	
	✖ 1 problem (1 error, 0 warnings)
	
	
	webpack 5.74.0 compiled with 1 error in 10512 ms
	```
5. 可以使用注解排除eslint的風格檢查
	參考：[How to disable ESLint for some lines, files or folders](https://learn.coderslang.com/0023-eslint-disable-for-specific-lines-files-and-folders/)
	```javascript
	// 應用info裝飾器
	// 表示下一行不用eslint檢查規則
	// eslint-disabel-next-line
	console.log(a + b);
	```

### 打包其他資源
不需要壓縮處理、直接輸出的資源，如：字體…，統稱其他資源

- 在webpack.config.js的`module`->`rules`節點中配置Loader
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則{
                // 排除其他已處理過資源
                exclude: /\.(js|json|html|css|less|sass|png|gif|jpg|jpeg)$/,
                loader: 'file-loader',
                options: {
                    // 輸出路徑
                    outputPath: 'otherFile/',
                    // 輸入路徑
                    publicPath: './otherFile',
                    // 資源名規則
                    // [name]，原檔名
                    // [hash:8]，檔案hash前8碼
                    // [ext]，原副檔名
                    name: '[name][hash:8].[ext]',
                },
            },
	        ]
	    }
	}
	```

## 🧊打包發布
### 配置打包指令
1. 在package.json，`scripts`節點下新增build命令
	- `--mode`，指定webpack的運行模式，`production`表示生産環境，打包時會執行代碼壓縮、性能優化，在此指定的參數在覆蓋webpack.config.js中的選項
	```json
	(省略)
	  "scripts": {
	    "//": "項目發布時，運行build命令",
	    "build": "webpack --mode production"
	  },
	(省略)
	```

### 打包資源整理
1. 將所有.js文件統一放入js目錄中
	```javascript
	module.exports = {
	    output: {
	        // 存放的目錄
	        path: path.join(__dirname, "dist"),
	        // 生成的文件名
	        filename: 'js/T07_Output.js'
	    },
	}
	```
2. 將所有圖片統一放入image目錄中，webpack.config.js中新增`outputPath`參數，webpack會將所有超過`limit`指定容量的圖片放至`outputPath`指定的路徑
	```javascript
	module.exports = {
	    // 所有第三方文件的匹配規則
	    module: {
	        rules: [ // 文件後綴名的匹配規則
	            {test: /\.jpg|png|gif$/, use: ['url-loader?limit=100&outputPath=images']},
	        ]
	    }
	}
	```

### 自動清理打包目錄插件
配置clean-webpack-plugin插件後，每次打包前都會自動清理打包目錄，避免新舊文件混合導致錯誤，參考配置：[Clean plugin for webpack - npmjs](https://www.npmjs.com/package/clean-webpack-plugin)

1. `npm install --save-dev clean-webpack-plugin`，安裝插件
2. 在webpack.config.js配置clean-webpack-plugin
	```javascript
	// 配置clean-webpack-plugin構造函數
	// 注意：左側的{}是結構賦值
	const { CleanWebpackPlugin } = require('clean-webpack-plugin');
	
	module.exports = {
	    // webpack運行時，會加載數組內的插件
	    plugins: [
	        new CleanWebpackPlugin(),
	    ], 
	}
	```

## 🧊環境優化
### 開發環境優化：模塊熱替換
Hot Modle Replacemant(HMR，模塊熱替換)為[[🧊Webpack#🧊常用插件 Plugin#webpack-dev-server]]的功能，它允許運行時更新單個模塊，無需進行完全刷新，可參考官方文檔：[devServer.hot](https://webpack.docschina.org/configuration/dev-server/#devserverhot)
```javascript
module.exports = {
    // 設置webpack-dev-server服務端路徑
    devServer: {
        static:{
            directory: path.join(__dirname, "/"),
        },
        open: true, // 初次打包完成，自動開啟瀏覽器
        host: '127.0.0.1', // 實時打包的主機地址
        port: 8081, // 實時打包的端口號
        hot: true, // 開啟模塊熱加載， webpack-dev-server v4後默認開啟
    },
}
```

### 生産環境優化：刪除無用代碼
- Webpack使用tree-shaking刪除沒被調用的JS代碼，可參考官方文檔：[Tree Shaking](https://webpack.docschina.org/guides/tree-shaking/)
	1. 需使用ES6模塊化
	2. 開啟production環境
- Webpack使用purgecss-webpack-plugin刪除沒被調用的CSS代碼，需要配合[[🧊Webpack#🧊常用插件 Plugin#mini-css-extract-plugin]]使用，可參考：[purgecss-webpack-plugin npmjs](https://www.npmjs.com/package/purgecss-webpack-plugin)

## 🧊SourceMap
SourceMap就是儲存位置信息的信息文件。webpack轉換、壓縮、打包後的若有錯誤信息或log打印，顯示的行數為壓縮打包後的行數，不易於debug，使用SourceMap後會將打包前的原始碼位置對應打包後的位置，便於後期調試
![Webpack_04_未配置SourceMap的錯誤信息](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_04_%E6%9C%AA%E9%85%8D%E7%BD%AESourceMap%E7%9A%84%E9%8C%AF%E8%AA%A4%E4%BF%A1%E6%81%AF.png?raw=true)

### 開發模式下開啟SourceMap
1. 在webpack.config.js配置SourceMap
	```javascript
	module.exports = {
	    // 開發調試時建議設置，以便debug
	    devtool: 'eval-source-map',
	    // devtool: 'source-map',
	}
	```
1. 配置完成後，webpack重新打包後就會顯示正確行號
	![Webpack_05_配置後SourceMap的錯誤信息](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_05_%E9%85%8D%E7%BD%AE%E5%BE%8CSourceMap%E7%9A%84%E9%8C%AF%E8%AA%A4%E4%BF%A1%E6%81%AF.png?raw=true)

### 發布關閉SourceMap
1. 當`mode`為production時，出於安全性考量關閉`devtool`，最終生成的文件不包含SourceMap，為了避免暴露原始碼
2. 折中的方式是只定位行數不暴露源始碼
	```javascript
	module.exports = {
	    // 只定位行數不暴露源碼
	    devtool: 'nosources-source-map',
	}
	```
3. 配置完成後，webpack重新打包後就會顯示正確行號，但無法從連結取得源始碼
	![Webpack_06_配置僅顯示行號SourceMap的錯誤信息](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%A7%8AWebpack/images/Webpack_06_%E9%85%8D%E7%BD%AE%E5%83%85%E9%A1%AF%E7%A4%BA%E8%A1%8C%E8%99%9FSourceMap%E7%9A%84%E9%8C%AF%E8%AA%A4%E4%BF%A1%E6%81%AF.png?raw=true)

## 🧊其他
### 以@顯示專案路徑
1. 用相對路徑找資源比較不直觀
	```javascript
	import logo from '../../../images/T017_Image.jpg'
	```
2. 在webpack.config.js配置
	```javascript
	// 導入path模塊
	const path = require('path')
	
	module.exports = {
	    // 設置@表示./src目錄
	    resolve: {
	        alias: {
	            '@': path.join(__dirname, './src')
	        }
	    }
	}
	```
3. 配置後就可以直接用@表示./src
	```javascript
	import logo from '@/images/T017_Image.jpg'
	```
