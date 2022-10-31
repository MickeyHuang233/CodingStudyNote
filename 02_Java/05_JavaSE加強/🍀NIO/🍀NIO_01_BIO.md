# NIO_01_BIO
## 🍀傳統BIO
- 可參考[[☎️Java Socket#☎️TCP]]代碼實現

### 傳統客戶端
- 連線流程
	1. 通過`Socket`請求與服務端連線
	2. 從`Socket`中得到字節輸入/輸出進行數據讀寫操作
- 缺點：僅能接收一個Socket請求

```java
@Test
@DisplayName("Client測試_多發")
public void test02() {
	System.out.println("Start Client");
	try {
		// 建立Socket，請求服務連接
		Socket socket = new Socket("127.0.0.1", 8888);
		// 取得Socket取得字節輸出流
		OutputStream outputStream = socket.getOutputStream();
		// 將字節輸出流包裝為打印流
		PrintStream printStream = new PrintStream(outputStream);
		Scanner scanner = new Scanner(System.in);
		while(true) {
			System.out.print("send message : ");
			String nextLine = scanner.nextLine();
//				printStream.print(nextLine); // print()，此行信息並未結束
			printStream.println(nextLine); // println()，此行信息換行結束
			printStream.flush();
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

### 傳統服務端_接收單個客戶端
![NIO_01_BIO_01_服務端接收單個客戶端架構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_01_BIO_01_%E6%9C%8D%E5%8B%99%E7%AB%AF%E6%8E%A5%E6%94%B6%E5%96%AE%E5%80%8B%E5%AE%A2%E6%88%B6%E7%AB%AF%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true)

- 連線流程
	1. 通過`ServerSocket`注冊端口
	2. 調用`accept()`監聽客戶端Socket請求
	3. 從`Socket`中取得字節輸入/輸出流進行數據讀寫操作

```java
@Test
@DisplayName("Server測試_多收")
public void test01() {
	// 如果要接收多個Client消息，需要使用多線程
	System.out.println("Start Server");
	try {
		// 定義ServerSocket注冊端口
		ServerSocket serverSocket = new ServerSocket(8888);
		// 監聽Socket請求
		Socket socket = serverSocket.accept();
		// 從Socket取得字節輸入流
		InputStream inputStream = socket.getInputStream();
		// 將字節輸入流包裝為緩沖字節輸入流
		BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
		// 處理接收的信息
		String message;
		while ((message = bufferedReader.readLine()) != null) {
			System.out.println("Server get message : " + message);
		}
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

### 傳統服務端_接收多個客戶端
![NIO_01_BIO_02_服務端接收多個客戶端架構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_01_BIO_02_%E6%9C%8D%E5%8B%99%E7%AB%AF%E6%8E%A5%E6%94%B6%E5%A4%9A%E5%80%8B%E5%AE%A2%E6%88%B6%E7%AB%AF%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true)

- 連線流程
	1. 建立線程任務類
		```java
		public class T07_ServerThread extends Thread {
			private Socket socket;
		
			public T07_ServerThread(Socket socket) {
				super();
				this.socket = socket;
			}
		
			@Override
			public void run() {
				try {
					// 從Socket取得字節輸入流
					InputStream inputStream = socket.getInputStream();
					// 將字節輸入流包裝為緩沖字節輸入流
					BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
					// 處理接收的信息
					String message;
					while ((message = bufferedReader.readLine()) != null) {
						System.out.println("Server get message : " + this.getName() + " - " + message);
					}
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		```
	2. 服務端接收到一個Socket請求，就交給獨立的線程
		```java
		@Test
		@DisplayName("Server測試_接收多客戶端")
		public void test03() {
			System.out.println("Start Server");
			try {
				// 定義ServerSocket注冊端口
				ServerSocket serverSocket = new ServerSocket(8888);
				// 持繼監聽Socket請求，新建線程處理
				while(true) {
					Socket socket = serverSocket.accept();
					new T07_ServerThread(socket).start();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		```
- 缺點
	1. 每個Socket會建立一個線程，線程的竟爭、切換上下文影響性能
	2. 每個線程都會占用棧空間、CPU資源
	3. 不是所有線程都有IO操作，可能浪費線程資源
	4. 訪問量越大越容易發生棧出，導致服務器當機

## 🍀偽異步IO
![NIO_01_BIO_03_服務端偽異步架構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_01_BIO_03_%E6%9C%8D%E5%8B%99%E7%AB%AF%E5%81%BD%E7%95%B0%E6%AD%A5%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true)

- 主要通過線程池、任務隊列實現
- 優點：資源可控
- 缺點：並發能力受限，無法從根本解決問題(阻塞)
- 連線流程
	1. 建立線程任務類
		```java
		public class T11_ServerThread implements Runnable {
			private Socket socket;
		
			public T11_ServerThread(Socket socket) {
				super();
				this.socket = socket;
			}
		
			@Override
			public void run() {
				try {
					// 從Socket取得字節輸入流
					InputStream inputStream = socket.getInputStream();
					// 將字節輸入流包裝為緩沖字節輸入流
					BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
					// 處理接收的信息
					String message;
					while ((message = bufferedReader.readLine()) != null) {
						long threadId = Thread.currentThread().getId();
						System.out.println("Server get message : " + threadId + " - " + message);
					}
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		```
	2. 建立線程池類
		```java
		public class T11_ServerThreadPool {
			// 線程池
			private ExecutorService executorService;
		
			/**
			 * 初始化線程池對象
			 * 
			 * @param maxThreadSize    線程池中線程數量
			 * @param messageQueueSize 任務隊列數量
			 */
			public T11_ServerThreadPool(int maxThreadSize, int messageQueueSize) {
				this.executorService = new ThreadPoolExecutor( //
						4, // corePoolSize，最小線程數
						maxThreadSize, // maximumPoolSize，最大線程數
						2, // keepAliveTime，過期時間
						TimeUnit.MINUTES, // unit，單位
						new ArrayBlockingQueue<>(messageQueueSize)); // workQueue，任務隊列
			}
		
			/**
			 * 將任務給線程池執行
			 * 
			 * @param runnable 任務
			 */
			public void doExecute(Runnable runnable) {
				this.executorService.execute(runnable);
			}
		}
		```
	3. 服務端接收到一個Socket請求，就交給線程池中的任務隊列
		```java
		@Test
		@DisplayName("Server測試_偽異步")
		public void test04() {
			System.out.println("Start Server");
			try {
				// 定義ServerSocket注冊端口
				ServerSocket serverSocket = new ServerSocket(8888);
				// 初始化線程池
				T11_ServerThreadPool serverThreadPool = new T11_ServerThreadPool(5, 10);
				// 持繼監聽Socket請求，交由線程池處理
				while(true) {
					Socket socket = serverSocket.accept();
					T11_ServerThread serverThread = new T11_ServerThread(socket);
					serverThreadPool.doExecute(serverThread);
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		```

## 🍀文件上傳
### 客戶端文件上傳
發送任意文件類型檔案

```java
@Test
@DisplayName("Client測試_文件上傳")
public void test05() {
	System.out.println("Start Client");
	File uploadFile = new File("D:\\2021091915263900_s.jpg");
	try ( //
			InputStream fileInputStream = new FileInputStream(uploadFile); //
	) {
		// 建立Socket，請求服務連接
		Socket socket = new Socket("127.0.0.1", 8888);
		// 將字節輸出流包裝為數據輸出流
		DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
		// 發送文件類型給服務端
		String filelName = uploadFile.getName();
		int fileTypeIndex = filelName.lastIndexOf(".");
		String fileType = "";
		if (fileTypeIndex != -1) {
			fileType = filelName.substring(fileTypeIndex);
		}
		dataOutputStream.writeUTF(fileType);
		// 發送文件數據
		byte[] buffer = new byte[1024];
		int len;
		while ((len = fileInputStream.read(buffer)) > 0) {
			dataOutputStream.write(buffer, 0, len);
		}
		dataOutputStream.flush();
		// 通知服務端發送完畢
		socket.shutdownOutput();
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

### 服務端文件接收
接收任意文件類型檔案

1. 建立線程任務類
	```java
	public class T14_ServerThread extends Thread {
	    private Socket socket;
	
	    public T14_ServerThread(Socket socket) {
	        this.socket = socket;
	    }
	
	    @Override
	    public void run() {
	        try {
	            // 從Socket取得字節輸入流
	            DataInputStream dataInputStream = new DataInputStream(this.socket.getInputStream());
	            // 取得文件接收類型
	            String fileType = dataInputStream.readUTF();
	            System.out.println("get file type : " + fileType);
	            // 將接收的文件輸出至本機
	            OutputStream fileOutputStream = new FileOutputStream("D:\\server\\" + UUID.randomUUID().toString() + fileType);
	            byte[] buffer = new byte[1024];
	            int len;
	            while ((len = dataInputStream.read(buffer)) > 0) {
	                fileOutputStream.write(buffer, 0, len);
	            }
	            fileOutputStream.close();
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	}
	```
2. 服務端接收到一個Socket請求，就交給獨立的線程
	```java
	@Test
	@DisplayName("Server測試_文件接收")
	public void test06() {
		System.out.println("Start Server");
		try {
			// 定義ServerSocket注冊端口
			ServerSocket serverSocket = new ServerSocket(8888);
			// 持繼監聽Socket請求，新建線程處理
			while (true) {
				Socket socket = serverSocket.accept();
				new T14_ServerThread(socket).start();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	```

## 🍀端口轉發
客戶端消息發送給所有的客戶端接收
![NIO_01_BIO_04_端口轉發架構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_01_BIO_04_%E7%AB%AF%E5%8F%A3%E8%BD%89%E7%99%BC%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true)

### 服務端
1. 建立線程任務類
	```java
	public class T04_SocketTest {
	    // 所有已連線Socket列表
	    public static List<Socket> socketList = new ArrayList<Socket>();
	
	    @Test
	    @DisplayName("Server測試_瑞口轉發")
	    public void test07() {
	        System.out.println("Start Server");
	        try {
	            ServerSocket serverSocket = new ServerSocket(8888);
	            // 持繼監聽Socket請求，新建線程處理
	            while (true) {
	                Socket socket = serverSocket.accept();
	                new T17_ServerThread(socket).start();
	                // 將已連線Socket加入集合
	                socketList.add(socket);
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}
	```
2. 服務端接收到一個Socket請求，就交給獨立的線程
	```java
	public class T17_ServerThread extends Thread {
	    private Socket socket;
	
	    public T17_ServerThread(Socket socket) {
	        super();
	        this.socket = socket;
	    }
	
	    @Override
	    public void run() {
	        try {
	            // Socket取得輸入流，並包裝成緩沖輸入流
	            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
	            // 取得消息內容
	            String message;
	            while ((message = bufferedReader.readLine()) != null) {
	                // 將消息推送至所有客戶端
	                this.sendMessageToAllSocket(message);
	            }
	        } catch (IOException e) {
	            // 當前有socket下線
	            T04_SocketTest.socketList.remove(socket);
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	
	    /**
	     * 將消息推送至所有客戶端
	     * 
	     * @param message 消息內容
	     * @throws IOException 
	     */
	    private void sendMessageToAllSocket(String message) throws IOException {
	        for (Socket socket : T04_SocketTest.socketList) {
	            PrintStream printStream = new PrintStream(socket.getOutputStream());
	            printStream.println(message);
	            printStream.flush();
	        }
	    }
	}
	```

### 客戶端
1. 建立線程任務類，用於接收其他Socket消息
	```java
	public class T19_ClientThread extends Thread{
	    private Socket socket;
	
	    public T19_ClientThread(Socket socket) {
	        super();
	        this.socket = socket;
	    }
	
	    @Override
	    public void run() {
	        try {
	            while (true) {
	                // 將字節輸入流包裝為緩沖字節輸入流
	                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
	                // 處理接收的信息
	                String message;
	                while ((message = bufferedReader.readLine()) != null) {
	                    System.out.println("\nGet message : " + message);
	                    System.out.print("send message : ");
	                }
	            }
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }
	}
	```
2. 客戶端主線程用於發送消息，獨立的線程接收消息
	```java
	@Test
	@DisplayName("Client測試_邊發邊收")
	public void test08() {
		System.out.println("Start Client");
		try {
			// 建立Socket，請求服務連接
			Socket socket = new Socket("127.0.0.1", 8888);
			// 取得Socket取得字節輸出流
			OutputStream outputStream = socket.getOutputStream();
			// 新建線程，監聽消息
			new T19_ClientThread(socket).start();
			// 將字節輸出流包裝為打印流
			PrintStream printStream = new PrintStream(outputStream);
			Scanner scanner = new Scanner(System.in);
			while (true) {
				System.out.print("send message : ");
				String nextLine = scanner.nextLine();
				printStream.println(nextLine);
				printStream.flush();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	```

