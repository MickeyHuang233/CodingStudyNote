# Angular路由、導航
- 單頁面應用(Single Page Application，SPA)，整個項目有且只有一個完整的HTML文件，其他的"頁面"都只是`<div>`片段。瀏覽器只需要下載一個HTML頁面，當"頁面"請的異步請求下來時，才會插入到HTML文件中
- 多頁面應用，一個項目前有多個完整的HTML文件，使用超鏈接跳轉，就是銷毀當前的DOM樹，並且同步發出請求、建立另一棵DOM樹
- 可參考官方文件：[Angular 路由 - Angular](https://angular.tw/guide/routing-overview)

## 🅰️單頁應用
1. 建立相應組件
	```bash
	npx ng g c t52c01
	npx ng g c t52c02
	npx ng g c t52c03
	npx ng g c t52c04
	```
2. 定義、注冊路由詞典
	- 路由path不能以`/`開頭或結尾，但中間可包含`/`
	- `{path: '', component: 組件名}`，指定默認首頁地址
	```js
	// src\app\app.module.ts
	// 引入模塊
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { RouterModule } from '@angular/router';
	// 引入組件
	import { AppComponent } from './app.component';
	import { T52c01Component } from './t52c01/t52c01.component';
	import { T52c02Component } from './t52c02/t52c02.component';
	import { T52c03Component } from './t52c03/t52c03.component';
	import { T52c04Component } from './t52c04/t52c04.component';
	import { T52c05Component } from './t52c05/t52c05.component';
	
	// 聲明路由詞典，也就是路由地址、路由組件的對應集合
	let routes = [
	  {path: 'index', component: T52c01Component},
	  {path: 'product-list', component: T52c02Component},
	  {path: 'product-detail', component: T52c03Component},
	  {path: 'user-center', component: T52c04Component},
	  {path: 'test-redirect', redirectTo: 'index'}, // 重定向
	  {path: '**', component: T52c05Component}, // **，表示匹配任意地址
	]
	
	@NgModule({
	  declarations: [
	    AppComponent,
	    T52c01Component,
	    T52c02Component,
	    T52c03Component,
	    T52c04Component,
	    T52c05Component
	  ],
	  imports: [
	    BrowserModule,
	    // 導入路由組件，并注冊路由詞典，應用於根模塊中
	    RouterModule.forRoot(routes)
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
3. 創建路由組件佳載點，也就是路由出口
	```html
	<!-- 路由，單頁應用 -->
	<h2>Test Router</h2>
	<!-- 路由組件佳載點 -->
	<router-outlet></router-outlet>
	```
4. 訪問測試
	- http://localhost:4200/index
	- http://localhost:4200/product-list
	- http://localhost:4200/product-detail
	- http://localhost:4200/user-center

## 🅰️路由跳轉
從一個路由地址跳轉到另一個路由地址

### 使用模版實現
```html
<!-- 使用傳統的當鏈接可以跳轉，但是屬於DOM樹完全重建 -->
<a href="product-list">enter product list - a href</a><br/>
<!-- 
    使用routerLink跳轉，可以實現單頁跳轉
    跳轉地址以/開頭，防止以相對路徑進行跳轉
    並且可以用於任意標籤
-->
<a routerLink="/product-list">enter product list - a routerLink</a>
<div routerLink="/product-list">enter product list - div</div>
```

### 使用TS腳本實現
- Router類是RouterModule提供的一個服務類，聲明依賴即可使用
	- `this.$route`，當前路由
	- `this.$router`，路由器
1. 注入路由器服務後，使用`this.router.navigateByUrl('跳轉路徑')`進行跳轉
	```js
	import { Component } from '@angular/core';
	import { Router } from '@angular/router';
	
	@Component({
	  selector: 'app-t52c01',
	  templateUrl: './t52c01.component.html',
	  styleUrls: ['./t52c01.component.css']
	})
	export class T52c01Component {
	  // 注入路由器服務
	  constructor(private router: Router){}
	
	  // 跳轉至product-list頁面
	  enterProductList(){
	    this.router.navigateByUrl('product-list')
	  }
	}
	```
2. html調用跳轉方法
	```html
	<!-- 調用跳轉方法 -->
	<button (click)="enterProductList()">enter product list - TS</button>
	```

## 🅰️路由參數
1. 路由詞典定義路由地址時，`:參數名`可定義可變參數
	```js
	// src\app\app.module.ts
	// 引入模塊
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { RouterModule } from '@angular/router';
	// 引入組件
	import { AppComponent } from './app.component';
	import { T52c01Component } from './t52c01/t52c01.component';
	import { T52c02Component } from './t52c02/t52c02.component';
	import { T52c03Component } from './t52c03/t52c03.component';
	
	// 聲明路由詞典，也就是路由地址、路由組件的對應集合
	let routes = [
	  {path: 'index', component: T52c01Component},
	  {path: 'product-list', component: T52c02Component},
	  {path: 'product-detail/:pid', component: T52c03Component}, // :參數名
	]
	
	@NgModule({
	  declarations: [
	    AppComponent,
	    T52c01Component,
	    T52c02Component,
	    T52c03Component,
	  ],
	  imports: [
	    BrowserModule,
	    // 導入路由組件，并注冊路由詞典，應用於根模塊中
	    RouterModule.forRoot(routes)
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
2. 注入`ActivatedRoute`依賴，使用觀察者模式讀取路由參數
	```js
	import { Component } from '@angular/core';
	import { ActivatedRoute } from '@angular/router';
	
	@Component({
	  selector: 'app-t52c03',
	  templateUrl: './t52c03.component.html',
	  styleUrls: ['./t52c03.component.css']
	})
	export class T52c03Component{
	  pid: number = 0
	
	  // 注入依賴，讀取參數需要"當前的路由"服務對象
	  constructor(private route: ActivatedRoute){}
	  
	  // 組件初始化完成調用
	  ngOnInit(){
	    // 使用觀察者模式讀取路由參數
	    this.route.params.subscribe(data => {
	      console.log('get subscribe route params')
	      console.log(data)
	      // 取得參數
	      this.pid = data['pid']
	    })
	  }
	}
	```
3. 在模版中可正常使用參數，因為已經賦值至組件屬性中
	```html
	<p>t52c03 works!</p>
	<p>pid : {{ pid }}</p>
	```
4. 路由跳轉url中，就可以為參數提供具體的參數
	- http://localhost:4200/product-detail/1231

## 🅰️路由嵌套
可參考：[巢狀(Nesting)路由 - Angular](https://angular.tw/guide/router#nesting-routes)
![Angular_06_路由、導航_01_路由嵌套](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_06_%E8%B7%AF%E7%94%B1%E3%80%81%E5%B0%8E%E8%88%AA_01_%E8%B7%AF%E7%94%B1%E5%B5%8C%E5%A5%97.png?raw=true)

1. 建立相應組件
2. 路由詞典可嵌套`children`屬性
	```js
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { RouterModule } from '@angular/router';
	// 引入組件
	// 省略
	
	// 聲明路由詞典，也就是路由地址、路由組件的對應集合
	let routes = [
	  {path: 'index', component: T52c01Component},
	  {path: 'test-redirect', redirectTo: '/index'},
	  {path: 'product-list', component: T52c02Component},
	  {path: 'product-detail/:pid', component: T52c03Component},
	  {path: 'user-center', component: T52c04Component},
	  {path: 'about-me', component: T58c01Component,
	    children: [ // 路由嵌套
	      {path: 'info', component: T58c02Component},
	      {path: 'change-password', component: T58c03Component},
	      {path: 'securty', component: T58c04Component},
	      {path: '**', component: T58c05Component},
	    ]
	  },
	  {path: '**', component: T52c05Component},
	]
	
	@NgModule({
	  declarations: [
	    // 聲明組件，省格
	  ],
	  imports: [
	    BrowserModule,
	    FormsModule,
	    HttpClientModule,
	    // 導入路由組件，并注冊路由詞典，應用於根模塊中
	    RouterModule.forRoot(routes)
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
3. 創建路由組件佳載點，也就是路由出口
	```html
	<!-- src\app\t58c01\t58c01.component.html -->
	<div>
		<!-- 路由組件佳載點 -->
		<router-outlet></router-outlet>
	</div>
	```
4. 路由跳轉url需要指定上一層，否則會指向錯誤路徑
	```html
	<!-- src\app\t58c01\t58c01.component.html -->
	<ul>
		<li><a routerLink="/about-me/info">info</a></li>
		<li><a routerLink="/about-me/change-password">change password</a></li>
		<li><a routerLink="/about-me/securty">securty</a></li>
	</ul>
	```
5. 測試
	- http://localhost:4200/about-me/info
	- http://localhost:4200/about-me/change-password
	- http://localhost:4200/about-me/securty

## 🅰️路由守衛
- 路由守衛(Guard)，訪問路由組件前的檢查功能，可參考：[路由守衛 - Angular](https://angular.tw/guide/router-tutorial-toh#milestone-5-route-guards)
- 路由守衛檔命名建議為`xxx.guard.ts`，以便識別

1. `[npx] ng generate guard 資料夾名/路由守衛名`，新增路由守衛
	```bash
	PS D:\01_MickeyHuang\02_codeProject\20230815_AngularTry\my-app> npx ng g g t60g01/t60g01
	? Which type of guard would you like to create? CanActivate
	CREATE src/app/t60g01/t60g01.guard.spec.ts (469 bytes)
	CREATE src/app/t60g01/t60g01.guard.ts (130 bytes)
	```
2. 編寫路由守衛放行邏輯
	```js
	// src\app\t60g01\t60g01.guard.ts
	import { inject } from '@angular/core';
	import { CanActivateFn, Router } from '@angular/router';
	
	// 路由守衛相當於是特殊的服務
	export const t60g01Guard: CanActivateFn = (privateroute, state) => {
	  let isPass: boolean = false
	
	  // TODO，判斷當前請求是否有權限
	  console.log('t60g01Guard.CanActivateFn()')
	  // true，表示可以繼續後續組件
	  // false，表示組止後續組件訪問
	  if(!isPass){
	    // 若不符權限，可以執行頁面跳轉
	    inject(Router).navigateByUrl('/index')
	  }
	  return true;
	};
	```
3. 路由詞典設置`canActivate`屬性，可指定多個路由守衛，要所有都依次通過之後才可以訪問到組件
	```js
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { RouterModule } from '@angular/router';
	// 省略引入組件
	// 引入路由守衛
	import { t60g01Guard } from './t60g01/t60g01.guard';
	
	// 聲明路由詞典，也就是路由地址、路由組件的對應集合
	let routes = [
	  {path: 'index', component: T52c01Component},
	  {path: 'test-redirect', redirectTo: '/index'},
	  {path: 'product-list', component: T52c02Component},
	  {path: 'product-detail/:pid', component: T52c03Component},
	  {path: 'user-center', component: T52c04Component},
	  {path: 'about-me', component: T58c01Component,
	    children: [
	      {path: 'info', component: T58c02Component},
	      {path: 'change-password', component: T58c03Component},
	      {path: 'securty', component: T58c04Component},
	      {path: '**', component: T58c05Component},
	    ],
		// 可指定多個路由守衛，要所有都依次通過之後才可以訪問到組件
	    canActivate: [t60g01Guard],
	  },
	  {path: '**', component: T52c05Component},
	]
	
	@NgModule({
	  declarations: [
	    AppComponent,
	    T52c01Component,
	    T52c02Component,
	    T52c03Component,
	    T52c04Component,
	    T52c05Component,
	    T58c01Component,
	    T58c02Component,
	    T58c03Component,
	    T58c04Component,
	    T58c05Component
	  ],
	  imports: [
	    BrowserModule,
	    // 導入路由組件，并注冊路由詞典，應用於根模塊中
	    RouterModule.forRoot(routes)
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
