# NIO_00_Overview
Java NIO(Non Blocking IO)是從Java 1.4引入的IO API，可替代標準的IO操作([[🍀JavaSE_04_IO]])，以更高效的方式進行文件的讀寫操作。

## 🍀概述
### Blocking I/O
- 阻塞IO，Blocking I/O(BIO)
	- 通常在執行`read()`或`write()`的IO操作時，代碼會阻塞直到關閉資源才可繼續讀/寫，可參考：[[🍀JavaSE_04_IO]]
	- 傳統的Server/Client模式，Server會為每個Client端建立線程處理請求，此時會消耗大量資源，若使用線程池又會有最大線程數量的限制，可參考：[[☎️Java Socket]]
- JDK1.4前唯一選擇，適用於連接數比較小且固定的架構，對服務器資源要求較高，但程序簡單易理解

![NIO_00_Overview_01_傳統Server／Client模式](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_00_Overview_01_%E5%82%B3%E7%B5%B1Server%EF%BC%8FClient%E6%A8%A1%E5%BC%8F.png?raw=true)

### Non-Block I/O
- 非阻塞IO(NIO)：基於Reactor模式，執行IO時不會阻塞，而是注冊特定的IO事件(可一次監聽多個通道)，若發生已注冊的事件時才會收到通知
- NIO的本質是延遲的IO操作，真正發生IO時才執行，沒有等待時間
- JDK1.4後支持，適用於連接數較多、連接時間短的架構，如：聊天服務器、彈幕系統、服務器通訊…等

![NIO_00_Overview_02_非阻塞IO架構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%8D%80NIO/images/NIO_00_Overview_02_%E9%9D%9E%E9%98%BB%E5%A1%9EIO%E6%9E%B6%E6%A7%8B.png?raw=true)

### Asynchronous I/O
- [Java8 官方API文檔 AsynchronousChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/AsynchronousChannel.html)
- 異步非阻塞IO(AIO)，相當於是NIO 2.0，客戶端的IO請求都是由OS先完成後再通知Server應用去啟動線程處理
- JDK1.7後支持，適用於連接數較多、連接時間較長的應用，但程序較為複雜，如：相冊服務器…等
- 包含以下異步通道
	1. `AsynchronousSocketChannel`
	2. `AsynchronousServerSocketChannel`
	3. `AsynchronousFileChannel`
	4. `AsynchronousDatagramChannel`

| BIO            | NIO                   | AIO                               |
| -------------- | --------------------- | --------------------------------- |
| `Socket`       | `SocketChannel`       | `AsynchronousSocketChannel`       |
| `ServerSocket` | `ServerSocketChannel` | `AsynchronousServerSocketChannel` | 