# Angular數據、指令綁定
## 🅰️HTML綁定 {{ }}
1. 組件 Class定義變量
	```js
	// src\app\t10c01\t10c01.component.ts
	@Component({
	  selector: 'app-t10c01',
	  templateUrl: './t10c01.component.html',
	  styleUrls: ['./t10c01.component.css']
	})
	export class T10c01Component {
	  username = 'Mickey';
	  age = 233;
	}
	```
2. `{{ Angular表達式 }}`，數據調用
	```html
	<!-- src\app\t10c01\t10c01.component.html -->
	<p>username = {{ username }}</p>
	<p>age = {{ age }}</p>
	```

### 運算符
除了支持數據的調用還支持運算符，可參考：[[🟨JavaScript_02_運算符]]
- 算數運算
	```html
	<!-- 算數運算 -->
	<p>age * 2 = {{ age * 2 }}</p>
	```
- 比較運算
	```html
	<!-- 比較運算 -->
	<p>age is bigger than 200 ? {{ age >= 200 }}</p>
	<p>age is bigger than 200 ? {{ age >= 200 ? 'YES' : 'NO'}}</p>
	```
- 邏輯運算
	```html
	<!-- 邏輯運算 -->
	<p>age is bigger than 200 and small than 220 ? {{ age >= 200 && age <= 220 }}</p>
	```
- 三目運算

### 調用函數
- 調用函數
	```html
	<!-- 調用函數 -->
	<p>username lenght : {{ username.length }}</p>
	<p>username uppercase : {{ username.toUpperCase() }}</p>
	<p>username second sign : {{ username[1] }}</p>
	```
- 創建對象 --> ==無法，會報錯==
	```html
	<!-- 創建對象 -->
	<!-- src/app/t10c01/t10c01.component.html:22:15 - error TS2339: Property 'new' does not exist on type 'T10c01Component'. -->
	<!-- <p>{{ new String() }}</p> -->
	```
- JSON序列化 --> ==無法，會報錯==
	```html
	<!-- JSON序列化 -->
	<!-- src/app/t10c01/t10c01.component.html:26:29 - error TS2339: Property 'JSON' does not exist on type 'T10c01Component'. -->
	<!-- <p>JSON String : {{ JSON.stringify({}) }}</p> -->
	```

## 🅰️屬性綁定 \[ \]
相當於是Vue的`v-bind:屬性名`，或省略為`:屬性名`，可參考：[[🌲Vue3_03_模版基礎語法#🌲綁定屬性]]

- 使用HTML綁定實現屬性綁定
	```html
	<p title="{{ username }}">
		Lorem, ipsum dolor sit amet consectetur adipisicing elit. Minus nihil at illum sit. Optio quas
		nisi ad, ipsa
		repellendus asperiores minus alias totam quibusdam sint sed consequatur, voluptates deleniti unde.
	</p>
	```
- `[屬性名]="變量名"`，如果需要使用字串常量，則要用`'字串常量'`再進行處理，如：`"'字串常量' + 變量名"`
	```html
	<p [title]="'creater : ' + username">
	    Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet dolorum distinctio, porro, fugit fuga
	    minima officiis non at velit impedit repudiandae ipsa quam repellendus culpa tempore rem, officia aliquam
	    maxime?
	</p>
	```

## 🅰️事件綁定 ( )
相當於是Vue的`v-on:事件名`，或省略為`@事件名`，可參考：[[🌲Vue3_03_模版基礎語法#🌲事件監聽]]

1. 組件Class定義方法函數
```js
// src\app\t10c01\t10c01.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-t10c01',
  templateUrl: './t10c01.component.html',
  styleUrls: ['./t10c01.component.css']
})
export class T10c01Component {
  username = 'Mickey';
  age = 233;

  // 方法函數
  printUsername() {
    console.log(this.username);
  }

  printLog(log: String) {
    console.log(log);
  }
}
```
2. `(事件名)="方法函數名()"`，事件綁定，注意函數名後必須有`()`
```html
<!-- src\app\t10c01\t10c01.component.html -->
<!-- 事件綁定 -->
<button (click)="printUsername()">output username</button>
<button (click)="printLog('my log')">output Log</button>
```

## 🅰️指令綁定 \*、\[ \]
- [所有指令 - Angular API參考手冊](https://angular.tw/api?type=directive)
- 分類
	- 結構型指令，會影響DOM樹結構，使用`*指令`，如：`*ngFor`、`*ngIf`
	- 屬性型指令，不會影響DOM樹結構，只影響元素外觀、行為，使用`[指令]`，如：`[ngStyle]`、`[ngClass]`、`[ngSwitch]`
	- 組件指令，繼承自`Directive`的組件，組件就是特殊的指令

### 循環綁定
`ngFor`，相當於是Vue的`v-for`，可參考：[[🌲Vue3_03_模版基礎語法#🌲循環遍歷]]、[NgFor - Angular API參考手冊](https://angular.tw/api/common/NgFor)

1. 組件Class定義變量
```js
import { Component } from '@angular/core';

@Component({
  selector: 'app-t10c01',
  templateUrl: './t10c01.component.html',
  styleUrls: ['./t10c01.component.css']
})
export class T10c01Component {
  employees = ['Anny', 'Molly', 'Tai', 'Will'];
}
```
2. `*ngFor`，循環綁定
```html
<ul>
    <li *ngFor="let employee of employees">{{employee}}</li>
</ul>
<ul>
    <!-- 自定義變量取得區域變數，如：下標 -->
    <li *ngFor="let employee of employees; let i = index">{{i}} = {{employee}}</li>
    <!-- 也可使用as，實現相同效果 -->
    <li *ngFor="let employee of employees; index as i">{{i}} = {{employee}}</li>
</ul>
```

### 選擇綁定
`ngIf`、`ngIfElse`、`ngIfThen`，相當於是Vue的`v-if="表達式"`，可參考：[[🌲Vue3_03_模版基礎語法#🌲條件判斷]]、[NgIf - Angular API參考手冊](https://angular.tw/api/common/NgIf)、[NgSwitch - Angular API參考手冊](https://angular.tw/api/common/NgSwitch)

- `*ngIf="布爾表達式"`
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t10c01',
	  templateUrl: './t10c01.component.html',
	  styleUrls: ['./t10c01.component.css']
	})
	export class T10c01Component {
	  age = 233;
	}
	```
	
	```html
	<!-- 選擇綁定 -->
	<!-- 如果為false，元素就會從DOM樹中刪除 -->
	<div *ngIf="age >= 200">You can't watch this context</div>
	```
- `*ngIf="布爾表達式"; else 編號`
	```html
	<!-- if else -->
	<div *ngIf="age >= 18; else myElse">You are adult</div>
	<!-- ng-template，#編號，為空白的標籤，用於Angular調用 -->
	<ng-template #myElse>
		<div>You are baby</div>
	</ng-template>
	```

### 樣式綁定
- `[ngStyle]="CSS對象"`，綁定必須為對象，對象的屬性名就是CSS樣式名，[NgStyle - Angular API參考手冊](https://angular.tw/api/common/NgStyle)
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t10c01',
	  templateUrl: './t10c01.component.html',
	  styleUrls: ['./t10c01.component.css']
	})
	export class T10c01Component {
	  myDivStyle = {
	    backgroundColor: '#555', // 必須改為駝峰命名
	    color: '#fff',
	    'font-size': '20px'
	  };
	}
	```

	```html
	<div [ngStyle]="myDivStyle" (click)="clickDiv()">try ngClass</div>
	```
- `[ngClass]="對象"`，，綁定必須為對象，對象的屬性名就是CSS Class名，屬性值為boolean，true表示使用此class，否則不使用，[NgClass - Angular API參考手冊](https://angular.tw/api/common/NgClass)
	```css
	.divClass01{
	    background-color: black;
	    color: white;
	    font-size: 20px;
	}
	.divClass02{
	    background-color: white;
	    color: black;
	    font-size: 20px;
	}
	```
	
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t10c01',
	  templateUrl: './t10c01.component.html',
	  styleUrls: ['./t10c01.component.css']
	})
	export class T10c01Component {
	  myDivClass = {
	    divClass01: true,
	    divClass02: false,
	  }
	
	  clickDiv2(){
	    if (this.myDivClass.divClass01) {
	      this.myDivClass.divClass01 = false
	      this.myDivClass.divClass02 = true
	    } else {
	      this.myDivClass.divClass01 = true
	      this.myDivClass.divClass02 = false
	    }
	  }
	}
	```
	
	```html
	<div [ngClass]="myDivClass" (click)="clickDiv2()">try ngClass</div>
	```

### 自定義指令
1. `[npx] ng g directive <指令名>`，使用工具建立指令
	```bash
	$ ng g directive needHighLine
	CREATE src/app/need-high-line.directive.spec.ts (250 bytes)
	CREATE src/app/need-high-line.directive.ts (153 bytes)
	UPDATE src/app/app.module.ts (978 bytes)
	```
2. 建立成功後，可在app.module.ts看到引入自定義指令
	```js
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	// 引入組件
	import { NeedHighLineDirective } from './need-high-line.directive';
	
	@NgModule({
	  declarations: [
	    AppComponent,
	    NeedHighLineDirective, // 聲明自定義指令
	  ],
	  imports: [
	    BrowserModule,
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
3. 建立成功後，會建立自定義指令的ts檔
	```js
	// src\app\need-high-line.directive.ts
	import { Directive, ElementRef } from '@angular/core';
	
	// 組件繼承指令，也就是說組件是一種特殊的指令
	@Directive({
	  selector: '[appNeedHighLine]'
	})
	export class NeedHighLineDirective {
	  // 構造方法，編寫自定義處理
	  constructor(element:ElementRef) {
	    console.log("NeedHighLineDirective constructor()")
	    console.log(element)
	    element.nativeElement.style.background = '#fcc'
	    element.nativeElement.style.fontSize = '20px'
	    element.nativeElement.style.color = '#833'
	  }
	}
	```
4. 使用自定義指令
	```html
	<!-- 自定義指令 -->
	<div id="myDirective01" appNeedHighLine>Lorem ipsum dolor sit amet consectetur adipisicing elit. Unde laudantium voluptatem dicta consectetur quasi, mollitia dignissimos non, natus pariatur harum debitis quod maiores recusandae id enim ipsum temporibus eum. Sed?</div>
	<div id="myDirective02" appNeedHighLine>Lorem ipsum dolor sit amet consectetur adipisicing elit. Unde laudantium voluptatem dicta consectetur quasi, mollitia dignissimos non, natus pariatur harum debitis quod maiores recusandae id enim ipsum temporibus eum. Sed?</div>
	```

## 🅰️雙向數據綁定 \[( )\]
- `[(ngModel)]`，相當於是Vue的`v-model="變量名"`，可參考：[[🌲Vue3_03_模版基礎語法#🌲雙向綁定]]、[NgModel - Angular API參考手冊](https://angular.tw/api/forms/NgModel)
	- 模型(Model)變則視圖(View)變，用`[]`綁定
	- 視圖(View)變則模型(Model)變，用`()`綁定
- 使用前必須引入`FormsModule`模塊
	```js
	// 引入模塊
	import { NgModule } from '@angular/core';
	import { BrowserModule } from '@angular/platform-browser';
	import { FormsModule } from '@angular/forms';
	// 引入組件
	import { AppComponent } from './app.component';
	import { T10c01Component } from './t10c01/t10c01.component';
	
	// 指定此class為Module
	@NgModule({
	  // 聲明組件
	  declarations: [
	    AppComponent,
	    T10c01Component
	  ],
	  // 導入其他模塊
	  imports: [
	    BrowserModule, // 瀏覽器模塊，只要Angular用於Web項目，就必須導入此模塊
	    FormsModule,
	  ],
	  providers: [],
	  bootstrap: [AppComponent]
	})
	export class AppModule { }
	```
- 需要定義變量
	```js
	import { Component } from '@angular/core';
	
	@Component({
	  selector: 'app-t10c01',
	  templateUrl: './t10c01.component.html',
	  styleUrls: ['./t10c01.component.css']
	})
	export class T10c01Component {
	  username = 'Mickey';
	  
	  modelChange(){
	    console.log('model change : ' + this.username)
	  }
	}
	```
- `[(ngModel)]="變量名"`，實現數據雙向綁定
	`(ngModelChange)="方法名()"`，監控雙向綁定的數據是否有變更，有則調用指定方法
	```html
	<!-- 雙向數據綁定 -->
	<!-- (ngModelChange)，用於監控雙向綁定的數據是否有變更，有則調用指定方法 -->
	<input [(ngModel)]="username" placeholder="input please" (ngModelChange)="modelChange()"><br/>
	<p>Input user name is : {{ username }}</p>
	```


