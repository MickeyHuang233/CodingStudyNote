###### tags: `📆2022年` `💻編程/🖼前端` `🗂Overview`

# Axios 異步網路請求
==主要用於學習往後前端框架的預備知識，內容暫不全。==

- [Axios官網](https://axios-http.com/docs/intro)
- Axios是基於ES6 promise的HTTP庫管理異步請求，而不是使用傳統callback的方式，可參考：[[🏖ECMAScirpt6#🏖Promise]]
	- `axios.request()`
	- `axios.get()`
	- `axios.post()`
	- `axios.delete()`
	- `axios.put()`
	- `axios.patch()`
	- `axios.header()`

## 🏖基礎用法
使用CDN的方式靜態引入Axios
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/axios/1.1.3/axios.min.js"></script>
```

```javascript
// 默認是用get請求
// axios('https://randomuser.me/api/')
axios.get('https://randomuser.me/api/')
.then(
    resolve => { // 請求成功時調用
        console.log('1. request success')
        console.log(resolve)
        console.log(resolve.data)
    },
    reject => { // 請求拒絕時調用
        console.log('1. request reject')
        console.log(reject)
        console.log(reject.data)
    }
).catch(exception => { // 請求有異常時調用
    console.log("have some error : " + error)
})

axios.get('https://randomuser.me/api/', {
    // get參數
    name: 'mickey',
    id: 123
}).then(resolve =>{
    console.log('2. request success')
    console.log(resolve.data)
})

axios.get('https://randomuser.me/api/', 'name=mickey&id=123')
.then(resolve =>{
    console.log('2. request success')
    console.log(resolve.data)
})

// post請求
axios.post('https://hexschool-tutorial.herokuapp.com/api/signup', {
    // post參數
    email: 'mickeytest123@gmail.com',
    password: '1234'
}).then(resolve =>{
    console.log('3. request success')
    console.log(resolve.data)
})

// 使用參數的方式指定，比較少用
axios({
    method: 'post',
    url: 'https://hexschool-tutorial.herokuapp.com/api/signup',
    headers: {
        'content-type': 'application/x-www-form-urlencoded'
    },
    data: {
        email: 'mickeytest123@gmail.com',
        password: '1234'
    },
}).then(resolve =>{
    console.log('4. request success')
    console.log(resolve.data)
})
```

## 🏖處理并發請求
使用方式類似於`Promise.all()`，可參考：[[🏖ECMAScirpt6#🏖Promise#ES6處理并發]]

```javascript
// 處理并發請求
axios.all([
    axios.get('https://randomuser.me/api/'),
    axios.post('https://hexschool-tutorial.herokuapp.com/api/signup', {
        email: 'mickeytest233@gmail.com',
        password: '1234'
    }),
]).then(resolve => { // 一次取得所有請求的響應內容
    console.log('2. request success : ')
    for(let i = 0; i < resolve.length; i ++){
        console.log(resolve[i])
    }
}).catch(exception => {
    console.log('2. request exception')
    console.log(exception)
})
```

## 🏖全局配置
基本上[請求配置](https://axios-http.com/zh/docs/req_config)中的所有參數都可以配置成全局參數

```javascript
// axios全局配置
axios.defaults.baseURL = 'https://randomuser.me/'
axios.defaults.timeout = 5000
axios.defaults.headers.post['content-type'] = 'application/x-www-form-urlencoded'
axios.get('api/')
.then(resolve => {
    console.log('3. request success : ')
    console.log(resolve.data)
}, reject => {
    console.log('3. request reject')
})
```

## 🏖axios實例封裝
可參考官方文檔：[axios instance](https://axios-http.com/zh/docs/instance)

```javascript
// axios實例
// 實例配置會覆蓋全局配置
let instance = axios.create({
    baseURL: 'https://randomuser.me/',
    timeout: 5000,
})
instance.get('api/')
.then(resolve => {
    console.log('4. request success : ')
    console.log(resolve.data)
}, reject => {
    console.log('4. request reject')
})

// 建立實列化對象時有指定請求方法，可以直接調用
let instance02 = axios.create({
    baseURL: 'https://randomuser.me/',
    method: 'get',
    timeout: 5000,
})
instance02().then()
```

## 🏖攔截器
- 可參考官方文檔：[axios interceptors](https://axios-http.com/zh/docs/interceptors)
- 應用
	1. 為每個請求帶上參數，如：token、時間戳…
	2. 對返回狀態進行判斷，如：token是否過期…

```javascript
// axios實例
let instance03 = axios.create({
    baseURL: 'https://randomuser.me/',
    timeout: 5000,
})

// 請求欄截器
instance03.interceptors.request.use(
    config => { // 處理攔截的函數
        console.log('interceptor request start')
        return config // 請求放行
    }, exception => { // 處理攔截時有異常的函數
        console.log(exception)
    }
)

// 響應攔截器
instance03.interceptors.response.use(
    response => {
        console.log('interceptor response start')
        return response // 響應放行
    }, exception => {
        console.log(exception)
    }
)

// 發送請求
instance03.get('api/')
.then(resolve => {
    console.log('5. request success : ')
    console.log(resolve.data)
}, reject => {
    console.log('5. request reject')
})
```

## 🏖使用webpack打包
1. `npm init -y`，初始化包管理配置文件package.json
2. `npm install axios -S`，安裝axios套件，並配置至生産環境
3. `npm install webpack webpack-cli -D`，安裝[[🧊Webpack]]套件，並配置至開發環境
4. `npm install html-webpack-plugin -D`，安裝套件
5. 配置webpack.config.js
	```javascript
	const HtmlWebpackPlugin = require('html-webpack-plugin')
	
	module.exports = {
	    // 設置入口文件
	    entry: './src/index.js',
	    plugins: [
	        new HtmlWebpackPlugin({
	            template: './src/index.html'
	        }),
	    ],
	    // 設置為開發模式
	    mode: 'development',
	}
	```
6. 配置package.json
	```json
	{
	  "scripts": {
	    "dev": "webpack"
	  },
	}
	```
7. 可以測試打包是否成功，`npm run dev`
8. index.js使用axios
	```javascript
	// 引入axios
	import axios from "axios";
	
	axios.get('https://randomuser.me/api/')
	.then(resolve => {
	    console.log("1. request success : ")
	    console.log(resolve.data)
	}, reject => {
	    console.log("1. request reject")
	})
	```
