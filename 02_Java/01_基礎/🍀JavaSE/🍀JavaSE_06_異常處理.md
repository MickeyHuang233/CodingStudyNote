#💻編程/🌠Java/01_Java基礎
# Exceptions
Exceptions java.lang.Exception
目的：若有錯誤産生Exception物件，程式還可以繼續運行。
* Throwable
	* Error：硬體錯誤、JVM錯誤：嚴重型錯誤，不處理
	* Exception
		* RuntimeException：邏輯上的錯誤-->不使用例外控制(例外控制比判斷耗資源)
		* Checked Exception：一定要例外控制

```java
try{
  // 程序代码
}catch(异常类型1 异常的变量名1){//異常類型子類先，父類後
  // Exception instanceof (catch語周中的異常類型) = true，運行catch的內容
//Exception instanceof (catch語周中的異常類型) = false，停止運行跳出異常
}catch(异常类型2|异常类型3 异常的变量名2){//java 7後出現的寫法
  // 程序代码
}finally{
  // 無論有無異常都會執行的程序代码
}
```

```java
class bigTrouble{
	// 若有問題産生throws後的例外
	// Exception Override 只能用子類別覆寫
	void trouble() throws IOException, MyException{}  
}

class subTrouble extends bigTrouble{  
	void trouble() throws EOFException{}//若用throws Exception會有錯誤  
}
```

常見的RuntimeException：
* NullPointerException：空指針異常
* NumberFormatException：字符串轉換為數字時的異常，如"ab3"
* ArrayIndexOutOfBoundsException：數組越異
* StringIndexOutOfBoundsException：字符串越界
* ClassCastException：類型轉換錯誤
* ArithmeticException：算术错误，典型的就是0作为除数的时候
* IllegalArgumentException：非法参数，在把字符串转换成数字的时候经常出现的一个异常

## 🍀内置異常類
### Java 内置異常类
|異常|描述|
|---|---|
|ArithmeticException|当出现异常的运算条件时，抛出此异常。例如，一个整数"除以零"时，抛出此类的一个实例。|
|ArrayIndexOutOfBoundsException|用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引。|
|ArrayStoreException|试图将错误类型的对象存储到一个对象数组时抛出的异常。|
|ClassCastException|当试图将对象强制转换为不是实例的子类时，抛出该异常。|
|IllegalArgumentException|抛出的异常表明向方法传递了一个不合法或不正确的参数。|
|IllegalMonitorStateException|抛出的异常表明某一线程已经试图等待对象的监视器，或者试图通知其他正在等待对象的监视器而本身没有指定监视器的线程。|
|IllegalStateException|在非法或不适当的时间调用方法时产生的信号。换句话说，即 Java 环境或 Java 应用程序没有处于请求操作所要求的适当状态下。|
|IllegalThreadStateException|线程没有处于请求操作所要求的适当状态时抛出的异常。|
|IndexOutOfBoundsException|指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。|
|NegativeArraySizeException|如果应用程序试图创建大小为负的数组，则抛出该异常。|
|NullPointerException|当应用程序试图在需要对象的地方使用 null 时，抛出该异常|
|NumberFormatException|当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。|
|SecurityException|由安全管理器抛出的异常，指示存在安全侵犯。|
|StringIndexOutOfBoundsException|此异常由 String 方法抛出，指示索引或者为负，或者超出字符串的大小。|
|UnsupportedOperationException|当不支持请求的操作时，抛出该异常。|

### 檢查性異常類
|異常|描述|
|---|---|
|ClassNotFoundException|应用程序试图加载类时，找不到相应的类，抛出该异常。|
|CloneNotSupportedException|当调用 Object 类中的 clone 方法克隆对象，但该对象的类无法实现 Cloneable 接口时，抛出该异常。|
|IllegalAccessException|拒绝访问一个类的时候，抛出该异常。|
|InstantiationException|当试图使用 Class 类中的 newInstance 方法创建一个类的实例，而指定的类对象因为是一个接口或是一个抽象类而无法实例化时，抛出该异常。|
|InterruptedException|一个线程被另一个线程中断，抛出该异常。|
|NoSuchFieldException|请求的变量不存在|
|NoSuchMethodException|请求的方法不存在|

## 🍀建立自己的例外類別
```java
public class MyException extends Exception{
	private int errorNumber;
	public MyException(String message, int error){
		super(message);
		errorNumber = error;
	}
	public int getErrorNumber(){
		return errorNumber;
	}
}
```

```java
public  void AAA() throws MyException{//用在宣告可能會有的例外
	boolean successful = false;
	if(!successful){
		throw new MyException("File not found", 404);//産生例外物件
	}
}
```

## 🍀斷言 Assertion
* 需使用==JDK 1.4或以上==的版本
* 概念：系統上線時期常用的除錯工具，assertion就是在程序中的一行，它對一個boolean表達式進行檢查，一個正確程式必須保證這個boolean的值為true；如果該值為false，表示程式已經處於不正確的狀態下，系統將給出警告或退出。一般來說，assertion用於保證程序最基本、關鍵處的正確性。
* 注意：加入Assertion機制後程序內部的邏輯、流程都不能改變

assert <boolean_expression>:<detail_expression>;
```java
public class TestAssert{  
	public static void main(String\[\] args){  
	assert 0 < value;//0 < value為false时，将会抛出一个AssertionError  
		System.out.println(name);  
	}  
}
```
  
assert <boolean_expression>:<detail_expression>;
當<boolean_expression>的值為false，則中斷程序抛出一个AssertionError並輸出<detail_expression>字串
```java
public class TestAssert{  
	public static void main(String\[\] args){  
		String name = "abner chai";  
		//String name = null;  
		assert (name!=null):"变量name为空null";//变量name为null时，将会抛出一个AssertionError，并输出错误信息。  
		System.out.println(name);  
	}  
}
```

預設Assertion是關閉的，打開系統類別的assertion功能
```console
java -enableassertions TestAssert  
java -ea TestAssert
```
  
關閉系統類別的assertion功能
```console
java -disenablesystemassertions TestAssert  
java -dsa TestAssert
```

## 🍀AutoCloseable
 1.7 版本以後，所有的Closeable物件都繼承了AutoCloseable；也同時提供了一個新的語法：`try-with-resoure statement`
JVM會在離開try之後會自動依建構的反向順序呼叫`AutoCloseable.close()`
在try 裡面的東西必需是AutoCloseable的物件(Closeable、AutoCloseable都是interface)

```java
public void tryWithResource() throws Exception {  
	String sql = "This is a sql statement";  
	try (Connection conn = getConnection();  
		PreparedStatement ps = conn.prepareStatement(sql)) {  
		ps.setString(1, "This is parameter");  
		try (ResultSet rs = ps.executeQuery(sql);) {  
			while (rs.next()) {  
			rs.getString(1);  
			}  
		}  
	}  
}
```
























































