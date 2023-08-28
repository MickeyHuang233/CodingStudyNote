#📆2023年 
狀態:: #☑DONE 
完成日期:: 2023-08-25
標籤:: #💻編程/🖼前端/02_技術框架 #🗂Overview 
子筆記:: [[🅰️Angular_01_HelloWorld]], [[🅰️Angular_02_組件]], [[🅰️Angular_03_數據、指令綁定]], [[🅰️Angular_04_Pipe]], [[🅰️Angular_05_服務、依賴注入]], [[🅰️Angular_06_路由、導航]]
教程:: [教程](https://www.bilibili.com/video/BV1R54y1J75g)、[教程](https://www.bilibili.com/video/BV1tB4y1g7TS)、[尚硅谷](https://www.bilibili.com/video/BV1ts411E7qg)
備註:: 

# Angular
- Angular官網，[英文](https://angular.io/)、[繁體中文](https://angular.tw/)
- Chrome插件，[ng-inspect for AngularJS](https://chrome.google.com/webstore/detail/ng-inspect-for-angularjs/cidepfmbgngpdapgncfhpecbdhmnnemf)
- Angular為Google開源的前端JS結構化框架
- 開源的Angular UI元件：[Awesome Angular](https://github.com/PatrickJS/awesome-angular)

## 🅰️預備知識
- [[🟧HTML]]
- [[🟦CSS]]
- [[🟨JavaScript_00_Overview]]
- [[📦NPM]]
- [[🟦TypeScript_00_Overview]]，Angular從V2.x後開始用TypeScript編寫

## 🅰️Catalog
- [[🅰️Angular_01_HelloWorld]]
- [[🅰️Angular_02_組件]]
- [[🅰️Angular_03_數據、指令綁定]]
- [[🅰️Angular_04_Pipe]]
- [[🅰️Angular_05_服務、依賴注入]]
- [[🅰️Angular_06_路由、導航]]

## 🅰️核心概念
- 模塊(Module)
- 指令
- 組件(Component)
- 模版(Template)
- 數據綁定
- 服務
- 依賴注入(DI)

## 🅰️module、component關系
- Angular項目中，會將component放至module，module類似java中的包，component則類似於類
- module(模塊)概念：
	- Angular中的模塊就是一個抽象的容器，用於對組件進行分組
	- 整個應用初始時有且只有一個主模塊：AppModule
- component(組件)概念：
	- Angular中的組件就是一段可反復使用的頁面片段
	- 組件必須在模塊中聲明才可以使用
	- Component(組件) = Template(模板) + Script(腳本) + Style(樣式)

```bash
Angular
├── module_01
│   ├── component_01
│   ├── component_02
│   └── component_03
├── module_02
│   ├── component_04
│   ├── component_05
│   └── component_06
└── module_02
    ├── component_07
    ├── component_08
    └── component_09
```

## 🅰️項目啟動流程
1. 讀取angular.json，起動起始頁面、JavaScript起始原行檔
	```json
	"options": {
		"index": "src/index.html",
		"main": "src/main.ts",
	}
	```
2. 執行JavaScript起始原行檔，導入、啟動模塊
	```js
	// src/main.ts
	// 導入模塊
	import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
	import { AppModule } from './app/app.module';
	
	// 引導啟動模塊
	platformBrowserDynamic().bootstrapModule(AppModule)
	  .catch(err => console.error(err));
	```
3. 執行app.module.ts，導入、啟動組件
	```js
	// ./app/app.module
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	// 引入組件
	import { AppComponent } from './app.component';
	
	@NgModule({
	  declarations: [
	    AppComponent
	  ],
	  imports: [
	    BrowserModule
	  ],
	  providers: [],
	  // 啟動組件
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
4. 執行app.component.ts，並加載模版文件、樣式表文件
	```js
	// ./app.component
	import { Component } from '@angular/core';
	
	@Component({
	  // 組件指定CSS選擇器，相當於是<app-root>標籤
	  selector: 'app-root',
	  // 組件模版文件url
	  templateUrl: './app.component.html',
	  // 組件樣式表文件url
	  styleUrls: ['./app.component.css']
	})
	export class AppComponent {
	  title = 'my-app';
	}
	```
5. 顯示起始頁面src/index.html，並加載app.module模塊中的app.component組件
	```html
	<body>
	  <app-root></app-root>
	</body>
	```



