# JPA_02_注解
主要定義實體類屬性和資料表欄位的關系

## 🗄️@Entity
`@Entity`表示該Java類為實體類，將映射到資料表

```java
@Entity // 讓JPA知道這是一個實體類，也就是和數據表映射的類
public class T02_User {
	// 省略
}
```

## 🗄️@Table
`@Table`用於定義實體類映射的資料表名稱，各參數詳細說明可參考：[JPA @Table 注解](https://fanlychie.github.io/post/jpa-table-annotation.html)

```java
@Entity // 讓JPA知道這是一個實體類，也就是和數據表映射的類
@Table(name = "jpa_t01_user") // 指定映射數據表，表名默認為類名小寫
public class T02_User {
	// 省略
}
```

## 🗄️主鍵相關注解
### @Id
`@Id`用於聲明實體類對應資料表的==主鍵==，建議定義於getter方法前

### @GeneratedValue
`@GeneratedValue`用於標注主鍵生成策略，在`jakarta.persistence.GenerationType`定義以下主鍵生成策略
- `AUTO`：默認，JPA自動選擇合適的策略
- `IDENTITY`：采用數據庫ID自增的方式生成主鍵，Oracle不支持
- `SEQUENCE`：通過序列産生主鍵，通過`@GenericGenerator`指定序列名，MySQL不支持
- `TABLE`：通過表産生主鍵，框架借由表摸擬序列産生主鍵；使用此策略可使應用更容易進行數據庫移植
- `UUID`

```java
@Entity
@Table(name = "jpa_t01_user")
public class T02_User {
//	@Id
//	@GeneratedValue(strategy = GenerationType.TABLE)
	private Integer id;
	
	// 建議定義於getter方法前
	@Id // 表示為主鍵
	@GeneratedValue(strategy = GenerationType.AUTO) // 自增主鍵
	public Integer getId() {
		return id;
	}
}
```

### @GenericGenerator
`@GenericGenerator`用於自定義主鍵生成策略，`strategy`可指定主鍵生成策略
- `native`，將主鍵生成工作交由數據庫完成
- `uuid`，主鍵編碼為一個32位的16進制字符串，占用空間較大
	```java
	@Id
	@GeneratedValue(generator = "test") // 指定主鍵生成器名稱
	@GenericGenerator(name = "test", strategy = "uuid")
	private String id;
	```
- `hilo`，需要額外一個資料表保存hi值
- `assigned`，將主鍵生原工作交由程序處理
- `identity`，使用SQL Server、MySQL自增字段，不能用於Oracle
- `select`，使用觸發器生成主鍵，使用較少
- `sequence`，調用底層數據庫的序列生成主鍵，需要另外指定序列名
	```java
	@GeneratedValue(generator = "test")  
	@GenericGenerator(name = "test", strategy = "sequence", parameters = { @Parameter(name = "sequence", value = "底層數據庫sequenceName") }
	```
- `seqhilo`
- `increment`
- `foreign`，使用另外一個關院對象的主鍵作為該表的主鍵，主要用於一對一關系中
- `guid`，采用數據庫底層guid算法機制，不同的數據庫對應不同的方法：
	1. MySQL，`uuid()`
	2. SQL Server，`newid()`
	3. Oracle，`rawtohex(sys_guid())`

### 自定義主鍵生成策略
- 需要在實體類定義對應的主鍵表名及相應欄位
	```java
	@Entity
	@Table(name = "jpa_t06_generator")
	public class T06_Generator {
		@Id  
		@GeneratedValue(
			strategy = GenerationType.TABLE, // 通過表産生主鍵
			generator = "test" // 指定主鍵生成器
		)  
		@TableGenerator(
			name = "test", // 主鍵生成器名稱
			allocationSize = 1, // 主鍵自增步長，默認為50
			table = "jpa_t06_sequence", // 對應主鍵表名稱
			pkColumnName = "sequence_max_id", // 對應主鍵表的欄位名，用於保存資料表名稱
			valueColumnName = "sequence_count" // 對應主鍵表的欄位名，用於保存上次生成的主鍵值
		)  
		private Integer id;
	
		// 省略getter、setter
	}
	```
- 插入資料，會在主鍵表中更新主鍵值
	```console
	sequence_count|sequence_max_id  |
	--------------+-----------------+
	             3|jpa_t06_generator|
	             7|jpa_t07_test     |
	```

## 🗄️@Basic
`@Basic`表示沒有任何標注的getter方法，默認會添加
- `fetch`屬性，表示讀取策略，EAGER表示直接讀取(默認)，LAZY表示延遲加載
- `optional`屬性，表示該屬性是否允許為null，默認為true

```java
@Entity
@Table(name = "jpa_t01_user")
public class T02_User {
	private String userName;

	@Basic(fetch = FetchType.LAZY, optional = false)
	public String getUserName() {
		return userName;
	}

	// 省略getter、setter
}
```

## 🗄️@Column
`@Column`標注屬性與表欄位的關系，屬性說明可參考：[API文檔](https://docs.oracle.com/javaee/5/api/javax/persistence/Column.html)
- `name`，當屬性名與表欄位名稱不同時，則需要指定映射的欄位名稱
- `unique`，表示該欄位是否為唯一標識，默認為false
- `nullable`，表示該欄位是否為null，默認為true
- `insertable`，表示使用`INSERT` SQL時是否需要插入此欄位
- `columnDefinition`，主要用於創建表時的SQL語句
- `table`
- `length`，當欄位屬性為varchar時，表示欄位長度，默認為255
- `precision`，當欄位屬性為double時，表示總長度
- `scale`，當欄位屬性為double時，表示小數點所占位數

```java
@Entity
@Table(name = "jpa_t01_user")
public class T02_User {
	@Column(name = "password", nullable = false)
	private String pwd;

// 也可定義於getter前
// @Column(name = "password", nullable = false)
	public String getPwd() {
		return pwd;
	}
}
```

## 🗄️@Transient
`@Transient`表示該屬性並非資料表欄位的映射，ORM框架會忽略此屬性，主要用於工具方法；若無標示`@Transient`，則默認標示為`@Basic`

```java
@Transient
public String getInfo() {
	return "T02_User [id=" + id + ", userName=" + userName + ", pwd=" + pwd + ", city=" + city + "]";
}
```

## 🗄️@Temporal
`@Temporal`用於定義日期時間的格式
- `TemporalType.TIMESTAMP`，日期和時間
- `TemporalType.DATE`，僅日期
- `TemporalType.TIME`，僅時間

```java
@Entity
@Table(name = "jpa_t01_user")
public class T02_User {
	@Temporal(TemporalType.TIMESTAMP)
	private Date birthDayTime;
	
	@Temporal(TemporalType.DATE)
	private Date birthDay;

	@Temporal(TemporalType.TIME)
	private Date birthTime;
}
```

```console
birthDay  |birthTime|birthDayTime       |
----------+---------+-------------------+
2023-08-01| 19:46:32|2023-08-01 19:46:32|
```