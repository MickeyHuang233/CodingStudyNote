###### tags: `#📆2021年`
###### tags: `💻編程/🌠Java/01_Java基礎`
###### tags: `🗂Overview`

# Java 8 新特性
## 💡底層處理調整
### HashMap處理
HashMap在Java8前是以**數組 + 鏈表**儲存，而在Java8是以**數組 + 鏈表/紅黑數**儲存
- Java8前，步驟：
	![image](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%92%A1Java%E6%96%B0%E7%89%B9%E6%80%A7/%F0%9F%92%A1Java8/images/Java%208_01_Java8%E5%89%8DHashMap.png?raw=true)
	1. key用hash算出要存放的數組index
	2. 如果數組中鏈表為空，直接加到鏈表
	3. 如果數組中鏈表不為空，依次比對鏈表的key，如果一樣則更新值；不一樣則在鏈表最後加上
- Java8，步驟：
	![image](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%92%A1Java%E6%96%B0%E7%89%B9%E6%80%A7/%F0%9F%92%A1Java8/images/Java%208_02_Java8%20HashMap.png?raw=true)
	1. 前面的步驟和Java8前都一樣
	2. 如果鏈表的數量很多的話，儲存方式會改為紅黑樹
- 改動優點：
	1. 紅黑數除新增外的效率都比鏈表好

### JVM調整
Java 8前後JVM的調整
![iamge](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%92%A1Java%E6%96%B0%E7%89%B9%E6%80%A7/%F0%9F%92%A1Java8/images/Java%208_03_JVM%E8%AA%BF%E6%95%B4.png?raw=true)

- Java8前：方法區占用一部分的永久區空間
- Java8
	1. 刪除永久區，並獨立出MetaSpace，將方法區存放至MetaSpace
	2. JVM參數失效：`PremGenSize`、`MaxPremGenSize`
	3. JVM參數出現：`MetaspaceSize`、`MaxMetaspaceSize`
- 改動優點：
	1. 減少方法區的垃圾回收機制(方法區容量快滿才會調用)，效率上升
	2. 減少OutOfMemeryException出現概率

---
Catalog
---
- [💡Java8_01_Lambda](https://hackmd.io/@W3snnHv8TgC_U2ElYL9ATQ/Java8_01_Lambda)
- [💡Java8_02_StreamAPI](https://hackmd.io/@W3snnHv8TgC_U2ElYL9ATQ/Java8_02_StreamAPI)
- []