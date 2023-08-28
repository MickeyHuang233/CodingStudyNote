# Angular服務、依賴注入
- 組件；是與用戶交互的對象
- 服務，是與用戶操作無關的對象，為組件服務，如：日志記錄、計時統計、後端服務器訪問…等
- 服務檔命名建議為`xxx.service.ts`，以便識別
- 注入的概念類似於Spring框架中的注入，可參考：[[🍃Spring_00_Overview]]

## 🅰️new服務對象
1. 建立服務對象
	```js
	export class T30LogService{
	    // 打印Log
	    printLog(action : string){
	        let username = 'mickey'
	        let time = new Date().getTime()
	        console.log(
	            'username:' + username +', ' + //
	            'time: ' + time + ', ' + //
	            'action: ' + action //
	        )
	    }
	}
	```
2. `new 對象名()`，建立服務對象的實例並調用
	```js
	import { Component } from '@angular/core';
	import { T30LogService } from '../t30.service';
	
	@Component({
	  selector: 'app-t30c01',
	  templateUrl: './t30c01.component.html',
	  styleUrls: ['./t30c01.component.css']
	})
	export class T30c01Component {
	  doAdd(){
	    let logservice = new T30LogService()
	    logservice.printLog('doAdd()') // 調幅對象方法
	  }
	}
	```

## 🅰️新建服務(NPM工具)
`[npx] ng generate service <服務名>`，使用npm(第三方模塊維護工具)建立Angular服務

```bash
PS D:\01_MickeyHuang\02_codeProject\20230815_AngularTry\my-app> npx ng g service t30s01
CREATE src/app/t30s01.service.spec.ts (357 bytes)
CREATE src/app/t30s01.service.ts (135 bytes)
```

## 🅰️依賴注入(Dependency Injection)
### 單例注入
1. 建立服務對象，並用`@Injectable`修飾，指定`providedIn`表示是以單例進行注入
	```js
	import { Injectable } from "@angular/core"
	
	// 表示此類是可被注入的，以單例進行注入
	@Injectable({
	    // 指定當前服務對象在"根模塊"提供
	    // 根模塊表示AppModule
	    providedIn: 'root'
	})
	export class T30LogService{
	
		constructor() {
		// 多次注入都只會調用一次
		console.log('T30LogService.constructor()')
		}
	  
	    // 打印Log
	    printLog(action : string){
	        let username = 'mickey'
	        let time = new Date().getTime()
	        console.log(
	            'username:' + username +', ' + //
	            'time: ' + time + ', ' + //
	            'action: ' + action //
	        )
	    }
	}
	```
2. 在建構子聲明依賴後即可調用
	```js
	import { Component } from '@angular/core';
	import { T30LogService } from '../t30.service';
	
	@Component({
	  selector: 'app-t30c01',
	  templateUrl: './t30c01.component.html',
	  styleUrls: ['./t30c01.component.css']
	})
	export class T30c01Component {
	  // 聲明變量
	  diLogService: T30LogService
	  // 聲明依賴，相當於是注冊
	  constructor(diLogService: T30LogService){
	    this.diLogService = diLogService
	  }
	
	  doAdd(){
	    // 使用建立服務對象調用
	    let logservice = new T30LogService()
	    logservice.printLog('doAdd()')
	  }
	
	  doDelete(){
	    // 使用依賴注入調用
	    this.diLogService.printLog('doDelete()')
	  }
	}
	```

### 多例注入
1. 建立服務對象，並用`@Injectable`修飾
	```js
	import { Injectable } from '@angular/core';
	
	@Injectable()
	export class T30s01Service {
	
	  constructor() {
		// 每次注入都會調用一次
	    console.log('T30s01Service.constructor()')
	  }
	
	  printLog(){
	    console.log('T30s01Service.printLog()')
	  }
	}
	```
2. 在`@Component`中聲明注入服務，在建構子聲明依賴後即可調用
	```js
	import { Component } from '@angular/core';
	import { T30s01Service } from '../t30s01.service';
	
	@Component({
	  selector: 'app-t35c01',
	  templateUrl: './t35c01.component.html',
	  styleUrls: ['./t35c01.component.css'],
	  providers: [T30s01Service] // 僅用於當前組件的服務提供者
	})
	export class T35c01Component {
	  // service: T30s01Service
	  // constructor(service: T30s01Service){
	  //   this.service = service
	  // }
	  // 相當於以上寫法
	  constructor(private service: T30s01Service){  }
	
	  doSomething(){
	    this.service.printLog()
	  }
	}
	```

## 🅰️Angular內置服務
[所有服務 - Angular API參考手冊](https://angular.tw/api?type=class)

### HttpClient
參考：[HttpClient - Angular API參考手冊](https://angular.tw/api/common/http/HttpClient)

1. 引入HttpClient服務所在的模塊`HttpClientModule`
	```js
	// 引入模塊
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { HttpClientModule }    from '@angular/common/http';
	import { AppComponent } from './app.component';
	import { T32c01Component } from './t32c01/t32c01.component';
	
	@NgModule({
	  declarations: [
	    AppComponent,
	    T32c01Component
	  ],
	  imports: [
	    BrowserModule,
	    HttpClientModule, // 引入HttpClient服務所在的模塊
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
2. 組件中聲明HttpClient服務依賴並調用相應方法
	```js
	import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t32c01',
	  templateUrl: './t32c01.component.html',
	  styleUrls: ['./t32c01.component.css']
	})
	export class T32c01Component {
	  httpClient: HttpClient
	
	  // 聲明依賴
	  constructor(httpClient: HttpClient){
	    this.httpClient = httpClient
	  }
	
	  doGet(){
	    let url = 'http://127.0.0.1:8080/test'
	    let header = new HttpHeaders({'Content-Type': 'text/plain'})
	    // 發起異步請求
	    this.httpClient.get(url, {headers: header}).subscribe(response => {
	      console.log('get response')
	      console.log(response) // {"message": "OK!"}
	    })
	  }
	}
	```

### 各異步請求工具比較
| 工具名             | 本質                             | 參考筆記                                   | 優劣勢                                                              |
| ------------------ | -------------------------------- | ------------------------------------------ | ------------------------------------------------------------------- |
| 原生XHR            | `let xhr = new XMLHttpRequest()` |                                            | 瀏覽器支塮的原生技術，基於回調方式處理響應                          |
| `jQuery.ajax()`    | 封裝原生XHR                      | [[🌗jQuery#🏖Ajax]]                          | 比原生XHR使用簡單，基於回調方式處理響應                             |
| Axios              | 封裝原生XHR                      | [[🔲Axios]]                                 | 比原生XHR使用簡單，基於Promise處理響應，可處理排隊、并發、撤銷      |
| Angular HttpClient | 封裝原生XHR                      | [[🅰️Angular_05_服務、依賴注入#HttpClient]] | 比原生XHR使用簡單，基於"觀察者模式"處理響應，以處理排隊、并發、撤銷 |
| Fetch              | W3C提出新技術，有望取代XHR       |                                            | 比XHR更先進，基於Promise，但瀏覽器會有兼容問題                      | 
