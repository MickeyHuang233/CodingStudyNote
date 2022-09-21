###### tags: `📆2021年`
###### tags: `💻編程/🚋共用`
###### tags: `🗂Overview`

# JSON
JSON(JavaScript Object Notation，JS對象表示法)，以字串為基底去儲存和傳送簡單結構資料，你可以透過特定的格式去儲存任何資料(字串、數字、陣列、物件)。
[JSON Editor Online](https://jsoneditoronline.org/)

- 優點：
	1. 相容性高
	2. 格式容易瞭解，閱讀及修改方便
	3. 支援格式：number、string、boolean、null、array、object
	4. 許多程式都支援函式庫讀取或修改JSON資料
- 應用：
	1. 使用者點選了線上產品縮圖
	2. JavaScript 透過 AJAX 方式將產品 ID 傳送給伺服器端
	3. 伺服器端收到 ID，將產品資料(如：價格、描述)編碼成 JSON 資料，並且回傳給瀏覽器
	4. JavaScript 收到 JSON 資料，將其解碼(decode) 並且將資料顯示在網頁上
- 例子：
	```json
	[
		{
			"name" : "mickey",
			"age" : "123", 
			"gender" : "male",
			"favor" : ["watch TV", "play game", "read"],
			"salary" : "2000"
		},
		{
			"name" : "mike",
			"age" : "233", 
			"gender" : "male",
			"favor" : ["write", "play game", "code"],
			"salary" : "2500"
		}
	]
	```

