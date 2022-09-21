# JavaSE_04_IO
`java.io`
[Java8 官方API文檔 IO](https://docs.oracle.com/javase/8/docs/api/java/io/package-frame.html)
[Java.io包方法說明(中文)](http://tw.gitbook.net/java/io/index.html)
![JavaSE_04_IO_01](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%8D%80JavaSE/images/JavaSE_04_IO_01.png?raw=true)
流：數據(字符、字節)，1個字符=2個字節=16個二進制位

### 頂級父類
|        | 輸入流                    | 輸出流                     |
| ------ | ------------------------- | -------------------------- |
| 字節流 | 字節輸入流<br>InputStream | 字節輸出流<br>OutputStream |
| 字符流 | 字符輸入流<br>Reader      | 字符輸出流<br>Writer       |

## 🍀File
[File類方法說明](http://tw.gitbook.net/java/io/java_io_file.html)
[Java8 官方API文檔 File](https://docs.oracle.com/javase/8/docs/api/java/io/File.html)
一個File物件代表一個檔案或一個資料夾
```java
String f = "c://hello";
File f1 = new File("hi.txt");
File f2 = new File(f + f1);
```
Windows的路徑分隔符(\\)和Linux的路徑分隔符(/)不一樣，若是跨平台系統需要使用`File.separator`代替分隔符。

### getParentFile()
获得父目录
```java
public class FileDemo {
   public static void main(String[] args) {
  	File f = null;
  	File f1 = null;
  	String v;
  	boolean bool = false;
      
  	try{
     	f = new File("C:\test.txt");// create new file
     	f1 = f.getParentFile();// returns abstract parent pathname
     	v = f1.getAbsolutePath();// absolute path from abstract pathname
     	bool = f.exists();// true if the file path exists
     	if(bool)// if file exists
     	{
        	System.out.print("Parent file path: "+v);//Parent file path: C:
     	}
  	}catch(Exception e){
     	e.printStackTrace();
  	}
   }
}
```

### mkdir()及mkdirs()
建立資料夾
```java
File file_date = new File("D:\\yyy\\2010-02-28"); 
boolean file_mkdir = file_date.mkdir(); //若路徑不存在會報錯
boolean file_mkdirs = file_date.mkdirs(); //若路徑不存在会自动建立不存在的父文建夹
```

## 🍀字節流
在電腦中都是以字節的型式保存。

### 字節輸出流 FileOutputStream
[Java8 官方API文檔 FileOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FileOutputStream.html)
- 構造方法
	```java
	FileOutputStream(File file)
	FileOutputStream(String name)
	// append為追加寫開關
	// true：創建對象不覆蓋原文件，繼續在文件末端寫數據
	// false：創建新文件，覆蓋原文件
	FileOutputStream(File file, boolean append)
	FileOutputStream(String name, boolean append)
	```
- 數據寫出原理
	Java程序-->JVM-->OS-->OS調用寫數據的方法-->數據寫入文件中
	1. 先在硬盤裡建立如"test.txt"的文件，並將FileOutputStream指向此文件
	2. 將要寫入的數據轉為二進制，並放至文件中
	3. 打開文件時都會先查編碼表並轉換表示，如SACII表：97-->a
- 使用步驟
	```java
	// 1. 創建FileOutputStream對象，建構子中傳運寫入數據的目的地
	FileOutputStream fos = new FileOutputStream("test.txt");
	// 2. 使用write()，把數據寫入文件中
	fos.write(97);
	// 3. 釋放資源，內存清空，提高效率
	fos.close();
	```

	```java
	FileOutputStream fos;
	fos = new FileOutputStream("test.txt");
	// 一次寫多個字節：
	// 若第一個字節是正數(-~127)，則會查詢ASCII表
	// 若第一個字節是負數，則會和第二個字節組成一個中文，查詢系覺默認碼表(GBK)
	byte[] bytes01 = {65, 66, 67, 78, 69};
	byte[] bytes02 = {-65, -66, -67, 78, 69};
	fos.write(bytes01);
	// fos.flush();// 刷新至文件中
	fos.close();// 刷新側文件中並關閉流

	// 字串轉為字節數組
	byte[] bytes03 = "測試".getBytes();
	System.out.println(Arrays.toString(bytes03));
	```
- 換行：windows(\\r\\n)、Linux(/n)、Mac(\\r)
	```java
	fos.write("\r\n".getBytes());
	```

### 字節輸入流 FileInputStream
[Java8 官方API文檔 FileInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FileInputStream.html)

- 構造方法
	```java
	FileInputStream(File file)
	FileInputStream(String name)
	```
- 使用步驟
	```java
	// 1. 創建FileInputStream對象，建構子綁定要讀取的數據源
	FileInputStream fis = new FileInputStream("test.txt");
	// 2. 使用read()，把讀取文件一個字節，讀到未尾返回-1
	int len = 0;
	while((len = fis.read()) != -1){
		// do something
	}
	// 3. 釋放資源
	fis.close();
	```

	```java
	FileInputStream fis = new FileInputStream("test.txt");
	// read(byte[] b)，從輸入流讀取一定數量的字節
	// 1. 緩沖作用
	// 2. 數組長度設定為1044(1kb)的整數倍
	Byte[] bytes = new byte[1024];
	int len = 0;// 讀取的有效字節個數
	while((len = fis.read()) != -1){
		System.out.println(new String(bytes), 0, len);// 只將有效轉為String
	}
	System.out.println(len);
	fis.close();
	```

## 🍀字符流
字節流讀取文本時，讀取中文可能有問題，因為一個中文字符可能占用多個字節，所以字符流用於讀取字符。
- GBK：占用2個字節
- UTF-8：占用3個字節

### 字符輸出流 FileWriter
[Java8 官方API文檔 FileWriter](https://docs.oracle.com/javase/8/docs/api/java/io/FileWriter.html)
用起來和FileOutputStream類及，只是將byte改為char
```java
FileWriter fw = new FileWriter("test.txt");
char[] cs = {'a', 'b', 'c', 'd', 'e'};
fw.write(cs);// 寫入字符數組，abcde
fw.write(cs, 1, 3);// 寫入部分字符數組，bcd
fw.write("測試字串");// 寫入字串
fw.write("測試字串", 2, 2);// 寫入部分字串
fw.close();
```

### 字符輸入流 FileReader
[Java8 官方API文檔 FileReader](https://docs.oracle.com/javase/8/docs/api/java/io/FileReader.html)
作用：將硬盤文件中的數據以字符的方式讀取至內存中

- 構造方法
	```java
	FileReader(File file)
	FileReader(String fileName)
	```
- 使用步驟
	```java
	// 1. 建立FileReader對象
	FileReader fr = new FileReader("test.txt");
	// 2. 讀取文件
	// 需要一次讀多個可用read(char[] c)
	int len = 0;
	while((len = fr.read()) != -1){
		System.out.println((char)len);
	}
	//  3. 釋放資源
	fr.close();
	```

## 🍀IO流的異常處理
### Java7以前的處理
缺點：比較麻煩
```java
FileWriter fw = null;
try{
	fw = new FileWriter("test.txt");
	char[] cs = {'a', 'b', 'c', 'd', 'e'};
	fw.write(cs);
}catch(IOException e){
	// 異常處理
}finally{
	if(fw != null){
		fw.close();
	}
}
```

### Java7新特性
try()中定義流對象，此對象只在try中有效，並且不用寫finally會自動釋放對象。
```java
try(FileWriter fw = new FileWriter("test.txt");){
	char[] cs = {'a', 'b', 'c', 'd', 'e'};
	fw.write(cs);
}catch(IOException e){
	// 異常處理
}
```

### Java9新特性
try()中放入先前定義的流對象，執行try完成後自動釋放對象。
```java
FileWriter fw1 = new FileWriter("test1.txt");
FileWriter fw2 = new FileWriter("test2.txt");
try(fw1;fw2){
	char[] cs = {'a', 'b', 'c', 'd', 'e'};
	fw1.write(cs);
	fw2.write(cs);
}catch(IOException e){
	// 異常處理
}
```

## Console
可以藉由System新增的console()方法取得標 準輸入輸出裝置的Console物件，並利用它來執行一些簡單的主控台文字輸入輸出
[Console類方法說明](http://tw.gitbook.net/java/io/java_io_console.html)

```java
public class ConsoleDemo {
   public static void main(String[] args) {
       BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        System.out.print("Enter String");
        String s = br.readLine();
        System.out.print("Enter Integer:");
        try{
            int i = Integer.parseInt(br.readLine());
        }catch(NumberFormatException nfe){
            System.err.println("Invalid Format!");
        }
   }
}
```

## 🍀Properties類
唯一與IO流相結合的集合。表示一個持久的屬性集，Properties可保存在流中或從流中加載，屬性列表中每個鍵及值都是字符串。
[Java8 官方API文檔 Properties](https://docs.oracle.com/javase/8/docs/api/java/util/Properties.html)

### 輸出properties
```java
Properties prop = new Properties();
// 添加數據
prop.setProperty("Mickey", "AAA123");
prop.setProperty("John", "ABC223");
prop.setProperty("Mike", "BBC456");

// 使用字節流輸出Properties至硬盤
// void store(Writer writer, String comments)
// 其中comments為說明，不能使用中文，默認是Unicode編碼，一般為空
FilWriter fw = new FileWriter("testProperties01.txt");
prop.store(fw, "save data");
fw.close();

// 使用字符流輸出Properties至硬盤
// void store(OutputStream out, String comments)
prop.store(new FileOutputStream("testProperties02.txt"), "");
```

### 輸入properties
文件中鍵值對連接符號可用=或(空格)…其他符號
```java
Properties prop = new Properties();
// 使用字符流將硬盤鍵值對信息輸入至集合中
// void load(Reader reader)
prop.load(new FileReader("testProperties.txt"));

// 使用字節流將硬盤鍵值對信息輸入至集合中
// void load(InputStream inputStream)
prop.load(new InputStream("testProperties.txt"));

// 取出Properties鍵的集合
Set<String> set = prop.stringPropertyNames();
// 遍歷Set取得Properties值
for (String key : set) {
	String value = prop.getProperty(key);
}
```

## 🍀緩沖流
創建內置緩沖區數據，減少系統IO次數；對4個基本的`FileXxx`流提高效率，因此也分為4個流。

### 字節輸出緩沖流 BufferedOutputStream
[Java8 官方API文檔 BufferedOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedOutputStream.html)
- 構造方法
```java
BufferedOutputStream(OutputStream out)
BufferedOutputStream(OutputStream out, int size)// 指定緩沖區大小
```
- 使用步驟
```java
// 1. 建立FileOutputStream
FileOutputStream fos = new FileOutputStream("test.txt");
// 2. 建立BufferedOutputStream
BufferedOutputStream bos = new BufferedOutputStream(fos);
// 3. 將數據寫入內部緩沖區中
bos.write("測試輸出".getBytes());
// 4. 刷新數據至文件中
bos.flush();
// 5. 釋放資源
bos.close();
```

### 字節輸入緩沖流 BufferedInputStream
[Java8 官方API文檔 BufferedInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedInputStream.html)
使用方法和BufferedOutputStream類似，相關方法可參考FileInputStream或API文檔。

### 字符輸出緩沖流 BufferedReader
[Java8 官方API文檔 BufferedReader](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedReader.html)
使用方法和BufferedOutputStream類似，相關方法可參考FileReader或API文檔。

### 字符輸入緩沖流 BufferedWriter
[Java8 官方API文檔 BufferedWriter](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedWriter.html)
使用方法和BufferedOutputStream類似，相關方法可參考FileWriter或API文檔。

## 🍀轉換流
作用：解決亂碼問題
字符編碼：一最自然語言的字符與二進制數間的對應規則。
編碼表(字符集)：支持所有字符的集合，包括各國文字、符號、數字…等。
![JavaSE_04_IO_02](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%8D%80JavaSE/images/JavaSE_04_IO_02.png?raw=true)

### 字符輸出轉換流 OutputStreamWriter
[Java8 官方API文檔 OutputStreamWriter](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStreamWriter.html)
- 構造方法
	```java
	OutputStreamWriter(OutputStream out)
	OutputStreamWriter(OutputStream out, String charsetName)
	```
- 使用步驟
```java
// 1. 建立OutputStreamWriter對象
OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("test.txt"), "UTF-8");
// 2. 使用write(), 把字符轉為字節存至緩沖區中(編碼)
osw.wirte("測試字串");
// 3. 使用flush，將內存緩沖區的字節刷新到文件
osw.flush();
// 4. 釋放資源
osw.close();
```

### 字符輸入轉換流 InputStreamReader
[Java8 官方API文檔 InputStreamReader](https://docs.oracle.com/javase/8/docs/api/java/io/InputStreamReader.html)
- 構造方法
	```java
	InputStreamReader(InputStream in)
	InputStreamReader(InputStream in, String charsetName)
	```
- 使用步驟
	```java
	// 1. 創建InputStreamReader對象
	InputStreamReader isr = new InputStreamReader(new FileInputStream("test.txt"), "UTF-8");
	// 2. 使用read()讀取文件
	int len = 0;
	while((len=isr.read()) != -1){
		System.out.println(len);
	}
	// 3. 釋放資源
	isr.close();
	```

## 🍀序列化流 Serializable
[Java8 官方API文檔 Serializable](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)
序列化：將對象寫入文件保存
反序列化：從文件讀取對象

### 序列化 ObjectOutputStream
[Java8 官方API文檔 ObjectOutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/ObjectOutputStream.html)
- 構造方法
	```java
	ObjectOutputStream(OutputStream out)
	```
- 使用步驟
	```java
	/*
		序列化和反序列化的對象需要實現java.io.Serializable接口(標記型接口)
	*/
	public class Person implements Serializable{
		private String name;
		private int age;
		// 構造方法、getter、setter、toString省略
	}
	```

	```java
	// 1. 建立ObjectOutputStream對象
	ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("test.txt"));
	// 2. 使用writeObject()，將對象寫入文件中
	oos.writeObject(new Person("Mickey", 123));
	// 3. 釋放資源
	oos.close();
	```

### 反序列化 ObjectInputStream
[Java8 官方API文檔 ObjectInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/ObjectInputStream.html)

- 構造方法
	```java
	ObjectInputStream(InputStream in)
	```
- 使用步驟
	```java
	// 1. 建立ObjectInputStream對象
	ObjectInputStream ois = new ObjectInputStream(new FileInputStream("test.txt"));
	// 2. 使用readObject讀取保存對象文件
	Object o = ois.readObject();
	// 3. 釋放資源
	ois.close();
	// 使用讀取的對象
	Person p = (Person) o;
	System.out.println(p.toString());
	```

### transient關鍵字
- 被transient修飾的成員變量，==不能被序列化==。
- static為靜態關鍵字，靜態優先於非靜態加載到內存中，因此被static修飾的成員變量不能被序列化，序列化的都是對象。

### InvalidClassException異常
- 問題：
	1. 編譯器(javac.exe)會將Person.java文件編譯成Person.class，而Person類實現Serializable接口，就會根據類的定義在class文件中添加一個序列號(seriaVersionUID)
	2. 若類的定義發生改變，序列號(seriaVersionUID)也會發生改變，若序列號不同則會出現InvalidClassException異常。
- 解決方案：手動給類添加序列號，因此編譯成class文件時就不會生成新的序列號
	```java
	static final long serialVersionUID = 42L;
	```

## 🍀打印流 PrintStream
打印流只負責數據輸出，不負責數據的讀取。
[Java8 官方API文檔 PrintStream](https://docs.oracle.com/javase/8/docs/api/java/io/PrintStream.html)

- 構造方法
	```java
	// 參數為要輸出的類型
	PrintStream(File file)// 輸出文件
	PrintStream(OutputStream out)// 字節輸出流
	PrintStream(String fileName)// 文件路徑
	```
- 使用方法
	另外，`System.setOut(PrintStream out)`改變輸出語句目的地改為參數中傳遞的打印流目的地
	```java
	PrintStream ps = new PrintStream("test.txt");
	ps.write(97);// 使用繼承父類的write()寫數據，查看數據會查詢編碼表，97->a
	ps.println(97);// 使用自己扲有方法print / println寫數據，數據會原樣輸出，97->97
	ps.close;
	```

### 格式化輸出
```java
PrintStream ps = new PrintStream(new FileOutputStream(new File("test.txt"))) ;
String name = "Mickey" ;  
int age = 123 ;  
char sex ='M';  
ps.printf("姓名：%s；年齡：%d；性別：%c",name,age,score,sex);  
ps.close();
```
|Code|Description|
|---|---|
|%s|Formats the argument as a string, usually by calling the toString method on the object.|
|%d<br>%o|Formats an integer, as a decimal, octal, or hexadecimal value.|
|%f<br>%g|Formats a floating point number. The %g code uses scientific notation.|
|%n|Inserts a newline character to the string or stream.|
|%%|Inserts the % character to the string or stream.|
|%b|Formats a boolean value.|







