#📆2022年 
狀態:: #☑DONE
完成日期:: 2022-02-03
標籤:: #💻編程/🌠Java/05_JavaSE加強 #🗂Overview 
子筆記:: [[🧠數據結構算法_01_排序]], [[🧠數據結構算法_02_線性表]], [[🧠數據結構算法_03_符號表]], [[🧠數據結構算法_04_樹]], [[🧠數據結構算法_05_圖]]
教程:: [教程_黑馬](https://www.bilibili.com/video/BV1iJ411E7xW), [教程_馬士兵](https://www.bilibili.com/video/BV1h5411x7Qf)
備註:: 

# 數據結構算法_00_Overview
- 參考教程：[数据结构与算法_菜鳥教程](https://www.runoob.com/data-structures/data-structures-tutorial.html)

## 🧠數據結構
### 邏輯結構分類
從具體問題中抽象出來的模型
- 集合結構
	![數據結構算法_00_Overview_01_集合結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_01_%E9%9B%86%E5%90%88%E7%B5%90%E6%A7%8B.png?raw=true)
- 線形結構：順序表、鏈表、棧、隊列
	![數據結構算法_00_Overview_02_線性結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_02_%E7%B7%9A%E6%80%A7%E7%B5%90%E6%A7%8B.png?raw=true)
- 樹形結構：二叉查找樹、堆、優先隊列、2-3查找樹、紅黑樹、B樹、B+樹、并查集
	![數據結構算法_00_Overview_03_樹形結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_03_%E6%A8%B9%E5%BD%A2%E7%B5%90%E6%A7%8B.png?raw=true)
- 圖形結構
	![數據結構算法_00_Overview_04_圖形結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_04_%E5%9C%96%E5%BD%A2%E7%B5%90%E6%A7%8B.png?raw=true)

### 物理結構分類
在電腦記憶體中實際的儲存方式
- 順序結構
	![數據結構算法_00_Overview_05_順序結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_05_%E9%A0%86%E5%BA%8F%E7%B5%90%E6%A7%8B.png?raw=true)
- 鏈式結構
	![數據結構算法_00_Overview_06_鏈式結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_06_%E9%8F%88%E5%BC%8F%E7%B5%90%E6%A7%8B.png?raw=true)

## 🧠算法
- 算法(Algorithm)：同一問題的不同解決方法；算法往往是針對特定數據結構，如：數組、鏈表…等。
- 測算算法的優劣：幅度不夠用循環來湊。
	1. 時間測算：核心懆作的執行次數(不包含循環、判斷)。`System.currentTimeMillis();`
		- 影響因素：算法采用的方案、騙譯産生的代碼質量(如Java7、Java8)、問題規模、機器效能
	2. 空間測算：同樣的效果所需要花費額外的儲存空間。

### 時間複雜度分析
時間複雜程度，電腦解決問題的時間隨著問題規模的擴大，時間是如何變化(一般講的是最差情況)

#### Big O標記法分析規則
1. 只保留最高階層，如：n<sup>2</sup> + n，轉為n<sup>2</sup>
2. 刪除常數項，如：2n + 2，轉為n

#### Big O標記法常見類型
- **O(1)**，以1代替常數
	隨著數組長度的增加，訪問所需要的時間不會改變。
	![數據結構算法_00_Overview_07_順序表](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_07_%E9%A0%86%E5%BA%8F%E8%A1%A8.png?raw=true)
- **O(n)**
	1. 求數組平均數：需要取得每個數組的所有數，數組長度為n則需要加n次再平均，需要n+1個步驟，省略常數項，故O(n)。
	2. 訪問鏈表需要從頭開始依序找到最後一個位置，若長度為10的鏈表需需要10個步驟，則長度為1000的鏈表需要1000的步驟。
		![數據結構算法_00_Overview_08_鏈表](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/05_JavaSE%E5%8A%A0%E5%BC%B7/%F0%9F%A7%A0%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95/images/%E6%95%B8%E6%93%9A%E7%B5%90%E6%A7%8B%E7%AE%97%E6%B3%95_00_Overview_08_%E9%8F%88%E8%A1%A8.png?raw=true)
- **O(n<sup>2</sup>)**
- **O(n<sup>3</sup>)**
- **O(logn)**

複雜度比較：O(1) < O(logn) < O(nlogn) < O(n<sup>2</sup>) < O(n<sup>3</sup>)

### 空間複雜度分析
- 基本型別的內存占用：可參考：[[🍀JavaSE_01_基礎語法]]--基本型別
- 電腦讀取數據，一次讀1個字節(8bit)
- 一個對象的引用，也就是變量或機器地址，需要用8個字節表示
- 對象是以8個字節為單位進行存儲，不足8個字節會補足
	```java
	/**
	* 此對象需要用12字節存儲，但實際上會用4個字節填充成16個字節存到內存
	*/
	class TestClass{// 8個字節
		int year = 2021;// 4個字節
	}
	```
- 數組需要另外用至少24個字節的頭信息：對象開銷(16字節) + 長度信息(4字節) + 填充(4字節)
- 空間複雜度也可以用Big O標記法分析
- 一般情況下比較少進行空間複雜度分析(嵌入式開發會比較需要)



