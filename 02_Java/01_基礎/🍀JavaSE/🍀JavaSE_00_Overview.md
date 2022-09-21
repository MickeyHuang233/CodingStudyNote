# JavaSE_00_Overview
[Java官方文件](https://docs.oracle.com/en/java/javase/index.html)

## 🍀Catalog
- [[🍀JavaSE_01_基礎語法]]
- [[🍀JavaSE_02_面向對象]]
- [[🍀JavaSE_03_集合]]
- [[🍀JavaSE_04_IO]]
- [[🍀JavaSE_05_多線程并發]]
- [[🍀JavaSE_06_異常處理]]
- [[🍀JavaSE_07_注解]]
- [[🍀JavaSE_08_反射]]

## 🍀JDK環境安裝
1. 下載OpenJDK：[官方下載](https://jdk.java.net/15/)
2. 解壓放至C:\Java；為防止可能出現的非預期問題，存放路徑避免有空白字串
3. 設置環境變數：控制台\系統及安全性\系統-->進階系統設定-->環境變數
	```console
	變數名稱：JAVA_HOME
	變數值：C:\Java\jdk-12
	```

	```console
	變數名稱：path
	變數值：%JAVA_HOME%\bin
	```
4. 打開cmd，輸入指令，出現以下信息代表成功
	```console
	C:\Users\a0909>java -version
	openjdk version "12" 2019-03-19
	OpenJDK Runtime Environment (build 12+32)
	OpenJDK 64-Bit Server VM (build 12+32, mixed mode, sharing)

	C:\Users\a0909>javac -version
	javac 12
	```

### 錯誤記錄
#### Error: could not find java.dll
* 概述：執行`java -version`遇到"Error: could not find java.dll"與"Error: Could not find Java SE Runtime Environment."錯誤
Java1.8升級可能會出現此問題。
* 解決方法([參考](https://errerrors.blogspot.com/2019/10/java-error-could-not-find-javadll.html))：至環境變數-->path-->刪除 Oracle\\Java\\javapath 路徑

## 🍀開發工具安裝
### Eclipse
1. [官網下載](https://www.eclipse.org/downloads/packages/)
2. 打開Eclipse Intaller，選擇 Eclipse IDE for Enterprise Java Developers

### IDEA
1. [官網下載](https://www.jetbrains.com/idea/download/other.html)
2. [破解補丁下載](http://down.123520.net/dir/195471-41857840-9bd3fb)