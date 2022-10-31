# NIO_02_NIO
![NIO_02_NIO_01_核心關系圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_02_NIO_01_%E6%A0%B8%E5%BF%83%E9%97%9C%E7%B3%BB%E5%9C%96.png?raw=true)

## 🍀緩沖 Buffer
- [Java8 官方API文檔 Buffer](https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html)
- 用於寫入數據、讀取數據的內存，常用的`Buffer`子類：
	- `ByteBuffer`
	- `CharBuffer`
	- `ShortBuffer`
	- `IntBuffer`
	- `LongBuffer`
	- `FloatBuffer`
	- `DoubleBuffer`
- 基本屬性
	![NIO_02_NIO_02_Buffer基本屬性](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_02_NIO_02_Buffer%E5%9F%BA%E6%9C%AC%E5%B1%AC%E6%80%A7.png?raw=true)
	- 容量(capacity)：Buffer是具有固定大小的內存塊，創建後不能更改
	- 限制(limit)：Buffer可操作的數據大小，limit後的數據不能進讀寫，limit <= capacity
		- 寫模式，limit = capacity
		- 讀模式，limit = 寫入的數據量
	- 位置(position)：下個要讀寫的數據索引，position <= limit
	- 標記(mark)：指定Buffer中一個特定的position，mark <= position
- 使用基本步驟
	1. 寫入數據至Buffer 
	2. 調用`filp()`，轉換為讀模式
	3. 從Buffer讀取數據
	4. 調用`clear()`或`compact()`，清除緩沖區

### 常用API
```java
@Test
@DisplayName("NIO_Buffer常用API")
public void test01() throws Exception {
	// 建立指定容量Buffer
	ByteBuffer byteBuffer = ByteBuffer.allocate(10);
	this.printBufferVar(byteBuffer);
	// 在Buffer添加數據
	byteBuffer.put("Hello".getBytes());
	this.printBufferVar(byteBuffer);
	// 將Buffer limit設置為當前位置，並將position設置為0
	byteBuffer.flip();
	this.printBufferVar(byteBuffer);
	// 讀取數據
	System.out.println((char) byteBuffer.get()); // 讀取1個字節，H
	byte[] b = new byte[2];
	byteBuffer.get(b); // 讀取多個字節，el
	System.out.println(new String(b));
	this.printBufferVar(byteBuffer);
	// 標記position
	byteBuffer.mark();
	System.out.println((char) byteBuffer.get()); // l
	// 返回標記position
	byteBuffer.reset();
	// position後是否還有數據
	if (byteBuffer.hasRemaining()) {
		// 取得未讀取數據數
		System.out.println("byteBuffer.remaining() : " + byteBuffer.remaining());
	}
	// 清除緩沖區數據，但實際上只是將position設置為0，添加數據時才會覆蓋原數據
	byteBuffer.clear();
	System.out.println((char) byteBuffer.get());
	this.printBufferVar(byteBuffer);
}

// 查看基本屬性
private void printBufferVar(ByteBuffer byteBuffer) {
	System.out.println("byteBuffer.capacity() : " + byteBuffer.capacity());
	System.out.println("byteBuffer.limit() : " + byteBuffer.limit());
	System.out.println("byteBuffer.position() : " + byteBuffer.position());
	System.out.println("---------");
}
```

### 直接緩沖區
- 直接內存，也就是非堆內存
- 因為直接內存是直接用於系統IO操作，所以申請使用時需要消費更高性能，但在JVM中使用有更高的性能
- 申請的直接緩沖區是在JVM外的內存，因此不會占用應用內存，如果需要緩存生命周期長的大數據，或是頻繁的IO操作時，比較適合用直接緩沖區
- 可使用`isDiret()`確定字節緩沖區是直接緩沖區還是非直接綏沖區
	```java
	@Test
	@DisplayName("直接緩沖區")
	public void test02() {
		// 建立直接緩沖區
		ByteBuffer byteBuffer = ByteBuffer.allocateDirect(10);
		// 緩沖區是否為直接緩沖區
		System.out.println(byteBuffer.isDirect());
	}
	```

### 非直接緩沖區
- 非直接內存，堆內存
- 非直接內存執行IO操作時，會覺從本進程內存複製至直接內存，再利用本機IO處理
- 一般來說使用非直接緩沖區
	```java
	@Test
	@DisplayName("非直接緩沖區")
	public void test03() {
		// 建立非直接緩沖區
		ByteBuffer byteBuffer = ByteBuffer.allocate(10);
		// 緩沖區是否為直接緩沖區
		System.out.println(byteBuffer.isDirect());
	}
	```

## 🍀通道 Channel
- [Java8 官方API文檔 Channel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/package-summary.html)
- Channel通道，Channel和IO中的Stream差不多，只是Stream是單向的，Channel是雙向的，可同時通過Buffer執行讀、寫操作
	- Channel數據源可用於文件、網絡Socket
	- Channel可以異步讀寫
	- 可通過Channel讀取、寫入數據，但實際的讀寫操作是Buffer處理
- Channel實現類型
	```mermaid
	graph BT
	AbstractSelectableChannel-->SelectableChannle
	ServerSocketChannel-->AbstractSelectableChannel
	SocketChannel-->AbstractSelectableChannel
	DatagramChannel-->AbstractSelectableChannel
	FileChannel-->AbstractInterruptibleChannel
	```
	1. `FileChannel`，從文件讀寫數據
	2. `DatagramChannel`，通過UDP讀寫網絡中的數據
	3. `SocketChannel`，通過TCP讀寫網絡中的數據
	4. `ServerSocketChannel`，監聽新進來的TCP連接，像Web服務器對每個連接創建`SocketChannel`

### FileChannel
- [Java8 官方API文檔 FileChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/FileChannel.html)

#### 讀操作
```java
@Test
@DisplayName("FileChannel讀操作")
public void test01() throws IOException {
	// 建立FileChannel
	RandomAccessFile randomAccessFile = new RandomAccessFile( //
			"D:\\note.txt", // 文件路徑
			"rw"); // Channel模式：r，只讀；rw，讀寫；rws，SYNC；rwd，DSYNC
	FileChannel channel = randomAccessFile.getChannel();
	// 另一個方試建立FileChannel
	// FileInputStream fileInputStream = new FileInputStream("D:\\test.txt");
	// FileChannel channel = fileInputStream.getChannel();
	// 建立Buffer
	ByteBuffer buffer = ByteBuffer.allocate(1024);
	// 讀取數據至緩沖區
	int bytesRead = channel.read(buffer); // 讀取字節數
	while (bytesRead != -1) { // bytesRead為-1表示到文件未尾
		// 初始化position
		buffer.flip();
		System.out.println(new String(buffer.array()));
		// 清空緩存
		buffer.clear();
		// 讀取下一行
		bytesRead = channel.read(buffer);
	}
	// 關閉channel
	channel.close();
	randomAccessFile.close();
}
```

#### 寫操作
```java
@Test
@DisplayName("FileChannel寫操作")
public void test02() throws IOException {
	// 建立FileChannel
	RandomAccessFile randomAccessFile = new RandomAccessFile("D:\\note_01.txt", "rw");
	FileChannel channel = randomAccessFile.getChannel();
	// 另一個方試建立FileChannel
	// FileOutputStream fileOutputStream = new FileOutputStream("D:\\note.txt");
	// FileChannel channel = fileOutputStream.getChannel(); //
	// 建立Buffer
	ByteBuffer buffer = ByteBuffer.allocate(1024);
	buffer.clear();
	// 寫入數據至緩沖區
	buffer.put("mickey study nio fileChannel".getBytes());
	// 初始化position
	buffer.flip();
	// 寫入文件
	channel.write(buffer);
	// 關閉channel
	channel.close();
	randomAccessFile.close();
}
```

#### 文件複製
```java
@Test
@DisplayName("FileChannel文件複製")
public void test03() throws IOException {
	try (
			// 建立通道
			FileInputStream fileInputStream = new FileInputStream("D:\\test.jpg");
			FileChannel inputChannel = fileInputStream.getChannel();
			// 建立輸出流通道
			FileOutputStream fileOutputStream = new FileOutputStream("D:\\test_01.jpg");
			FileChannel outputChannel = fileOutputStream.getChannel(); //
	) {
		// 建立緩沖區
		ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
		// 執行數據傳送
		while (true) {
			byteBuffer.clear(); // 先清空數據再寫入緩沖區
			int bytesRead = inputChannel.read(byteBuffer);
			// 無讀取到數據
			if (bytesRead == -1) {
				break;
			}
			// 初始化position
			byteBuffer.flip();
			// 緩沖區輸出
			outputChannel.write(byteBuffer);
		}
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

#### 其他方法
```java
@Test
@DisplayName("FileChannel其他方法")
public void test03() throws IOException {
	RandomAccessFile randomAccessFile = new RandomAccessFile("D:\\note_01.txt", "rw");
	FileChannel channel = randomAccessFile.getChannel();
	ByteBuffer buffer = ByteBuffer.allocate(1024);
	
	// 取得當前FileChannel位置
	System.out.println("channel.position() : " + channel.position());
	// 設置FileChannel位置
	// 注意：若將位置設置於文件結束符後，可能道致磁盤上物理文件中的數據有空隙，也就是"文件空洞"
	channel.position(10);
	buffer.put("(put new message)".getBytes());
	buffer.flip();
	channel.write(buffer);

	// 取得文件的總大小
	System.out.println("channel.size() : " + channel.size());
	
	// 截取文件長度，指定長度後的文字會刪除
	channel.truncate(20);
	
	// 將Channel中尚未寫入磁盤的數據強制寫入，出於性能，操作系統會將數據緩存至內存中
	channel.force(true); // true，表示同時將文件元數據(取限信息…)寫入磁盤

	channel.close();
	randomAccessFile.close();
}
```

#### 分散、聚集
- 分散(Scatter)：將Channel通道的數據讀到多個Buffer中
	```java
	@Test
	@DisplayName("FileChannel分散")
	public void test04() {
		try (
				// 建立通道
				FileInputStream fileInputStream = new FileInputStream("D:\\test.txt");
				FileChannel channel = fileInputStream.getChannel(); //
		) {
			// 建立多個綏沖區
			ByteBuffer[] byteBuffers = { //
					ByteBuffer.allocate(4), //
					ByteBuffer.allocate(1024), //
			};
			// 從通道讀取數據至各緩沖區中
			channel.read(byteBuffers);
			// 打印緩緩區數據
			for (ByteBuffer buffer : byteBuffers) {
				buffer.flip();
				System.out.println(new String(buffer.array(), 0, buffer.remaining()));
				System.out.println("-----");
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	```
- 聚集(Gather)：將多個Buffer數據集中至一個Channel
	```java
	@Test
	@DisplayName("FileChannel聚集")
	public void test05() {
		try (
				// 建立通道
				FileOutputStream fileOutputStream = new FileOutputStream("D:\\test01.txt");
				FileChannel channel = fileOutputStream.getChannel(); //
		) {
			// 建立多個緩沖區
			ByteBuffer[] byteBuffers = { //
					ByteBuffer.allocate(10), //
					ByteBuffer.allocate(20), //
			};
			// 緩沖區添加數據
			byteBuffers[0].put("Hello, ".getBytes());
			byteBuffers[1].put("FileChannel Gather!".getBytes());
			// 重置position
			for (ByteBuffer buffer : byteBuffers) {
				buffer.flip();
			}
			// 緩沖區輸出
			channel.write(byteBuffers);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	```

#### Channel間數據傳輸
- `transferFrom()`，從目標通道複製到原通道數據
	```java
	@Test
	@DisplayName("Channel間數據傳輸_transferFrom")
	public void test04() throws IOException {
		RandomAccessFile fromRandomAccessFile = new RandomAccessFile("D:\\note.txt", "rw");
		RandomAccessFile toRandomAccessFile = new RandomAccessFile("D:\\note_02.txt", "rw");
		FileChannel fromChannel = fromRandomAccessFile.getChannel();
		FileChannel toChannel = toRandomAccessFile.getChannel();
	
		toChannel.transferFrom(fromChannel, 0, fromChannel.size());
	
		fromChannel.close();
		toChannel.close();
	}
	```
- `transferTo()`，從原通道數據複製到目標通道
	```java
	@Test
	@DisplayName("Channel間數據傳輸_transferTo")
	public void test05() throws IOException {
		RandomAccessFile fromRandomAccessFile = new RandomAccessFile("D:\\note.txt", "rw");
		RandomAccessFile toRandomAccessFile = new RandomAccessFile("D:\\note_03.txt", "rw");
		FileChannel fromChannel = fromRandomAccessFile.getChannel();
		FileChannel toChannel = toRandomAccessFile.getChannel();
	
		fromChannel.transferTo(0, fromChannel.size(), toChannel);
	
		fromChannel.close();
		toChannel.close();
	}
	```

### ServerSocketChannel
- [Java8 官方API文檔 ServerSocketChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/ServerSocketChannel.html)
- Socket通道類(`ServerSocketChannel`、`SocketChannel`、`DatagramChannel`)被實例化的時候會創建`java.net`的類(`Socket`、`ServerSocket`、`DatagramSocket`)，因此Socket通道類可重複使用，而Socket類則不行
- Socket通道類可選擇是否阻塞`configureBlocking(boolean block)`，提高靈活度
- `ServerSocketChannel`是基於socket監聽器，並不用於傳輸數據，可用於非組塞狀態下監聽

```java
// 當Client連線至Server時，Server會通過ServerSocketChannel得到SocketChannel
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
// 切換非阻塞模式
serverSocketChannel.configureBlocking(false);
// 指定連線端口
serverSocketChannel.bind(new InetSocketAddress(8888));
```

### SocketChannel
- [Java8 官方API文檔 SocketChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SocketChannel.html)
- Socket通道類(`ServerSocketChannel`、`SocketChannel`、`DatagramChannel`)被實例化的時候會創建`java.net`的類(`Socket`、`ServerSocket`、`DatagramSocket`)，因此Socket通道類可重複使用，而Socket類則不行
- 服務器取得`SockteChannel`
	```java
	// 當Client連線至Server時，Server會通過ServerSocketChannel得到SocketChannel
	ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
	// 切換非阻塞模式
	serverSocketChannel.configureBlocking(false);
	// 指定連線端口
	serverSocketChannel.bind(new InetSocketAddress(8888));
	// 取得接入的客戶端Socket通道
	SocketChannel socketChannel = serverSocketChannel.accept();
	// 切換非阻塞模式
	socketChannel.configureBlocking(false);
	```
- 客戶端取得`SockteChannel`
	```java
	// 取得通道
	SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8888));
	// 切換非阻塞模式
	socketChannel.configureBlocking(false);
	```

### DatagramChannel
- [Java8 官方API文檔 DatagramChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/DatagramChannel.html)

## 🍀選擇器 Selector
- 一個Selector可同時監控多個`SelectableChannel`的IO，是非阻塞IO的核心，也避免多線程間的上下文切換道致的效能開銷
- 可監聽的事件類型
	1. `SelectionKey.OP_READ`，1，讀
	2. `SelectionKey.OP_WRITE`，4，寫
	3. `SelectionKey.OP_CONNECT`，8，連接
	4. `SelectionKey.OP_ACCEPT`，16，接收
	5. 如果注冊時不此監聽一個事件，可以使用"位或"操作符連接

### 服務端
```java
@Test
@DisplayName("Selector_服務端")
public void test01() throws IOException {
	System.out.println("Start Server");
	// 當Client連線至Server時，Server會通過ServerSocketChannel得到SocketChannel
	ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
	// 切換非阻塞模式
	serverSocketChannel.configureBlocking(false);
	// 指定連線端口
	serverSocketChannel.bind(new InetSocketAddress(8888));
	// 取得選擇器
	Selector selector = Selector.open();
	// 將通道注冊至選擇器上，並指定監聽事件
	serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
	// 處理選擇器中已經就緒好的事件
	while (selector.select() > 0) {
		// 取得所有已注冊通道中已經就緒的事件
		Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
		// 遍歷已就緒的事件
		while (iterator.hasNext()) {
			// 取得要處理事件
			SelectionKey selectionKey = iterator.next();
			// 事件為接收事件，相當於就是讀事件
			if (selectionKey.isAcceptable()) {
				// 取得接入的客戶端Socket通道
				SocketChannel socketChannel = serverSocketChannel.accept();
				// 切換非阻塞模式
				socketChannel.configureBlocking(false);
				// 將客戶端Socket通道注冊至選擇器
				socketChannel.register(selector, SelectionKey.OP_READ);
			}
			// 事件為讀取事件
			else if (selectionKey.isReadable()) {
				// 取得要處理Socket通道
				SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
				// 讀取數據
				ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
				int len = 0;
				while ((len = socketChannel.read(byteBuffer)) > 0) {
					byteBuffer.flip();
					System.out.println(new String(byteBuffer.array(), 0, byteBuffer.remaining()));
					byteBuffer.clear();
				}
			}
			// 處理完畢，將教緒好事件移除
			iterator.remove();
		}
	}
}
```

### 客戶端
```java
@Test
@DisplayName("Selector_客戶端")
public void test02() throws IOException {
	System.out.println("Start Client");
	// 取得通道
	SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 8888));
	// 切換非阻塞模式
	socketChannel.configureBlocking(false);
	// 建立緩沖區
	ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
	// 發送數據至Server
	Scanner scanner = new Scanner(System.in);
	while(true) {
		System.out.print("Send Message : ");
		String message = scanner.nextLine();
		byteBuffer.put(message.getBytes());
		byteBuffer.flip();
		socketChannel.write(byteBuffer);
		byteBuffer.clear();
	}
}
```

### 群聊服務端
- 服務端類
	```java
	/*
	 * 群聊服務端
	 */
	public class T44_Server {
		private Selector selector;
		private ServerSocketChannel serverSocketChannel;
		private static final int PORT = 8888;
	
		public T44_Server() {
			try {
				// 建立選擇器
				this.selector = Selector.open();
				// 建立通道
				this.serverSocketChannel = ServerSocketChannel.open();
				// 切換非阻塞模式
				this.serverSocketChannel.configureBlocking(false);
				// 指定連線端口
				this.serverSocketChannel.bind(new InetSocketAddress(PORT));
				// 將通道注冊至選擇器上，並指定監聽事件
				this.serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	
		/*
		 * 開始監聽服務
		 */
		public void listen() {
			try {
				// 處理就緒好的事件
				while (this.selector.select() > 0) {
					// 遍歷已就緒的事件
					Iterator<SelectionKey> iterator = this.selector.selectedKeys().iterator();
					while (iterator.hasNext()) {
						SelectionKey selectionKey = iterator.next();
						// 客戶端初次連線
						if (selectionKey.isAcceptable()) {
							// 取得客戶端Socket通道，並切換為非阻塞模式、注冊至選擇器
							SocketChannel socketChannel = this.serverSocketChannel.accept();
							socketChannel.configureBlocking(false);
							socketChannel.register(selector, SelectionKey.OP_READ);
						}
						// 客戶端發送消息
						else if (selectionKey.isReadable()) {
							// 處理客戶端發送的信息，轉發至所有已連線客戶端
							this.processClientMessage(selectionKey);
						}
						// 移除已處理事件
						iterator.remove();
					}
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	
		/*
		 * 處理客戶端發送的信息，轉發至所有已連線客戶端
		 */
		private void processClientMessage(SelectionKey selectionKey) {
			SocketChannel socketChannel = null;
			try {
				socketChannel = (SocketChannel) selectionKey.channel();
				// 取得客戶端通道信息
				ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
				int readCount = socketChannel.read(byteBuffer);
				if (readCount > 0) { // 有讀取到才處理
					// 初始化緩沖區position
					byteBuffer.flip();
					String message = new String(byteBuffer.array(), 0, byteBuffer.remaining());
					this.sendMessageToAllClient(message, socketChannel);
				}
			} catch (Exception e) {
				// 表示當前客戶端離線，關閉、取消通道連線
				selectionKey.cancel();
				try {
					socketChannel.close();
				} catch (IOException e1) {
					e1.printStackTrace();
				}
			}
		}
	
		/**
		 * 發送消息至所有已連線客戶端
		 * 
		 * @param message 消息內容
		 */
		private void sendMessageToAllClient(String message, SocketChannel curretChannel) {
			System.out.println("send client message : " + Thread.currentThread().getName() + " : " + message);
			try {
				// 取得所有客戶端通道
				for (SelectionKey selectionKey : this.selector.keys()) {
					Channel channel = selectionKey.channel();
					// 排除通道
					if ( //
					channel instanceof SocketChannel && channel == curretChannel // 當前通道
							|| selectionKey.isAcceptable() // 初次連線用的通道
					) {
						continue;
					}
					ByteBuffer byteBuffer = ByteBuffer.wrap(message.getBytes());
					((SocketChannel) channel).write(byteBuffer);
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	```
- 啟動服務端
	```java
	@Test
	@DisplayName("群聊功能_服務端")
	public void test03() {
		System.out.println("Start Server");
		T44_Server server = new T44_Server();
		server.listen();
	}
	```

### 群聊客戶端_多接多發
- 客戶端類
	```java
	/*
	 * 群聊客戶端
	 */
	public class T46_Client {
		private Selector selector;
		private SocketChannel socketChannel;
		private static int PORT = 8888;
	
		public T46_Client() {
			try {
				this.selector = Selector.open();
				this.socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", PORT));
				this.socketChannel.configureBlocking(false);
				this.socketChannel.register(selector, SelectionKey.OP_READ);
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	
		/*
		 * 監聽服務端發送的消息
		 */
		public void readMessageFromServer() {
			try {
				while (this.selector.select() > 0) {
					Iterator<SelectionKey> iterator = this.selector.selectedKeys().iterator();
					while (iterator.hasNext()) {
						SelectionKey selectionKey = iterator.next();
						// 讀取消息
						if (selectionKey.isReadable()) { // 排除其他事件
							SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
							ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
							socketChannel.read(byteBuffer);
							System.out.println("\n" + new String(byteBuffer.array()).trim());
							System.out.print("Send Message : ");
						}
					}
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	
		/*
		 * 發送消息至服務端
		 * 
		 * @param message
		 */
		public void sendMessage(String message) {
			try {
				ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
				byteBuffer.put(message.getBytes());
				byteBuffer.flip();
				this.socketChannel.write(byteBuffer);
				byteBuffer.clear();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	```
- 啟動客戶端
	```java
	@Test
	@DisplayName("群聊功能_客戶端_多接多發")
	public void test04() {
		System.out.println("Start Client");
		T46_Client client = new T46_Client();
		// 定義新線程監聽Server發送的消息
		new Thread(new Runnable() {
			@Override
			public void run() {
				client.readMessageFromServer();
			}
		}).start();
		// 發送數據至Server
		Scanner scanner = new Scanner(System.in);
		while(true) {
			System.out.print("Send Message : ");
			String message = scanner.nextLine();
			client.sendMessage(message);
		}
	}
	```