###### tags: `#📆2021年`
###### tags: `💻編程/🌠Java/01_Java基礎`
###### tags: `🗂Overview`

# Java 8 新特性
## 💡底層處理調整
### HashMap處理
HashMap在Java8前是以**數組 + 鏈表**儲存，而在Java8是以**數組 + 鏈表/紅黑數**儲存
- Java8前，步驟：
	![[images/Java 8_01_Java8前HashMap.png]]
	1. key用hash算出要存放的數組index
	2. 如果數組中鏈表為空，直接加到鏈表
	3. 如果數組中鏈表不為空，依次比對鏈表的key，如果一樣則更新值；不一樣則在鏈表最後加上
- Java8，步驟：
	![[images/Java 8_02_Java8 HashMap.png]]
	1. 前面的步驟和Java8前都一樣
	2. 如果鏈表的數量很多的話，儲存方式會改為紅黑樹
- 改動優點：
	1. 紅黑數除新增外的效率都比鏈表好

### JVM調整
Java 8前後JVM的調整
![[images/Java 8_03_JVM調整.png]]

- Java8前：方法區占用一部分的永久區空間
- Java8
	1. 刪除永久區，並獨立出MetaSpace，將方法區存放至MetaSpace
	2. JVM參數失效：`PremGenSize`、`MaxPremGenSize`
	3. JVM參數出現：`MetaspaceSize`、`MaxMetaspaceSize`
- 改動優點：
	1. 減少方法區的垃圾回收機制(方法區容量快滿才會調用)，效率上升
	2. 減少OutOfMemeryException出現概率
