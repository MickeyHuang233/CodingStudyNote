###### tags: `📆2022年` `💻編程/🖼前端` `🗂Overview`

# ECMAScirpt6
==以下只記錄ES5和ES6差異較大且常用的語法、概念，主要用於學習往後前端框架的預備知識。==

- European Computer Manufacturers Association(ECMA，歐洲計算機制造聯合會)，是評估、開發、認可電信和計算機標準的組織
- ECMA是標準，JavaScript是實現。參考：[[🏖JavaScript_00_Overview#🏖簡介]]
	- 目的是讓所有前端腳本都實現ECMA，但目前只有JavaScript實現ECMA標準，因此ECMAScript ≈ JavaScript
- ECMAScript，ECMAScript 2015，簡稱ECMA或ES(ES6)
	- 高級瀏覽器支持ES6
	- 低級濟覽器主要支持ES5(ES3.1)
- ECMA官網：[https://www.ecma-international.org/]()
- [ES6兼容表](https://kangax.github.io/compat-table/es6/)

## 🏖變量聲明
- 使用`var`聲明變量的缺點：
	1. `var`可重複聲明
	2. `var`無法限制修改
	3. `var`無塊級作用域(如：`{}`、`if`、`for`…)，只有在`function()`中有作用域

### let
- `let`不可重複聲明
	```javascript
	let test01 = 10
	// Uncaught SyntaxError: Identifier 'test01' has already been declared
	let test01 = 20
	```
- `let`有作用域
	```javascript
	// Uncaught ReferenceError: test01 is not defined
	{
		let test01 = 10
	}
	console.log(test01)
	```
- `let`不存在變量提升，變量必須先使用後聲明
	```javascript
	// Uncaught ReferenceError: Cannot access 'test01' before initialization
	console.log(test01)
	let TESE_01 = 10
	```

### const
`const`用於聲明一個只讀的常量，像Java中的`final`，並且一旦聲明必須初始化

- 不可重複聲明
- 有作用域
- 不存在變量提升，變量必須先使用後聲明
- `const`聲明的變量為只讀的常量；如果聲明的是複合的常量，複合常量的值可以修改，但無法再賦值其他複合常量
	```javascript
	// Uncaught TypeError: Assignment to constant variable.
	const TESE_01 = 10
	TESE_01 = 20
	// 如果聲明的是複合的常量，複合常量的值可以修改，但無法再賦值其他複合常量
	// 因為const聲明的變量只是不能修改棧內存的地址，但可以修改引用地址中的內容
	const TEST_02 = {name: "mickey", id: "123"}
	TEST_02.id = 233
	console.log(TEST_02)
	```
- 一旦聲明必須初始化
	```javascript
	// Uncaught SyntaxError: Missing initializer in const declaration
	const test01
	```

## 🏖=> 箭頭函數
### 基本用法
將`function`簡化為`=>`，類似於java中的Lambda表達式，參考：[[💡Java8_01_Lambda#💡基礎語法]]

```javascript
// ES6前寫法
const FUN_01 = function(num){
    return num * num
}
console.log(FUN_01(10))

// ES6簡化1，方法中有多行代碼
const FUN_02 = (num) => {
    return num * num
}
console.log(FUN_02(20))

// ES6簡化2，方法中僅一行代碼，可省略{}和return
// 如果只有一個參數，可省略()
const FUN_03 = num => num * num
console.log(FUN_03(30))
const FUN_04 = (num1, num2) => num1 * num2
console.log(FUN_04(10, 20))

// ES6簡化3，方法無輸入參數
const FUN_05 = () => 10
console.log(FUN_05())

// 直接返回複合常量，可以用()包起，避免歧義造成語法錯誤
const FUN_06 = id => ({name: 'mickey', id: id})
console.log(FUN_06('233'))
```

### this指向問題
- 無法用`=>`作為構造器方法
	```javascript
	// ES6前，function構造器方法
	function constructer01() {}
	const OBJ_01 = new constructer01()
	console.log(OBJ_01)
	
	// ES6，無法用=>作為構造器方法
	// Uncaught TypeError: constructer02 is not a constructor
	constructer02 = () =>{}
	const OBJ_02 = new constructer02()
	console.log(OBJ_02)
	```
- `=>`建立的函數，不是調用對象本身，可參考：[[🏖JavaScript_05_函數#🏖this]]
	```javascript
	// function函數中，this代表調用對象本身
	const OBJ_03 = {
	    fun: function(){
	        console.log(this)
	    },
	}
	OBJ_03.fun(); // {fun: ƒ}
	
	// 沒有調用者默認為window
	const OBJ_04 = function(){
	    console.log(this)
	}
	// 相當於window.OBJ_04()
	OBJ_04(); // Window {window: Window, self: Window, document: document, name: '', location: Location, …}
	
	// =>函數中，this代表window對象，和調用對象無關
	const OBJ_05 = {
	    fun: () => {
	        console.log(this)
	    },
	}
	OBJ_05.fun(); // Window {window: Window, self: Window, document: document, name: '', location: Location, …}
	```
- 箭頭函數的`this`為定義時所在的對象，默認使用父級的`this`，也就是說箭頭函數的`this`是繼承來的，沒有自己的`this`
	```html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta http-equiv="X-UA-Compatible" content="IE=edge">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <title>Document</title>
	    <style>
	        #box{
	            width: 100px;
	            height: 100px;
	            background-color: aqua;
	        }
	        #box.bgcolor{
	            background-color: cadetblue;
	        }
	    </style>
	</head>
	<body>
	    <div id="box"></div>
	    <script>
	        // 箭頭函數的this為定義時所在的對象
	        const myBox = document.getElementById('box')
	        myBox.onclick = function(){
	            setTimeout(() => {
	                console.log(this)
	                this.className = 'bgcolor'
	            }, 3000);
	        }
	    </script>
	</body>
	</html>
	```

## 🏖數組新增的高級方法
- ES5數組常用方法，可參考：[[🏖JavaScript_08_常用對象#🏖Array 數組]]
- 使用方法類似於Java 8中的StreamAPI，可參考：[[💡Java8_02_StreamAPI]]
- 以下只列出常用方法

```javascript
let array01 = [5, 20, 30, 60, 13, 50, 6, 120, 8]
// filter，過濾器，過濾返回false的參數
let array02 = array01.filter(num => num >= 10)
console.log(array02)
// map，映射，返回處理後的參數
let array03 = array02.map(num => num * 0.5)
console.log(array03)
// reduce，匯總，將流中元素反複結合，得到一個值
let total = array03.reduce(
    (preNum, nowNum) => preNum + nowNum,
    0 // 初始值
)
console.log(total)

// 合并使用
let newTotal = array01
	.filter(num => num >= 10)
	.map(num => num * 0.5)
	.reduce(
		(preNum, nowNum) => preNum + nowNum,
		0 // 初始值
	)
console.log(newTotal)
```

## 🏖新增Set、Map數據結構
- ES6中新增Set和Map兩種數據結構，相關用法可參考：[ES6 Map 与 Set - 菜鳥教程](https://www.runoob.com/w3cnote/es6-map-set.html)
	- Set為不可重覆的List，和Java的概念不一樣，可參考[Set - MDN Web Docs - Mozilla](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Set)
	- Map為鍵值對，其中鍵不可重覆，概念類似於Java中的Map
		- `size`屬性，Map總成員數
		- `set(key, value)`，添加鍵值對
		- `get(key)`，取得鍵值對
		- `has(key)`，Map中是否包含指定鍵
		- `delete(key)`，刪除鍵值對
		- `clear()`，清空所有鍵值對
		- `keys()`，返回所有鍵名遍歷器
		- `values()`，返回所有鍵的值遍歷器
		- `entries()`，返回Map中所有成員遍歷器
		- `forEach()`，遍歷Map所有成員

## 🏖字符串新增功能
- 新方法
	- `startWith(str)`，字符串是否以str開頭
	- `endWith(str)`，字符串是否以str結尾
- 模版字符串，Template String
	```javascript
	// 模版字符串
	let value = 'Hello My World' // 變量聲明必須在模版字符串前面
	let templateStr01 = `
	    <h1>Hello World</h1>
	    <b>支持換行<br>
	    <p>支持變量：${value}</p>
	`
	console.log(templateStr01)
	```

## 🏖解構賦值
解構賦值可以方便數組或物件中的資料，可參考：[解構賦值 - mozilla.org](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

- 數組中成員數左右必須一致
	```javascript
	let list01 = ['a', 'b', 'c']
	// 原始方式取得陣列中的值
	// let str01 = list01[0]
	// let str02 = list01[1]
	// let str03 = list01[2]
	// ES6新方式
	let [str01, str02, str03] = list01
	console.log(str01)
	console.log(str02)
	console.log(str03)
	```
- 物件的屬性名和變量名必須一致
	```javascript
	let obj01 = {
	    name: 'Mickey',
	    id: 123,
	    gender: 'male',
	}
	// 原始方式取得物件中的值
	// let name = obj01.name
	// let id = obj01.id
	// let gender = obj01.gender
	// ES6新方式
	let {name, id, gender} = obj01
	console.log(name)
	console.log(id)
	console.log(gender)
	```
- 聲明時必須賦值
	```javascript
	let list01 = ['a', 'b', 'c']
	// 會報錯
	// let [str01, str02, str03]
	// [str01, str02, str03] = list01
	```

### ... 三點運算符
詳細用法可參考：[Spread Operator & Rest Operator(展開與其餘運算符)](https://ithelp.ithome.com.tw/articles/10185496)

- 展開運算符，用於展開數組
	```javascript
	// 展開運算符
	let arr01 = [11, 12, 13]
	let arr02 = [21, 22, 23]
	console.log([...arr01, 7, 8, 9, ...arr02])
	```
- 其餘運算符
	```javascript
	// 後面多余的參數都會放入數組中
	function test02(a, b, ...c){
	    console.log(c) // (5) [53, 54, 55, 65, 57]
	}
	test02(51, 52, 53, 54, 55, 65, 57)
	```

## 🏖Class類的概念
更接近於面向對象的用法
- `constructor`，表示構造方法
- `this`，表示實例對象
- `extends`，表示繼承
- `super`，表示父類構造函數，用於建立父類`this`對象

```javascript
// ES5前，類的使用方式
function Person01(name, age, gender){
    this.name = name
    this.age = age
    this.gender = gender
    this.printName = function(){
        console.log(this.name)
    }
}
let person01 = new Person01('Mickey', '123', 'male')
person01.printName()

// ES6的方式
class Person02{
    // 構造方法，有參
    constructor(name, age, gender){
        this.name = name
        this.age = age
        this.gender = gender
    }
    // 構造方法，無參
    // constructor(){
    //     this.name = 'unknow'
    //     this.age = 'unknow'
    //     this.gender = 'unknow'
    // }
    // 類方法
    printName(){
        console.log(this.name)
    }
}
// 類繼承
class Employee extends Person02{
    constructor(name, age, gender, department){
        super(name, age, gender)
        this.department = department
    }
    printDepartment(){
        console.log(this.department)
    }
}
let person02 = new Person02('Mickey', '123', 'female')
person02.printName()
let person03 = new Employee('Jack', '223', 'male', 'PG')
person03.printName()
person03.printDepartment()
```

## 🏖JSON新應用
- 屬性名和值名稱一樣可以省略
	```javascript
	// 屬性名和值名稱一樣可以省略
	let name = 'Mickey'
	let mickeyAge = 123
	let obj01 = {name: name, age: mickeyAge, gender: 'male'}
	let obj02 = {
	    name,
	    age: mickeyAge, // 屬性名和值的變量名稱不一樣就不能省洛
	    gender: 'male',
	    printName(){ // 定義函數，可省略function關鍵字
	        console.log(name)
	    },
	}
	obj02.printName()
	```
- `JSON.stringify()`，串行化
	```javascript
	let obj03 = {name: 'jack', age: 223, gender: 'male'}
	// 串行化
	let jsonStr = JSON.stringify(obj03)
	console.log(jsonStr) // {"name":"jack","age":223,"gender":"male"}
	```
- `JSON.parse()`，反串行化
	```javascript
	// 反串行化
	let obj04 = JSON.parse(jsonStr)
	console.log(obj04.name)
	```

## 🏖Module模塊化編程
- 模塊化優點
	1. 避免變量、函數重名導致的錯誤
	2. 避免引入時的層層依賴
	3. 可提升執行效率

### export
`export`，用於規定模塊對外接口

1. 一次指定所有的對外接口
	```javascript
	// T037_Moduel01.js
	let name = 'Mickey'
	let id = 123
	
	function printName(){
		console.log(name)
	}
	
	function add(a, b){
		console.log(a + b)
	}
	
	// 指定暴露的變量、函數
	export {name, id, printName, add}
	```
2. 在變量、函數前指定此接口是否對外
	```javascript
	// T037_Moduel02.js
	// 指定暴露的變量、函數
	export let name = 'Jack'
	let id = 223
	
	export function printName(){
		console.log(name)
	}
	
	export function printId(){
		console.log(id)
	}
	
	export class Employee{
		constructor(){}
	
		doSomething(){
			console.log("do something...")
		}
	}
	```
3. `export default`，由導入方決定函數名，但一個模塊只能使用一次
	```javascript
	// T037_Moduel03.js
	
	let name = 'Marry'
	let id = 323
	
	// 由導入方決定函數名，但一個模塊只能有一個export default
	export default function(args){
		console.log(args)
	}
	```

### import
`import`，用於引入其他模塊的功能

1. 使用模塊定義的變量名、函數名
	```html
	<!-- 在html引入模塊要用type="module" -->
	<script type="module">
		// 使用模塊定義的變量名、函數名
		import {name, id, printName} from './T037_Moduel01.js'
		console.log(name)
		printName()
	</script>
	```
2. 如果引入重名的變量或函數，`as`指定別名
	```html
	<script type="module">
		// 如果引入重名的變量或函數，指定別名
		import {name as jackName, printName as printJack, printId, Employee} from './T037_Moduel02.js'
		console.log(jackName) // 使用別名調用變量
		printJack() // 使用別名調用函數
		let employee = new Employee()
		employee.doSomething()
	</script>
	```
3. 導入`export default`，指定名稱
	```html
	<script type="module">
		// 導入export default，指定名稱
		import printArgs from './T037_Moduel03.js'
		printArgs([1, 2, 3, 4])
	</script>
	```
4. 將模塊所有內容導入
	```html
	<script type="module">
		// 將模塊所有內容導入
		import * as test from './T037_Moduel01.js'
		console.log(test.name)
		test.printName()
	</script>
	```

## 🏖Promise
- Promise主要用於異步計算，它可以將異步操作隊列化，並按期望的順序執行

### ES6前異步用法
ES6前處理多層異步請求，很容易陷入回調地獄，導致程式碼層次過多

```javascript
// ES5，回調地獄，之一
$.get('url01', data1 => {
	// do something...
	$.get('url02', data2 => {
		// do something...
		$.get('url03', data3 =>{
			// do something...
		})
	})
})

// ES5，回調地獄，之二
setTimeout(() => {
	console.log('first step')
	setTimeout(() => {
		console.log('second step')
		setTimeout(() => {
			console.log('third step')
		}, 1000)
	}, 1000)
}, 1000)
```

### ES6異步用法
ES6 Promise可以解決程式碼層次過多的問題
```javascript
// ES6 Promise用法
const promise01 = new Promise((reslove, reject) => {
	console.log('first step')
	// // 執行.then第一個函數，狀態為成功
	// reslove('success')
	// // 執行.then第二個函數，狀態為拒絕
	// reject('error')
	// 執行catch函數，表示出現異常
	throw 'test exception'
})
.then(
	reslove => { // 成功時調用
		console.log('reslove message : ' + reslove)
		return 'sucess = first' // 返回做為下一個then的參數
	},
	reject => { // 拒絕時調用
		console.log('reject message : ' + reject)
		return 'error = first'
	}
).catch(err =>{ // 異常時的處理
	console.log("have some error : " + error)
})

// Promise可作為變量做後續處理
promise01.then(reslove =>{
		console.log('second step')
		console.log('reslove message : ' + reslove)
	},
	reject => {
		console.log('second step')
		console.log('reject message : ' + reject)
	}
)

// 無限定狀態都會調用，並按順序執行
promise01.then(function(){
	console.log('third step')
}).then(function(){
	console.log('fourth step')
})

```

### ES6處理并發
```javascript
// 并發請求
Promise.all([
	new Promise((resolve, reject) =>{
		// do something
		resolve('first request')
	}),
	new Promise((resolve, reject) =>{
		// do something
		resolve('second request')
	}),
]).then( // 包含所有并發請求的信息
	resolve => {
		console.log(resolve) // ['first request', 'second request']
	}
)
```
