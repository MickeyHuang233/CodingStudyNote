#📆2021年 
狀態:: #☑DONE 
完成日期:: 2021-08-01
標籤:: #💻編程/🌠Java/15_其他 #🗂Overview 
子筆記:: 
教程:: 
備註:: 筆記搬運(Google Drive-->Obsidian)

# Launch4j
導出jar後可以從cmd輸入`java -jar test.jar`執行，但這樣執行不太方便，就找了一下網上的資料，launch4j不僅免費而且用起來難度也不大，因此在這記錄一下我這次的使用過程，以便以後忘記可以查閱。
    
## 🧰在eclipse導出.jar
專案→右鍵→Export...→Runable JAR file
![[Launch4j_01_在eclipse導出jar.png]]
Launch configuration，main方法所在的程式
Export destination，導出.jar路徑

## 🧰下載launch4j
launch4j官網([http://launch4j.sourceforge.net/](http://launch4j.sourceforge.net/))中有安裝版和綠色版可以下載。

## 🧰啟動launch4j
### Basic
- Output file，要導出.exe文件位置及文件名稱。
- jar，要轉換.jar的位置。
- icon，.exe文件的顯示圖式， 因為懶得找.icon就空白。

![[Launch4j_02_Basic.png]]

### Classpath
看其他教程很多都會寫Classpath，但我寫的時候執行時會出現”無法載入主要類別”的錯誤，嘗試了一下不寫Classpath反而不會出錯，launch4j會識別主執行程序，還蠻方便的，暫時還找不到除此之外的解決方法。
![[Launch4j_03_Classpath.png]]

### Header
Header type：
- GUI，執行.exe不會出現控制台。
- Console，執行.exe會出現控制台。

![[Launch4j_04_Header.png]]

### JRE
Min JRE version、Max JRE version，支持最小和最大的JRE版本，格式為 x.x.x[_xx]。
Min JRE version右邊的下拉選擇Prefer public JRE, but use JDK runtime if newer。
![[Launch4j_05_JRE.png]]

## 🧰執行exe
設定完成之後，點擊工具欄上的齒輪就可以導出.exe。
![[Launch4j_06_執行exe.png]]