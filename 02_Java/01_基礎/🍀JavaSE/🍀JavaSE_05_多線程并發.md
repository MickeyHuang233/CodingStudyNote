# JavaSE_05_多線程并發
[Java 编程要点之并发（Concurrency）详解](https://waylau.com/essential-java-concurrency/)
![JavaSE_05_多線程并發_總覽](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%8D%80JavaSE/images/JavaSE_05_%E5%A4%9A%E7%B7%9A%E7%A8%8B%E5%B9%B6%E7%99%BC_%E7%B8%BD%E8%A6%BD.png?raw=true)

* 概念
	1. 进程(Processes)：进程有一个独立的执行环境。进程通常有一个完整的、私人的基本运行时资源；特别是，每个进程都有其自己的内存空间。
	2. 线程(Threads)：线程有时被称为轻量级进程。进程和线程都提供一个执行环境，但创建一个新的线程比创建一个新的进程需要更少的资源。
* 特點：
	1. 线程中存在于进程中，每个进程都至少一个线程。
	2. 线程共享进程的资源，包括内存和打开的文件。这使得工作变得高效，但也存在通信的问题。

## 🍀Runnable
`java.lang`
[Java8 官方API文檔 Runnable](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)
Runnable為接口，繼承實現`run()`定義執行緒功能。
```java
public class RunnableImpl01 implements Runnable {
	@Override
	public void run() {//當線程執行時要做的動作
    	for (int c = 0; c < 20; c++) {
             System.out.println(Thread.currentThread().getName()+ ":" + c);//取得當前線程名
    	}
	}
}
```

## 🍀Thread
`java.lang`
[Java8 官方API文檔 Thread](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html)

### 建立線程
要先實作一個Runnable的實體類，並實作`run()`。
```java
public class ThreadDemo {
	public static void main(String args[]) {
    	RunnableImpl01 ri = new RunnableImpl01();
    	Thread t = new Thread(ri, "ThreadName01");//這樣直接建立線程實例，比較沒靈活性
   	Thread t2 = new Thread(ri);
    	t.start();//進入Java排程器
    	t2.start();
 
    	for (int c = 0; c < 20; c++) {
        	System.out.println("main: " + c);
        	if(c == 5){
            	try {
                    t.join();
            	} catch (InterruptedException ex) {
                    Logger.getLogger(ThreadDemo.class.getName()).log(Level.SEVERE, null, ex);
            	}
        	}
    	}
	}
}
```

### 常用方法
|Method|Description|
|---|---|
|public void start()|使该线程开始执行；Java 虚拟机调用该线程的 run 方法。|
|public void run()|如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，该方法不执行任何操作并返回。|
|public void yield()|先讓給別的Thread執行|
|public final void setName(String name)|改变线程名称，使之与参数 name 相同。|
|public final void setPriority(int priority)| 更改线程的优先级。|
|public final void setDaemon(boolean on)|将该线程标记为守护线程或用户线程。|
|public final void join(long millisec)|等待该线程终止的时间最长为 millis 毫秒。|
|public void interrupt()|中断线程。|
|public final boolean isAlive()|测试线程是否处于活动状态。|

* `join()`：呼叫ThreadA.join()的執行緒會等到ThreadA結束後,才能繼續執行
	```java
	public class JoinExample extends Thread {
		String myId;
		public JoinExample(String id) {
			myId = id;
		}
		@Override
		public void run() { // overwrite Thread's run()
	for (int i=0; i < 500; i++) {
				System.out.println(myId+" Thread");
			}
		}

		public static void main(String[] argv) {
			Thread t1 = new JoinExample("T1"); // 產生Thread物件
			Thread t2 = new JoinExample("T2"); // 產生Thread物件
			t1.start(); // 開始執行t1.run()
			t2.start();
			try {
				t1.join(); // 等待t1結束，才執行下一行
				t2.join(); // 等待t2結束，才執行下一行
			} catch (InterruptedException e) {}
			for (int i=0;i < 5; i++) {
				System.out.println("Main Thread");
			}
		}
	}
	```

* `Thread.sleep(int time)`：休息time mini second(1/1000秒)
```java
public class TestSleep {
  public static void main(String[] args) {
       MyThread2 t1 = new MyThread2("TestSleep");
       t1.start();
       for(int i=0 ; i<10; i++){
           System.out.println("I am Main Thread");
       }
}

class MyThread2 extends Thread {
     MyThread2(String s) {
        super(s);
     }
  public void run() {
     for(int i = 1; i <= 10; i++) {
         System.out.println("I am "+getName());
         try {
            sleep(1000); //暂停，每一秒输出一次, 或用Thread.sleep(1000)也可以
         }catch (InterruptedException e) {
            return;
         }
     }
   }
}
```

* `wait()`：等待執行
```java
public synchronized void withdraw(int amount) {
    while (balance < amount) {
            try {
                 wait();//等待執行	
            }
            catch (InterruptedException e) {//處理非預期的結束
                  System.out.println(e.toString());
            }
    }
    	balance -= amount;
    	String name = Thread.currentThread().getName();
        System.out.printf("%s\t\t%d\t%d%n", name, amount, balance);
}
```

### 常用靜態方法

|Static Method|Description|
|---|---|
|public static void yield()|暂停当前正在执行的线程对象，并执行其他线程。|
|public static void sleep(long millisec)|在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。|
|public static boolean holdsLock(Object x)|当且仅当当前线程在指定的对象上保持监视器锁时，才返回 true。|
|public static Thread currentThread()|返回对当前正在执行的线程对象的引用。|
|Thread.currentThread().getName()|目前執行序的名稱|
|public static void dumpStack()|将当前线程的堆栈跟踪打印至标准错误流。|

## 🍀線程鎖 Synchronized
* 定義：Synchronized使用時，需指定一個鎖定物件，當程式進入Synchronized區塊或方法時，該物件會被鎖定，直到離開Synchronized區塊時才會被釋放，讓下一個使用。
* 無論用什麼方法都要注意死結deadlock(每個執行緒都在等存取權)。
* 使用方法

	```java
	private synchronized void methodName() {
	// do something
	}
	```
* 高并發的處理可參考：[[👩‍👩‍👧‍👦Java高并發編程_01_基本使用]]

### synchronized keyWord
sample：
```java
synchronized(this){// 將自己設定為鎖定的物件
// do something
}
```

```java
public class RunnableImpl02 implements Runnable {
	private int c;
	@Override
	public void run() {
    	//r取得使用權才可執行的部分
    	synchronized(this){
        	for (c = 0; c < 50; c++) {
            System.out.println(Thread.currentThread().getName()
                	+ " - " + c);
        	}
    	}
	}
}

public class ThreadDemo07 {
 	public static void main(String args[]) throws InterruptedException {
    	RunnableImpl02 ri = new RunnableImpl02();
    	Thread t = new Thread(ri);
    	Thread t2 = new Thread(ri);
    	t.start();
    	t2.start();
	}
}
```

```java
synchronized method
public class RunnableImpl04 implements Runnable {
	private int c;
	@Override
	public void run() {
    	show();
	}
 
	private synchronized void show() {//確保這個方法只會有一個物件使用
    	for (c = 0; c < 10; c++) {
            System.out.println(Thread.currentThread().getName()
                	+ " - " + c);
    	}
	}
}
 
public class ThreadDemo09 {
	public static void main(String args[]) {
    	RunnableImpl04 ri = new RunnableImpl04();
    	Thread t = new Thread(ri);
    	Thread t2 = new Thread(ri);
    	t.start();
    	t2.start();
	}
}
```
