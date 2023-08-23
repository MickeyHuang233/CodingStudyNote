# JPQL
- JPQL(Java Persistence Query Language)類似於SQL，但查詢資料來源是實體類(Entity)，並不是資料庫本身，通過`Query`接口封裝執行
- Hibernate官方文檔：https://docs.jboss.org/hibernate/orm/6.2/userguide/html_single/Hibernate_User_Guide.html#query-language

## 🗄️Select
### JPQL查詢所有欄位
- 實體類定義
	```java
	@Entity
	@Table(name = "jpa_t18_user")
	public class T18_User {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String userName;
	
		private String password;
	
		private Date createDate;
		// 省略getter、setter、toString()
	}
	```
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	/**
	 * 使用JPQL查詢實體類所有欄位
	 */
	@Test
	public void t18_01_helloWorld() {
		String jpql = "SELECT u FROM T18_User u WHERE u.userName = ?1"; // 查詢全部欄位
	//		String jpql = "FROM T18_User u WHERE u.userName = ?1"; // 可省略SELECT
		Query query = this.entityManager.createQuery(jpql);
		query.setParameter(1, "Mickey"); // 占位符從1開始
		List<T18_User> results = query.getResultList();
		results.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```

### JPQL查詢部分欄位
- 實體類定義
	```java
	@Entity
	@Table(name = "jpa_t18_user")
	public class T18_User {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String userName;
	
		private String password;
	
		private Date createDate;
	
		public T18_User() {
			super();
		}
	
		// 用於查詢部分屬性會用到的構造器
		public T18_User(String password, Date createDate) {
			super();
			this.password = password;
			this.createDate = createDate;
		}
		// 省略getter、setter、toString()
	}
	```
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	/**
	 * 使用JPQL查詢實體類部分欄位
	 */
	@Test
	public void t18_02_helloWorld() {
		// 默認情況下，若只查詢部分屬性，則返回Object[]或List<Object[]>
	//		String jpql = "SELECT new u.password, u.createDate FROM T18_User u WHERE u.userName = ?1"; // 返回List<Object[]>
		// 指定目標類，需要有相應的構造器才可使用
		String jpql = "SELECT new T18_User(u.password, u.createDate) FROM T18_User u WHERE u.userName = ?1";
		List results = this.entityManager.createQuery(jpql).setParameter(1, "Mickey").getResultList();
		results.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```

###  @NamedQuery定義JPQL
- 實體類定義
	```java
	// 定義JPQL
	@NamedQueries ({
	 @NamedQuery(
	     name="test01",
	     query="SELECT u FROM T18_User u WHERE u.userName = ?1")
	})
	@Entity
	@Table(name = "jpa_t18_user")
	public class T18_User {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String userName;
	
		private String password;
	
		private Date createDate;
		// 省略getter、setter、toString()
	}
	```
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	/**
	 * 呼叫實體類@NamedQuery定義的JPQL
	 */
	@Test
	public void t18_03_helloWorld() {
		Query query = //
			this.entityManager.createNamedQuery("test01") //
			.setParameter(1, "Monkey");
		List<T18_User> results = query.getResultList();
		results.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```

### SQL查詢資料庫
- 實體類定義
	```java
	@Entity
	@Table(name = "jpa_t18_user")
	public class T18_User {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String userName;
	
		private String password;
	
		private Date createDate;
		// 省略getter、setter、toString()
	}
	```
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	/**
	 * 使用SQL查詢資料庫
	 */
	@Test
	public void t18_04_helloWorld() {
		String sql = "select * from jpa_t18_user where userName=?";
		Query query = //
			this.entityManager.createNativeQuery(sql, T18_User.class) //
			.setParameter(1, "Monkey");
		List results = query.getResultList();
		results.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```

### 查詢緩存
```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

/**
 * 查詢緩存
 */
@Test
public void t19_01_selectCache() {
	String jpql = "SELECT u FROM T18_User u WHERE u.userName = ?1"; // 查詢全部欄位
	this.entityManager.createQuery(jpql) //
		.setParameter(1, "Mickey") //
		.setHint(QueryHints.HINT_CACHEABLE, true) // 設置hibernate查詢緩存
		.getResultList() //
		.forEach(u -> System.out.println(u.toString()));

	// 查詢重覆的JPQL，實標上只會在資料庫查詢一次，後面的查詢結果是從緩存拿的
	this.entityManager.createQuery(jpql) //
	.setParameter(1, "Mickey") //
	.setHint(QueryHints.HINT_CACHEABLE, true) // 設置hibernate查詢緩存
	.getResultList() //
	.forEach(u -> System.out.println(u.toString()));
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

### ORDER BY
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t20_01_orderBy() {
		String jpql = "SELECT u FROM T18_User u WHERE u.userName = ?1 ORDER BY u.password";
		this.entityManager.createQuery(jpql) //
			.setParameter(1, "Mickey") //
			.getResultList() //
			.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 執行SQL語句
	```sql
	select t1_0.id,t1_0.createDate,t1_0.password,t1_0.userName from jpa_t18_user t1_0 where t1_0.userName=? order by t1_0.password
	```

### GROUP BY
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t20_02_groupBy() {
		String jpql = "SELECT u FROM T18_User u GROUP BY u.userName";
		this.entityManager.createQuery(jpql) //
			.getResultList() //
			.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 執行SQL語句
	```sql
	select t1_0.id,t1_0.createDate,t1_0.password,t1_0.userName from jpa_t18_user t1_0 group by t1_0.userName
	```

### 關聯查詢
- 實體類設置，可參考：[[🗄️JPA_04_映射關聯關系#🗄️單向一對多]]
	```java
	@Entity
	@Table(name = "jpa_t21_user")
	public class T21_User {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String userName;
	
		@JoinColumn(name = "userId")
		@OneToMany(cascade = CascadeType.REMOVE)
		private Set<T21_Order> orders = new HashSet<>();
		// getter、setter、toString()省略
	}
	```

	```java
	@Entity
	@Table(name = "jpa_t21_order")
	public class T21_Order {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String orderName;
	
		@Temporal(TemporalType.TIMESTAMP)
		private Date createDate;
		// getter、setter、toString()省略
	}
	```
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}

	@Test
	public void t21_01_join() {
		// 分批查詢
		String jpql = "SELECT u FROM T21_User u";
	//		String jpql = "SELECT u FROM T21_User u LEFT OUTER JOIN u.orders";
		this.entityManager.createQuery(jpql) // 
			.getResultList() // 會先查詢JPQL的表
			.forEach(u -> System.out.println(u.toString())); // 使用到時才會到外鍵關連的表再查詢

		// 一次執行Join語句查詢
		String joinJpql = "SELECT u FROM T21_User u LEFT OUTER JOIN FETCH u.orders";
		this.entityManager.createQuery(joinJpql) // 
			.getResultList() //
			.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 執行SQL語句
	```sql
	-- 分批查詢
	select t1_0.id,t1_0.userName from jpa_t21_user t1_0
	select o1_0.userId,o1_0.id,o1_0.createDate,o1_0.orderName from jpa_t21_order o1_0 where o1_0.userId=?
	
	-- 一次執行Join語句查詢
	select t1_0.id,o1_0.userId,o1_0.id,o1_0.createDate,o1_0.orderName,t1_0.userName from jpa_t21_user t1_0 left join jpa_t21_order o1_0 on t1_0.id=o1_0.userId
	```

### 子查詢
- 測試方法
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}

	/**
	 * 子查詢，查詢指定用戶名的所有訂單
	 */
	@Test
	public void t22_01_subQuery() {
		String jpql = //
			"SELECT o FROM T21_Order o " + //
			"WHERE o in ( " + //
			"	SELECT u.orders FROM T21_User u WHERE u.userName = ?1" + //
			")";
		this.entityManager.createQuery(jpql) //
			.setParameter(1, "Mickey") //
			.getResultList() //
			.forEach(u -> System.out.println(u.toString()));
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 執行SQL語句
	```sql
	select t1_0.id,t1_0.createDate,t1_0.orderName 
	from jpa_t21_order t1_0 
	where t1_0.id in(
		select o1_0.id 
		from jpa_t21_user t2_0 
		join jpa_t21_order o1_0 
		on t2_0.id=o1_0.userId 
		where t2_0.userName=?
	)
	```

## 🗄️Update
```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t23_01_update() {
	String jpql = //
		"UPDATE T21_Order o SET o.createDate = ?1 " + //
		"WHERE o.orderName = ?2";
	int count = this.entityManager //
		.createQuery(jpql) //
		.setParameter(1, new Date()) //
		.setParameter(2, "order01") //
		.executeUpdate();
	System.out.println(count);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

## 🗄️Delete
```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t23_02_delete() {
	String jpql = "DELETE FROM T21_Order o WHERE o.id = ?1";
	int count = this.entityManager //
		.createQuery(jpql) //
		.setParameter(1, 2) //
		.executeUpdate();
	System.out.println(count);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```