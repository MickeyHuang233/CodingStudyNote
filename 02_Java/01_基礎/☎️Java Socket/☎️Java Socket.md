#📆2021年 
狀態:: #☑DONE
完成日期:: 2021-02-01
標籤:: #💻編程/🌠Java/01_Java基礎 #🗂Overview 
子筆記::
教程:: 
備註:: 筆記搬運(Google Drive-->Obsidian)

# Java Socket
## ☎️基礎知識點
1.  網絡：將不同區域的計算機連接一起；局域網、城域網、互聯網
2.  地址：IP地址，確定網絡上，一個絕對地址、位置
3.  端口號：區分計算機軟件的，0-65535
	- 在同一個協議下，端口號不能重複，不同協議下可以重複
	- 1024以下的不要使用，80-->http，21-->ftp
4.  資源定位：URL-->統一資源+定位符；URI-->統一資源
5.  數據的傳輸
	- 協議：TCP和UDP協議
	- TCP(Transfer Confrol Protocol)：電話，類似三次握手，面向連接，安全可靠，效率低下
	- UDP(User Datagram Protocol)：短信，非面向連接，效率高
	- 傳輸：先封裝，後拆封
	![Java Socket_01_TCP／IP模型]()

## ☎️相關Java類
- InetAddress(不包含端口)  InetSocketAddress(包含端口)
	DNS域名解析
- URL
- TCP: ServerSocket  Socket
- UDP: DatagramSocket  DatagramPacket

### InetAddress
封裝IP及DNS域名解析，不封裝端口
[Java8 官方API文檔 InetAddress](https://docs.oracle.com/javase/8/docs/api/java/net/InetAddress.html)
```java
InetAddress addr = InetAddress.getLocalHost();//使用getLocalHost方法創建InetAddress對象  
System.out.println(addr.getHostAddress());//192.168.1.100  
System.out.println(addr.getHostName());//返回域名或計算機名，DESKTOP-2496BOD  
addr = InetAddress.getByName("www.163.com");//根據域名得到InetAddress對象  
System.out.println(addr.getHostAddress());//210.61.180.160  
System.out.println(addr.getHostName());//www.163.com  
addr = InetAddress.getByName("61.135.253.15");//根據IP得到InetAddress對象  
System.out.println(addr.getHostAddress());//61.135.253.15  
System.out.println(addr.getHostName());//61.135.253.15；若這個IP地址不存在或域名不能解析，則返回IP地址
```

### InetSocketAddress
[Java8 官方API文檔 InetSocketAddress](https://docs.oracle.com/javase/8/docs/api/java/net/InetSocketAddress.html)

- 建構子
	```java
	InetSocketAddress(String hostname, int port)
	InetSocketAddress(InetAddress addr, int port)
	```
- 常用方法
	```java
	InetSocketAddress address = new InetSocketAddress("127.0.0.1", 9999);  
	address = new InetSocketAddress(InetAddress.getByName("127.0.0.1", 9999));  
	System.out.println(address.getAddress());/// 獲取InetAddress，IP地址，127.0.0.1  
	System.out.println(address.getHostName());// 獲取計算機名，activate.adobe.com  
	System.out.println(address.getPort());// 獲取端口號，9999
	```

### URL
[Java8 官方API文檔 URL](https://docs.oracle.com/javase/8/docs/api/java/net/URL.html)
URI(Uniform Resource Identifier)：統一資源標識符，用來唯一的標識一個資源
URL(Uniform Resource Locator)：統一資源定位器，它是一種具體的URI
- 組成：
	1. 協議
	2. 存放資源的主機域名
	3. 端口(默認:80)
	4. 資源名(端口之後，相對於主機的地址)
- 建構子
	```java
	URL(String spec);// 絕對路徑構建
	URL(URL context, String spec);// 相對路徑構建
	```
- 常用方法
	```java
	URL url = new URL("http://www.baidu.com:80/index.html#aa?uname=michong");//絕對路徑構建  
	url = new URL("http://www.baidu.com:80/a/");  
	url = new URL(url, "b.txt");//相對路徑構建  
	System.out.println(url.toString());  
	System.out.println("協議：" + url.getProtocol());//協議：http  
	System.out.println("域名：" + url.getHost());//域名：www.baidu.com  
	System.out.println("端口：" + url.getPort());//端口：80  
	System.out.println("資源：" + url.getFile());//資源：/a/b.txt  
	System.out.println("相對路徑：" + url.getPath());  
	System.out.println("錨點：" + url.getRef());  
	System.out.println("參數：" + url.getQuery());//存在錨點為null
	```

#### 獲取網頁資源
IO流的使用可參考[[🍀JavaSE_04_IO]]。
- 使用字節流獲取：不建議使用此方法，會亂碼。
	```java
	URL url = new URL("http://www.baidu.com");//主頁默認資源index.html
	//獲取資源網絡流，會亂碼
	InputStream is = url.openStream();
	byte[] flush = new byte[1024];
	int len = 0;
	while(-1 != (len=is.read(flush))){
		System.out.println(new String(flush, 0, len));
		is.colse();
	}
	```
- 使用字符流獲取：建議使用的方法。
	```java
	//轉換流，網路爬蟲(下載資源)
	BufferedReader br = new BufferedReader(new InputStreamReader(url.openStream(), "utf-8"));//解碼字符集
	BrfferedWriter bw = new BufferedWriter(new OutputStream(new FileOutputStream("baidu.html", "utf-8")));//指定轉出的文件
	String msg = null;
	while((msg = br.readLine()) != null){
		System.out.println(msg);
		bw.append(msg);
		bw.newLine();
	}
	bw.flash;
	bw.close();
	br.close();
	```
- 字節數組的輸出
	```java
	//字節數據 + Data輸出流
	//DataOutputStream数据输出流 将java基本数据类型写入数据输出流中。并可以通过数据输入流DataInputStream将数据读入。
	public static byte[] convert(double num) throws IOException{
		byte[] data = null;
		ByteArrayOutputStream bos = new ByteArrayOutputStream();
		DateOutputStream dos = new DataOutputStream(bos);
		dos.writeDouble(num);
		dos.flush();
		//獲取數據
		data = data = bos.toByteArray();
		dos.close();
		return data;
	}
	```
- 字節數組的輸入
	```java
	//字節數組 + Data 輸入流
	public static double convert(byte[] data) throws IOEception{
		DataInputStream dis = new DataInputStream(new ByteArrayInputStream(data));
		double num = dis.readDouble();
		dis.close();
		return num;
	}
	```

## ☎️UDP
以數據為中心，非面向連接，不安全，數據可能丟失，效率高
DatagramSocket，相當於是碼頭
DatagramPacket，相當於是發送和接收數據的集裝箱

### 客戶端
```java
// 1. 創建客戶端：DatagramSocket類 + 指定端口
DatagramSocket client = new DatagramSocket(6666);
// 2. 準備數據：字節數組
String msg = "USG編程";
byte[] data = msg.getBytes();
// 3. 打包：DatagramPacket + 服務器地址及端口
//DatagramPacket(byte[] buf, int length, InetAddress, int port)
DatagramPacket packet = new DatagramPacket(data, data.length, new InetSocketAddress("localhost", 8888));
// 4. 發送
client.send(packet);
// 5. 釋放資源
client.close();
```

### 服務器端
```java
// 1. 創建服務端：DatagramSocket類 + 指定端口
DatagramSocket server = new DatagramSocket(8888);
// 2. 準備接收容器：字節數組，封裝DatagramPacket
byte[] container = new byte[1024];
// 3. 封裝成包，DatagramPacket(byte[] buf, int length)
DatagramPacket packet = new DatagramPacket(container, container.length);
// 4. 接受數據
server.receive(packet);
// 5. 分析數據
byte[] data = packet.getData();
int len = packet.getLength();
System.out.println(new String(data, 0, len));
// 6. 釋放資源
server.close();
```

## ☎️TCP
面向連接，安全、可靠、效率低，類似於打電話
![Java Socket_02_TCP](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%E2%98%8E%EF%B8%8FJava%20Socket/images/Java%20Socket_TCP.png?raw=true)

1.  面向連接：請求Request-響應Response
2.  Socket編程
	- 服務器(指定端口號)：ServerSocket([Java8 官方API文檔 Socket](https://docs.oracle.com/javase/8/docs/api/java/net/ServerSocket.html))
	- 客戶端(端口號自動分配)：Socket([Java8 官方API文檔 Socket](https://docs.oracle.com/javase/8/docs/api/java/net/Socket.html))
3. 聊天室Socket實現原理
	- 客戶端可以發送數據 + 接收數據，並且每個客戶端相互獨立的
	- 為每個客戶端建立線程
	- 輸入流和輸出流應彼此獨立(多個線程)

### 客戶端
```java
//1. 創建客戶端，必須指定服務器+端口，此時就在連接
//Socket (String host, int port)
Socket client = new Socket("localhost", 8888);
//2. 接收數據 + 發送數據
/*
//輸入流，使用BufferedReader
BufferedReader br = new BufferedReader(new BufferedReader(new InputStreamReader(client.getInputStream())));
String echo = br.readLine();//阻塞式方法，無行結束符會出錯
System.out.println(echo);
*/
DataInputStream dis = new DataInputStream(client.getInputStream());
String echo = dis.readUTF();
System.out.println(echo);
```

### 服務器端
```java
//1. 創建服務器，指定端口，ServerSocket(int port)
ServerSocket server = new ServerSocket(8888);//localhost:8888
//2. 接收客戶端連接，阻塞式
while(true) {//為了可以接收多個客戶端，一個accept()一個客戶端
	Socket socket = server.accept();//accept()偵園並接受到此套接字的連接，返回Socket
	System.out.println("客戶端建立連接");
	//3. 發送數據 + 接收數據
	String msg = "歡迎使用";
	/*
	//輸出流，使用BufferedWriter
	BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
	bw.write(msg);
	bw.newLine();//強制加上行結束符，BufferedWriter和BufferedReader傳遞時要注意
	bw.flush();
	*/
	DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
	dos.writeUTF(msg);
	dos.flush();
}
```


