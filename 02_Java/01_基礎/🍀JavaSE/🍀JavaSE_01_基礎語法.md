# JavaSE_01_基礎語法
## 🍀關鍵字 Java Keywords
| Keywords |        |            |           |            |           |              |
| -------- | ------ | ---------- | --------- | ---------- | --------- | ------------ |
| bstract  | assert | boolean    | break     | byte       | case      | catch        |
| char     | class  | const      | continue  | default    | do        | double       |
| else     | enum   | extends    | final     | finally    | float     | for          |
| goto     | if     | implements | import    | instanceof | int       | interface    |
| long     | native | new        | package   | private    | protected | public       |
| return   | short  | static     | strictfp  | super      | switch    | synchronized |
| this     | throw  | throws     | transient | try        | void      | volatile     |
| while    |        |            |           |            |           |              |

## 🍀基本型別 Primitive Type
| default | Type    | Description             | Default | Size    | Example Literals                                   |
| ------- | ------- | ----------------------- | ------- | ------- | -------------------------------------------------- |
|         | boolean | true or false           | false   | 1 bit   | true, false                                        |
|         | byte    | twos complement integer | 0       | 8 bits  | \-2^7 ~ 2^7 -1                                     |
|         | char    | Unicode character       | \\u0000 | 16 bits | '\\u0000' (or 0) ~ '\\uffff' (or 65,535 inclusive) |
| V       | int     | twos complement integer | 0       | 32 bits | \-2^31 ~ 2^31 -1                                   |
|         | long    | twos complement integer | 0       | 64 bits | \-2^63 ~ 2^63 -1                                   |
| V       | float   | IEEE 754 floating point | 0.0     | 32 bits | 1.23e100f, -1.23e-100f, .3f, 3.14F                 |
|         | double  | IEEE 754 floating point | 0.0     | 64 bits | 1.23456e300d, -1.23456e-300d, 1e1d                 |
- 不用特別記，若想知道byte的作用域可用`Byte.MINVALUE和Byte.MAXVALUE`
- logn及float的寫法：
	```java
	long v01 = 123L; 
	float v02 = 0.123F;
	```
- 任何數字值或字元值都不會被視為boolean值，也無法轉成boolean型別
- char以'`'\\u0000'`表示
- 自動轉型
![JavaSE_01_基礎語法_轉型](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/01_%E5%9F%BA%E7%A4%8E/%F0%9F%8D%80JavaSE/images/JavaSE_01_%E5%9F%BA%E7%A4%8E%E8%AA%9E%E6%B3%95_%E8%BD%89%E5%9E%8B.png?raw=true)
- 強制轉型 Casting：`(float)`
	- 注意：把數值範圍比較大的資料強制轉換為數值範圍比較小的型別, 因此有可能在轉型後造成資料值不正確。

## 🍀轉義符
| Notation | Character represented                |
| -------- | ------------------------------------ |
| \\n      | Newline (0x0a)                       |
| \\r      | Carriage return (0x0d)               |
| \\f      | Formfeed (0x0c)                      |
| \\b      | Backspace (0x08)                     |
| \\s      | Space (0x20)                         |
| \\t      | tab                                  |
| \\"      | Double quote                         |
| \\'      | Single quote                         |
| \\\\     | \\                                   |
| \\ddd    | Octal character (ddd)                |
| \\uxxxx  | Hexadecimal UNICODE character (xxxx) |

## 🍀運算子 Operator
| 類型         | 操作符                               | 描述                                                                                                       | 輸入型別         | 輸出型別                 |
| ------------ | ------------------------------------ | ---------------------------------------------------------------------------------------------------------- | ---------------- | ------------------------ |
| 算術         | +  -  \*  /  ％                      | 加、减、乘、除、余数<br>先乘除後加減                                                                       | 所有整數、浮點數 | int long<br>float double |
| 比較         | \==  !=  >  <  >=  <=                | 相等、不相等、大於、小於、大於等於、小於等於                                                               | 數字型別(含char) | boolean                  |
| 邏輯         | ＆ &&<br>\| \|\|<br>^                | 且，最好用&&(短路運算)<br>或，最好用\|\|(短路運算)<br>如果相对应位值相同，则结果为0，否则为1               | boolean          | boolean                  |
| 位元(嵌入式) | ~  &  \|  ^                          | ~：按位补运算符翻转操作数的每一位，即0变成1，1变成0。                                                      | 整數型別(含char) | int long                 |
| 位移(嵌入式) | <<     >>    >>>                     | \*2     /2<br>按位右移补零操作符。左操作数的值按右操作数指定的位数右移，移动得到的空位以零填充。(二進位用) | 整數型別(含char) | int long                 |
| 遞增/減      | ++ --                                | 自增、自减                                                                                                 | 數字型別         | 與輸入型別一樣           |
| 字串         | +                                    |                                                                                                            | 至少一個是String | String                   |
| 條件         | boolean運算? true時的值: false時的值 |                                                                                                            | boolean          | 值是什麼型別就是什麼     |

| 操作符 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| \=     | 賦值                                                         |
| \+\=   | 加和赋值操作符，它把左操作数和右操作數相加赋值给左操作数     |
| \-\=   | 减和赋值操作符，它把左操作数和右操作数相减赋值给左操作数     |
| \*\=   | 乘和赋值操作符，它把左操作数和右操作数相乘赋值给左操作数     |
| /\=    | 除和赋值操作符，它把左操作数和右操作数相除赋值给左操作数     |
| (％)\= | 取模和赋值操作符，它把左操作数和右操作数取模后赋值给左操作数 |
| <<\=   | 左移位赋值运算符                                             |
| \>\>\= | 右移位赋值运算符                                             |
| ＆\=   | 按位与赋值运算符                                             |
| ^\=    | 按位异或赋值操作符                                           |
| \|\=   | 按位或赋值操作符                                             |

## 🍀循環
```java
while( 布尔表达式 ) {  
//循环内容
}
```

```java
do {  
//循环内容
}while(布尔表达式);
```

```java
for(初始化; 布爾表達式; I++) {  
//循环内容
}
```

```java
for(聲明語句 : 表达式) { //代加強型迴圈，較無靈活性，但方便使用  
//循环内容 
}
```

```java
break;
```

```java
continue;
```

## 🍀條件
```java
if(布爾表達式) {
//为true的语句
}
```

```java
if(布爾表達式){
//為true的執行內容
}
else{
//為false的執行內容
}
```

```java
if(布爾表達式 1){ /布尔表达式 1值为true的執行內容}  
else if(布爾表達式 2){ //布尔表达式 2值为true的執行內容}  
else if(布爾表達式 3){ //布尔表达式 3值为true的執行內容}  
else { //以上布尔表达式都不为true的執行內容}
```

```java
switch(byte|char|short|int|String 運算式){  
case value :  
	//執行內容
	break;  
case value :  
	//執行內容
	break;  
default :  
	//執行內容
}
```

## 🍀陣列 Array
- 相同型態資料的集合，長度建立後不能再變動
```java
int[] a = new int[20];
dataType[] arrayRefVar = {value0, value1, ..., valuen};
```
- 建立一個陣列後，陣列每個元素都會自動初始化(依型本類型別的初始值)
- 可透過`System.arraycopy();`可復製陣列內容
- 額外：
	- [Java陣列操作的10大方法](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/321316/)
	- [Java 8官方文件_Array](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)






























