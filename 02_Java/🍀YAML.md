#📆2022年 
狀態:: #☑DONE
完成日期:: 2022-02-06
標籤:: #💻編程/🚋共用 #🗂Overview
子筆記:: 
教程:: 
備註:: 

# YAML
YAML Ain't Markup Language，ymal既是也不是一個標記語言；yaml是以數據為中心，比json、xml更適合做配置文件。

```yaml
#yaml范例  
server:  
  port: 8081
```
  
```xml
<!-- xml范例 -->  
<server>  
	<port>8081</port>  
</server>
```

## 🍀yaml基本語法
1.  key: value，表示一對鍵值對(空格必須有)
2.  使用縮進表示層級關系，而且只能用空格縮進
3.  縮進空格數相同代表相同的層級
4.  大小寫敏感

## 🍀yaml支持的數據結構
1.  普通的值：數字、字符串、布爾
	- `key: value`，字符串默認不用加單引號或雙引號
	- `“”`(雙引號)：不會轉義字符串的特殊字符，如：\n就是換行
	- `‘’`(單引號)：會轉義字符串的特殊字符，如：\n就是\n文字
2.  對象：屬性和值
	- `key: value`
3.  數組
	- `- value`，表示數組的元素

- 多行寫法：
	```yaml
	#yaml表示對象  
	student:  
	 id: 1  
	 name: mickey  
	 isPass: true  
	 #輸出：hello!(換行)my name is mickey.  
	 introduction01: "hello!\nmy name is mickey."  
	 #輸出：hello!\nmy name is mickey.  
	 introduction02: 'hello!\nmy name is mickey.'  
	#yaml表示數組  
	class:  
	 - Java  
	 - Spring  
	 - Hibernate  
	 - SpringMVC
	```
- 行內寫法：
	```yaml
	#yaml表示對象  
	student: { id: 1,name: mickey,isPass: true,introduction01: "hello!\nmy name is mickey."}  
	#yaml表示數組  
	class: [Java,Spring,Hibernate,SpringMVC]
	```










  





