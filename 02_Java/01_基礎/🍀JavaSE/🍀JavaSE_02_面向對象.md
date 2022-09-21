#💻編程/🌠Java/01_Java基礎
# Object Oriented
面向對象內容：封裝 Encapsulation、多載 Override、繼承 Extends、多型 Polymorphism

## 🍀封裝 Encapsulation
1. 成員變數 Member Variable：class內宣告的變數
	* 宣告在類別中的變數-->記憶體建立時，系統會根據型別給初值
	* 作用域：JAVA對存儲范圍非常嚴謹

	||同類別|同package|子孫類|其他package|
	|---|---|---|---|---|
	|\+ public|V|V|V|V|
	|\# protected<br>class無法用|V|V|V||
	|~ default(沒指定作用域)|V|V|||
	|\- private<br>class無法用|V||||
2. 區域變數 Local Variable：宣告在方法中的變數
* 記憶體建立時，系統不會根據型別給初值
* 不能用static、public/protected/private修飾
* NULL值作為方法的參數都會有錯誤(NPE)

## 🍀多載方法 Overloading Method
* 參數個數不同
* 或參數個數相同，但對應的資料型別不同
```java
public doSomething(int param01);

public doSomething(int param01, int param02);

public doSomething(int param01, float param02);
```

## 🍀繼承 Extends
```java
[public] [final|avstract] class <類別名> [extends <父類別>]{}
```

### 建構子 Constructor
```java
Customer customer = new Customer();
public Customer(){}//若類別沒有任何Constructor時，JAVA會自動生成一個空的Constructor
```
* Constructor could be overloading
* 物件建立過程
	1.  依類別定義在HEAP Memory配置物件記憶體
	2.  依據資料型別設定屬性的初值
	3.  將程式中各屬性明確宣告的初值指派給屬性
	4.  依據參數對應執行建構子程式，完成物件的建立
	5.  透過new ClassName()回傳該物件的參考

### this
* `this.變數名稱`，表示成員變數
* 當變數與屬性變數同名時：`this.變量名`，代表物件屬性；變量名，代表方法的參數(常在建構子出現)

### super
* `super.變數名稱`，表示父類別物件
* `super()`，只能用於建講子第一行，呼叫父類建構子
* 若子類的建構子無`this()`或`super()`時，java會自動寫入`super()`

### package
```java
package tw.michong  
import java.util.\*;  
import java.util.Date;
```

### 包裝類別 Wrapper Classes
|Primitive Type|Wrapper class|
|---|---|
|boolean|Boolean|
|char|Character|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|

## 🍀多型 Polymorphism
```java
Customer v = new VIP();
```
父類編譯，子類執行；決定可以做什麼，不能做什麼(方法)
父類別的狀態，子類別的物件

### 多型判斷 instanceof 
```java
( Object reference variable ) instanceof  (class/interface type)
```
* 必須是合理的判斷；不合理的判斷為兩個無關物件的判斷(無子、父類別)
* 物件轉型，為了要使畢物件VIP中的方法，所以使用物件轉型，但必須是合理的轉型(兩物件要有關系)
```java
/* 
用於多型方法的判斷
多加判斷，以免錯誤(專案常有此問題)
*/
public double order(Customer c, Product p, int q){
if(c instanceof GoldenVIP){
//語句 
}else if(c instanceof VIP){
 //語句
}else{
 //語句
}//判斷順序應先從子類別至父類別
}
```

### 抽象類 Abstract Method
* 定義：只要有抽象方法的類別就是抽象類別(abstract class)
* 抽象類限制：抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。
* 目的：用技術解決人為疏失<--為了覆寫方法
* 子類限制：沒有實做的方法，繼承一定要覆寫
另外，抽象子類別可不用覆寫抽象方法

``java
public abstract class Employee{  
public abstract double getSalary();  
}  
public class Engineer extends Employee{  
@Override  
pulbic double getSalary(){};  
}``

### 介面 Interface
* 全部都是抽象方法
* Interface(sub interface)可繼承Interface(super interface)，也可多繼承
* extends先寫, implement後寫
```java
public interface Worker{
	public void work();
	public void reward();
}

public class Employee implement Worker{
	@Override
	public void work(){
		// doSomething
	};
 
	@Override
	public void reward(){
		// doSomething
	};
}
```

### 靜態變量static
* `static int v;`：屬於類別的變數，如：消費稅率…等。
* `int v;`：屬於物件的變數，編號…等。
* 使用static變數無需建立物件，如：```System.out.println(Math.PI);```
	```java
	類別名稱.變量名稱;//static變量
	類別名稱.方法名稱();//static方法
	```
* 因為會先加載static屬性和方法，還沒建立物件(non-static)的屬性和方法，因此會找不到值。
	* static member-->類別加載時呼叫方法和建立屬性
	static initializer：配置static變數，由上住下執行static block
	```java
	static{  
		PI = 3.14;//初始化static屬性  
		//或是前置初始化作業邏輯  
	}
	```
	* non-static member-->建立物件時才會有物件的方法和屬性

* static import
```java
import static packagename.classname.\*;  
import static packagename.classname.membername;//static Attribute, static Method
```

### final
```java
public final int GOOGLE_URL = https://www.google.com.tw;  
```
* 類別宣告為final，代表此類別無法被繼承
* 方法宣告為final，代表此方法無法被覆寫
* 變量宣告為final，代表此變數為常數，設定後無法再補修改(public static final)，習慣用大寫

### enum
有限定物件的class
*　舊式列舉型態，會用static final
```java
public class Seasons{  
	public static final int SPRING = 0;  
	pulbic static final int SUMMER = 1;  
	public static final int FALL = 2;  
	public static final int WINTER = 3;  
}
```
* 新式列舉型態，可宣告類別，每個ENUM都會有相同的資料類型
```java
public enum Eseasons{  
	SPRING('春'), SUMMER('夏'), FALL('秋'), WINTER('冬')  
}
```


|方法|描述|
|---|---|
|values()|回傳一個所有該列舉型別之列舉值的陣列|
|valueOf(name: String)|回傳符合參數值的列舉值物件|
|name()|回傳列興值物件的名稱|
|ordinal()|回傳列舉值物件的序號|











