# Angular 組件
## 🅰️新建組件(手動)
官方建議組件檔名為`xxx.component.ts`，較為容易視別

### 創建組件 Class
```js
// src\app\T07_MyComponent\t07_01.component.ts
import { Component } from "@angular/core";

// 裝飾器(Decorator)，用於指定class的用途
// 指定此class為Component
@Component({
    selector: 'T07_01', // <T07_01></T07_01>
    // selector: '[T07_01]', // <div myTitle="T07_01"></div>
    // selector: '.T07_01', // <p class="T07_01"></p>
    template: '<h2>this is my Component 01</h2>',
    // templateUrl: './t07_01.component.html', // 外部引入
    styles: ['h2 { background-color: yellow; }'],
    // styleUrls: ['./t07_01.component.css'], // 外部引入
})
// 導出組件
export class T07_01 {

}
```

### 在模塊注冊組件
```js
// src\app\app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
// 引入組件
import { AppComponent } from './app.component';
import { T07_01 } from './T07_MyComponent/t07_01.component';

@NgModule({
  // 聲明組件
  declarations: [
    AppComponent, T07_01, T07_02
  ],
  // 導入其他模塊
  imports: [
    BrowserModule // 瀏覽器模塊，只要Angular用於Web項目，就必須導入此模塊
  ],
  providers: [],
  // 啟動組件
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 使用已注冊組件
```html
<!-- src\app\app.component.html -->
<!-- T07_MyComponent，建立新的組件 -->
<div>
  <T07_01></T07_01>
</div>
```

## 🅰️新建組件(NPM工具)
- 官方文檔：[ng generate component - Angular](https://angular.tw/cli/generate#component-command)
- Visual Studio Code啟動Terminal，Veiw --> Terminal
    
	![Angular_02_組件_01_VScode啟動命令行](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_02_%E7%B5%84%E4%BB%B6_01_VScode%E5%95%9F%E5%8B%95%E5%91%BD%E4%BB%A4%E8%A1%8C.png?raw=true)
- `ng generate component <組件名>`、`ng g component <組件名>`，使用npm(第三方模塊維護工具)建立Angular組件
	```bash
	$ cd /e/PracticeWork/03_JavaStudy/20230815_AngularTry/my-app
	$ ng generate component t10c01
	CREATE src/app/t10c01/t10c01.component.html (21 bytes)
	CREATE src/app/t10c01/t10c01.component.spec.ts (559 bytes)
	CREATE src/app/t10c01/t10c01.component.ts (202 bytes)
	CREATE src/app/t10c01/t10c01.component.css (0 bytes)
	UPDATE src/app/app.module.ts (686 bytes)
	```
- `npx ng generate component <組件名>`、`npx ng g component <組件名>`，使用npx(Node Package Executor，第三方可執行文件執行工具)建立Angular組件，需要在node_modules\.bin\ng.cmd看到指令文件
	```bash
	PS E:\PracticeWork\03_JavaStudy\20230815_AngularTry\my-app> npx ng generate component t09c01
	CREATE src/app/t09c01/t09c01.component.html (21 bytes)
	CREATE src/app/t09c01/t09c01.component.spec.ts (559 bytes)
	CREATE src/app/t09c01/t09c01.component.ts (202 bytes)
	CREATE src/app/t09c01/t09c01.component.css (0 bytes)
	UPDATE src/app/app.module.ts (608 bytes)
	```

## 🅰️生命周期
可參考官方文件：[元件的生命週期 - Angular](https://angular.tw/guide/lifecycle-hooks)

1. `constructor()`，組件對象創建時調用
2. `ngOnChanges()`，組件綁定的屬性值發生改變時調用，需要重新綁定
	```html
	<p [title]="username"></p>
	```
3. `ngOnInit()`，常用，組件初始化完畢調用，對應`OnInit`接口，相當於Vue.js的mounted
4. `ngDoCheck()`，組件檢查到系統對自己有影響時調用
5. `ngAfterContentInit()`，組件內容初始化完成時調用
6. `ngAfterContentChecked()`，組件內容發生變化需要檢查時調用
	```html
	<myc01>
		<!-- 組件內容可能發生變化 -->
		<p>{{ username }}</p>
	</myc01>
	```
7. `ngAfterViewInit()`，組件的視圖初始化完成
8. `ngAfterViewChecked()`，組件的視圖發生變化需要檢查時調用
	```js
	@Component({
	  selector: 'app-t35c01',
	  templateUrl: './t35c01.component.html', // 視圖內容發生改變
	  styleUrls: ['./t35c01.component.css']
	})
	export class T35c01Component {}
	```
9. `ngOnDestroy()`，常用，組件即將從DOM樹上銷毀，適合執行資源釋放
	```html
	<div *ngIf="false">Something</div>
	```

## 🅰️父子組件資料共享
- 可參考官方文件：[在父子指令及元件之間共享資料 - Angular](https://angular.tw/guide/inputs-outputs)
- Angular和Vue.js中父子消息傳運原理一樣

### 父傳子
![Angular_02_組件_02_組件資料父傳子](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_02_%E7%B5%84%E4%BB%B6_02_%E7%B5%84%E4%BB%B6%E8%B3%87%E6%96%99%E7%88%B6%E5%82%B3%E5%AD%90.png?raw=true)

父組件通過"子組件的自定義屬性"向下傳運數據給子組件

1. `@Input()`，子組件中將變量指定為輸入型屬性，表示此屬性可從父組件傳值
	```js
	import { Component, Input } from '@angular/core';
	
	@Component({
	  selector: 'app-t46c03',
	  templateUrl: './t46c03.component.html',
	  styleUrls: ['./t46c03.component.css']
	})
	export class T46c03Component {
	  // 普通屬性不能被父組件傳值
	  // 指定為輸入型屬性，表示此屬性可從父組件傳值
	  @Input()
	  usernameChild2: string = 'Unknow'
	}
	```
2. 父組件正常聲明變量
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t46c01',
	  templateUrl: './t46c01.component.html',
	  styleUrls: ['./t46c01.component.css']
	})
	export class T46c01Component {
	  username: string = 'Mickey'
	}
	```
3. `[子組件變量]="父組件變量"`，子組件標籤使用自定義屬性，將父屬性數據下傳至子組件
	```html
	<!-- src\app\t46c01\t46c01.component.html -->
	<div>
	    <h2>{{ username }}'s Blog</h2>
	    <!-- 使用自定義屬性，將父屬性數據下傳至子組件 -->
	    <app-t46c03 [usernameChild2]="username"></app-t46c03>
	</div>
	```
4. 子組件正常調用數據即可
	```html
	<!-- src\app\t46c03\t46c03.component.html -->
	<div>
	    <h2>{{ usernameChild2 }}'s picture wall</h2>
	</div>
	```

### 子傳父
![Angular_02_組件_02_組件資料子傳父](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/03_%E5%89%8D%E7%AB%AF/%F0%9F%85%B0%EF%B8%8FAngular/images/Angular_02_%E7%B5%84%E4%BB%B6_02_%E7%B5%84%E4%BB%B6%E8%B3%87%E6%96%99%E5%AD%90%E5%82%B3%E7%88%B6.png?raw=true)

子組件通過觸發特定事件，攜帶數據傳遞給父組件

1. 子組件設置
	- 聲明事件發射器`EventEmitter`，並指定`@Output()`，表示此屬性可向父組件傳值
	- 子組件方法中調用`EventEmitter.emit(發送的數據)`，執行發送數據
	```js
	import { Component, EventEmitter, Output } from '@angular/core';
	
	@Component({
	  selector: 'app-t46c02',
	  templateUrl: './t46c02.component.html',
	  styleUrls: ['./t46c02.component.css']
	})
	export class T46c02Component {
	  usernameChild1: string = 'Mickey'
	
	  // 聲明事件發射器
	  // 指定為輸出型屬性，表示此屬性可向父組件傳值
	  @Output()
	  private eventEmitter = new EventEmitter()
	
	  changeUsername(){
	    console.log('T46c02Component.changeUsername() : ' + this.usernameChild1)
	    // 發射數據給父組件
	    this.eventEmitter.emit(this.usernameChild1)
	  }
	}
	```
2. 父組件新增方法，用於接收子組件發送來的數據
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t46c01',
	  templateUrl: './t46c01.component.html',
	  styleUrls: ['./t46c01.component.css']
	})
	export class T46c01Component {
	  username: string = 'Mickey'
	
	  // 處理子組件發送來的數據
	  doChangeUsername(event: any){
	    console.log('T46c01Component.doChangeUsername()')
	    console.log(event) // 子組件發送來的數據
	    this.username = event
	  }
	}
	```
3. `(事件發射器屬性名)="父組件接收數據方法($event)"`，子組件標籤綁定事件，讓父組件監聽子組件事件
	```html
	<div>
	    <h2>{{ username }}'s Blog</h2>
	    <!--
	        通過子組件的事件發射器，調用父組件方法
	        $event，表示子組件要發送的事件數據
	    -->
	    <app-t46c02 (eventEmitter)="doChangeUsername($event)"></app-t46c02>
	</div>
	```
4. 子組件正常使用數據即可
	```html
	<div>
	    <span>change username : </span><br/>
	    <input type="text" [(ngModel)]="usernameChild1"/>
	    <button (click)="changeUsername()">Submit</button>
	</div>
	```

### 子傳父(簡便方式)
直接使用子組件的引用，雖然很方便，但不建議用此方式取得資料

1. `#編號`，組件製定編號
	```html
	<div #component01>
	    <h2>{{ username }}'s Blog</h2>
	    <app-t46c04 #component02></app-t46c04>
	    <button (click)="print()">print()</button>
	</div>
	```
2. `@ViewChild('編號名', {static: true})`，將變量和標籤編號綁定，之後就可以直接使用屬性
	```js
	import { Component, ViewChild } from '@angular/core';
	
	@Component({
	  selector: 'app-t46c01',
	  templateUrl: './t46c01.component.html',
	  styleUrls: ['./t46c01.component.css']
	})
	export class T46c01Component {
	  username: string = 'Mickey'
	
	  // 將變量和標籤編號綁定，並指定編號為靜態的
	  // 靜態組件，表示不會時有時無的組件
	  @ViewChild('component01', {static: true})
	  private parentNode: any;
	  
	  @ViewChild('component02', {static: true})
	  private childNode: any;
	
	  print(){
	    // 可直接調用取得指定標籤編號的組件屬性
	    console.log(this.parentNode)
	    console.log(this.childNode.usernameChild3)
	  }
	}
	```