#📆2022年 
狀態:: #☑DONE 
完成日期:: 2022-02-05
標籤:: #💻編程/🌠Java/01_Java基礎 #🗂Overview 
子筆記:: 
教程:: [尚硅谷](https://www.bilibili.com/video/BV17W411g7eK)
備註:: 

# Java9新特性
2017年9月公布Java9，並且從Java9開始，後續的Java版本更新周期為==6個月==，並逐漸Oracle JDK商業特性開源；以三年為一個周期發布長期支持版本。

- [Open JDK9 Download](https://jdk.java.net/java-se-ri/9)
- 環境安裝參考：[[🍀JavaSE_00_Overview]]
- [JDK9官方文件](https://docs.oracle.com/javase/9/)
- [JDK9 API Documentation](https://docs.oracle.com/javase/9/docs/api/overview-summary.html)
	- JEP(JDK Enhancement Proposals)：JDK改進提案，非正式的規范。
	- JSR(Java Specification Requests)：Java規范提案，新特性出現在此階段，如Java8的Lambda、Java9的JigSaw。
- JDK9、JRE9目錄結構：[JDK9、JRE9目錄結構](https://www.codenong.com/what-is-the-structure-of-jdk-and-jre-directory-in-java-9/)

## 💡模塊化系統 Jigsaw
後面被稱為Modularity
- Java9之前啟動JVM會將整個jar加載到內存，使得Java運行環境過於巨大、緩慢；Java9則可以根據模塊化的需要加載需要的class
- Java9之前難以對jar包進行封裝，jar包間無明確的依賴關系
- Java9之前無法得知jar包是否存在，或是重覆存在

![image](http://www.tellmehow.co/wp-content/uploads/2017/09/project-jigsaw-in-jdk-9-modularity-comes-to-java-19-638.jpg)

### 被依賴專案準備
- 被依賴專案新增類
	```java
	package com.mickey.importtest;
	public class T08_User1 {
		private String name;
		private int age;
		// getter, setter, 構造器省略
	}
	```
	```java
	package com.mickey.noimporttest;
	public class T08_User2 {
		private String name;
		private int age;
		// getter, setter, 構造器省略
	}
	```
- 新增module-info.java，設置暴露的包
	專案右鍵->Configure->Create module-info.java
	```java
	module com.mickey.importmodule.api {
		exports com.mickey.importtest;
	}
	```

### 導入專案準備
- maven引入依賴包
	```xml
	<dependency>
		<groupId>tw.com.mickey</groupId>
		<artifactId>20220204_java9-test-import</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
	```
- 新增module-info.java
	專案右鍵->Configure->Create module-info.java
- 更新引入的module
	專案右鍵->Build Path->Configure Build Path...
	![[Java9_00_Overview_01_更新引入的module.png]]
- 設置引入的包
	```java
	module java9_test {
		requires org.junit.jupiter.api;
		requires com.mickey.importmodule.api;// 此處是引入module名稱
	}
	```
- 再來就可以在自己的程式中使用
```java
public class T08_JigsawTest {
    private Logger logger = Logger.getLogger(this.getClass().getName());

    @Test
    @DisplayName("測試java9新特性，Jigsaw")
    public void testcase01() {
        final T08_User1 user1 = new T08_User1("Mikcey", 233);
        this.logger.info(user1.toString());
        // 因為T08_User2所在的包沒在module-info.java設置暴露，所以會報錯
        //        final T08_User2 user2 = new T08_User2("Mikcey", 233);
    }
}
```

## 💡jShell命令
- Java 9以前要執行代碼，必須創建文件-->聲明類-->提供測試方法，才可實現，相較於Python和Scala都有交互式編程環境(REPL，read - evaluate - print - loop)還要不方便
- 官方文件：[Java Shell User’s Guide](https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm#JSHEL-GUID-630F27C8-1195-4989-9F6B-2C51D46F52C8)

- `jshell`，啟動jShell
	```console
	a0909@LAPTOP-C500TM84 MINGW64 ~
	$ jshell
	|  Welcome to JShell -- Version 9
	|  For an introduction type: /help intro

	jshell>
	```
- 直接使用java語句
	```console
	jshell> System.out.println("Hello World");
	Hello World
	```
- 添加方法、調用
	```console
	jshell> public int add(int i, int j){
	   ...> int result = i + j;
	   ...> System.out.println("result is "+result);
	   ...> return result;
	   ...> }
	|  created method add(int,int)

	jshell> System.out.println(add(50, 60));
	result is 110
	110
	```
- 從界面修改方法
	```console
	jshell> /edit add
	```
- `/list`，查看執行記錄
	```console
	jshell> /list

	   1 : System.out.println("Hello World");
	   2 : public int add(int i, int j){
		   int result = i + j;
		   System.out.println("result is "+result);
		   return result;
		   }
	   3 : System.out.println(add(50, 60));
	```
- 申明變量
	```console
	jshell> String str = "my first jshell";
	str ==> "my first jshell"
	```
- `/vars`，查看已申明變量
```console
jshell> /vars
|    String str = "my first jshell"
```
- `/methods`，查看已申明方法
- `/open <file>`，導入java文件
- `/save <file>`，導出java文件
- `/imports`，查看導入的包
	```console
	jshell> /imports
	|    import java.io.*
	|    import java.math.*
	|    import java.net.*
	|    import java.nio.file.*
	|    import java.util.*
	|    import java.util.concurrent.*
	|    import java.util.function.*
	|    import java.util.prefs.*
	|    import java.util.regex.*
	|    import java.util.stream.*
	```
- `/exit`，離開jShell

## 💡多版本兼容jar包
參考：[Multi-Release Jar Files](https://www.baeldung.com/java-multi-release-jar)

- 編譯Java7
	```console
	javac --release 7 -d classes src\main\java\com\baeldung\multireleaseapp\*.java
	```
- 編譯Java9
	```console
	javac --release 9 -d classes-9 src\main\java9\com\baeldung\multireleaseapp\*.java
	```
- 導出多版本兼容jar包
	```console
	jar --create --file target/mrjar.jar --main-class com.baeldung.multireleaseapp.App -C classes . --release 9 -C classes-9 .
	```

## 💡接口定義私有方法
```java
public interface T12_InterfacePrivateMethod {
    // JDK7，只能聲明全局常量(public static final)和抽象方法(public abstract)
    public void method1();

    // JDK8，聲明靜態方法 和 默認方法
    public static void method2() {
        System.out.println("do something in method2.");
    }
    public default void method3() {
        System.out.println("do something in method3.");
        this.method4();
    }

    // JDK9，聲明私有方法
    private void method4() {
        System.out.println("do something in method4.");
    }
}
```

## 💡鑽石操作符使用改進
鑽石操作符(Diamond Operator)與匿名實現類同時使用。

```java
@Test
@DisplayName("鑽石操作符(Diamond Operator)使用改進")
public void testcase01() {
	// JDK7，<>中必須填寫類型
	final Set<String> set1 = new HashSet<String>();

	// JDK8，<>中可空白，類型推斷
	final Set<String> set2 = new HashSet<>();

	// JDK9，<>可以與匿名實現類同時使用
	final Set<String> set3 = new HashSet<>();
	set3.add("Mikcey");
	set3.add("Tai");
	set3.add("Molly");
	final List<String> list3 = new ArrayList<>(set3);
	for (final String str : list3) {
		System.out.println(str);
	}
}
```

## 💡try語句使用改進
```java
@Test
@DisplayName("JDK7之前寫法")
public void testcase01() {
	final InputStreamReader reader = new InputStreamReader(System.in);
	try {
		reader.read();
	} catch (final IOException e) {
		e.printStackTrace();
	} finally {
		if (reader != null) {
			try {
				reader.close();
			} catch (final IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

```java
@Test
@DisplayName("JDK7、JDK8建議寫法，執行完畢會自己調用close()")
public void testcase02() {
	try (InputStreamReader reader = new InputStreamReader(System.in)) {
		reader.read();
	} catch (final IOException e) {
		e.printStackTrace();
	}
}
```

```java
@Test
@DisplayName("JDK9可在try()中添加已實例化的final對象")
public void testcase03() {
	final InputStreamReader reader = new InputStreamReader(System.in);
	final OutputStreamWriter writer = new OutputStreamWriter(System.out);
	try (reader; writer) {
		reader.read();
	} catch (final IOException e) {
		e.printStackTrace();
	}
}
```

## 💡下劃線使用限制
JDK9之前，變量名可以為_；而JDK9之後就不允許。
```java
@Test
@DisplayName("JDK9之前，變量名可以為_")
public void testcase01() {
	final String _ = "under line";
	System.out.println(_);
}
```

## 💡String存儲結構改變
JDK9以前，String底層是以`char[]`進行存儲；而JDK9改為用`byte[]`進行存儲，並用encoding-flag存使用的編碼器，StringBuilder、StringBuffer(線程安全)也做相應的改變。
官方說明：[JEP 254: Compact Strings](https://openjdk.java.net/jeps/254)

## 💡建立只讀集合
```java
@Test
@DisplayName("JDK9前，建立只讀集合")
public void testcase01() {
	final List<String> list = new ArrayList<>();
	list.add("Mickey");
	list.add("Tai");
	list.add("Molly");
	// 建立只讀List
	final Collection<String> onlyReadList = Collections.unmodifiableCollection(list);
	onlyReadList.forEach(System.out::println);
	//        onlyReadList.add("Jack");// 執行會報錯，java.lang.UnsupportedOperationException
	// 建立只讀Set
	final Set<String> onlyReadSet = Collections.unmodifiableSet(new HashSet<String>(list));
	// 建立只讀Map
	final Map<String, Integer> onlyReadMap = Collections.unmodifiableMap(new HashMap<String, Integer>() {
		{
			for (final String str : list) {
				this.put(str, (int) (100 * Math.random()));
			}
		}
	});
	onlyReadMap.forEach((k, v) -> System.out.println(k + " - " + v));
}
```

```java
@Test
@DisplayName("JDK9，建立只讀集合")
public void testcase02() {
	// 建立只讀List
	final List<String> onlyReadList = List.of("Mickey", "Tai", "Molly");
	onlyReadList.forEach(System.out::println);
	//        onlyReadList.add("Jack");// 執行會報錯，java.lang.UnsupportedOperationException
	// 建立只讀Set
	final Set<String> onlyReadSet = Set.of("Mickey", "Tai", "Molly");
	// 建立只讀Map
	final Map<String, Integer> onlyReadMap1 = Map.of("Mickey", 233, "Tai", 123, "Molly", 456);
	final Map<String, Integer> onlyReadMap2 = //
			Map.ofEntries(Map.entry("Mickey", 233), Map.entry("Tai", 123), Map.entry("Molly", 456));
}
```

## 💡Stream API新增方法
基礎用法可參考：[[💡Java8_02_StreamAPI]]
- `takeWhile()`，按順序比較，只要遇到不符合的值，後面則不再比較
- `dropWhile()`，和takeWhile類似，只要遇到符合的值，後面則不再比較
- `ofNullable`，可建立完全為null的Stream
- `iterate()`重載方法，可以直接在iterate()中設定停止的判斷

```java
@Test
@DisplayName("Stream API新增方法")
public void testcase01() {
	final List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15);
	System.out.println("takeWhile()");
	list.stream().takeWhile(x -> x < 3 || x > 10).forEach(System.out::println);

	System.out.println("dropWhile()");
	list.stream().dropWhile(x -> x < 3).forEach(System.out::println);

	System.out.println("ofNullable()");
	// JDK8中，Stream不能完全為null；JDK9中，ofNullAble()允許建立單元素為null的Stream
	Stream.of(1, 2, null);
	//        Stream.of(null);// java.lang.NullPointerException
	Stream.ofNullable(null);
	System.out.println(Stream.ofNullable(null).count());// 0

	System.out.println("iterate()重載方法");
	// JDK8，用iterate()産出的Stream需要靠額外的判斷停止
	Stream.iterate(0, x -> x + 1).limit(10).forEach(System.out::println);
	// JDK9，可以直接在iterate()中設定停止的判斷
	Stream.iterate(0, x -> x < 10, x -> x++).forEach(System.out::println);
}
```

## 💡Optional可轉為Stream
基礎用法可參考：[[💡Java8_03_Optional]]

```java
@Test
@DisplayName("Optional新增Stream API")
public void testcase02() {
	final List<String> list = Arrays.asList("Mickey", "Tai", "Molly");
	final Stream<String> flatMap = Optional.ofNullable(list).stream().flatMap(l -> l.stream());
	flatMap.forEach(System.out::println);
}
```

## 💡多分辨率圖像API【了解】
- 比較多用於Android開發
- 使用Direct2D for Windows、GTK+的API，而不是Xlib for Linux
- 提供處理不同分辨率相同圖像的文件
- 定義於`java.awt.image`

## 💡新Http客戶端API
HttpClient可適用於HTTP/2協議

```java
@Test
@DisplayName("新Http客戶端API")
public void testcase01() {
	try {
		final HttpClient client = HttpClient.newHttpClient();
		final HttpRequest request = HttpRequest.newBuilder(URI.create("https://www.google.com.tw/"))//
				.GET().build();
		HttpResponse<String> response = null;
		response = client.send(request, HttpResponse.BodyHandlers.ofString());
		System.out.println(response.statusCode());// 響應碼
		System.out.println(response.version().name());// 協議版本
		System.out.println(response.body());// 響應體
	} catch (final IOException e) {
		e.printStackTrace();
	} catch (final InterruptedException e) {
		e.printStackTrace();
	}
}
```

## 💡Deprecated API【了解】
- Java現在很少做客戶端，因此將Applet API標記為廢棄的
- 主流瀏覽器已經對Java瀏覽器插件的支持，appletviewer工具標記為廢棄

## 💡智能Java編譯工具【了解】
- javac，之前所使用的Java編譯工具-->更新使java9在低版本java中可運行
- sjavac，智能Java編譯工具-->漸漸替代javac

## 💡統一JVM日誌系統【了解】
JDK9前的JVM日誌太過碎片化，導致難以對JVM調試。

## 💡javadoc支持HTML5【了解】
JDK9前的javadoc是HTML4

## 💡Javascript引擎升級->Nashorn【了解】
- JDK8以前使用Rhino(效率較低)，Nashorn為JDK8提供的Javascript引擎；在JDK9進行改進，以提供輕量級Javascript運行
- 提供解析Nashorn的ECMAScript語法樹的API

## 💡Java動態編譯器【了解】
相對於C和C++相比，Java體現出吃內存、啟動慢的缺點，主要因為.java文件需要進行解釋和編譯成電腦可執行的二進制文件，因此出現JIT(just-in-time)編譯器，但JIT編釋器效率慢；JDK9則推出AOT(ahead-of-time)編釋器，但此特性還在試驗階段。

## 💡錯誤記錄
### 使用Junit5測試時找不到相應的module
```console
Error occurred during initialization of boot layer
java.lang.module.FindException: Module org.junit.jupiter.api not found, required by test
```
解決方法：將module-info.java刪除即可