# JPA_03_API
## 🗄️Persistence
`Persistence`用於建立`EntityManagerFactory`實例

```java
@Test
public void t07_persistence() {
	// 使用persistence.xml定義的參數建立EntityManagerFactory
//		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	
	// 指定EntityManagerFactory屬性
	Map<String, Object> properties = new HashMap<>();
	properties.put("hibernate.show_sql", false);
	// 在代碼中指定參數建立EntityManagerFactory，鍵值與persistence.xml的name、value相對應
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit", properties);
	
	EntityManager entityManager = entityManagerFactory.createEntityManager();
	EntityTransaction entityTransaction = entityManager.getTransaction();
	entityTransaction.begin();
	
	T06_Generator generator = new T06_Generator();
	generator.setName("mickey");
	entityManager.persist(generator);
	
	entityTransaction.commit();

	entityManager.close();
	entityManagerFactory.close();
}
```

## 🗄️EntityManagerFactory
`EntityManagerFactory`主要用於建立`EntityManager`實例

### 前置：實體類屬性配置
```java
@Entity
@Table(name = "jpa_t01_user")
public class T02_User {
	@Id
	@GeneratedValue(strategy = GenerationType.TABLE)
	private Integer id;

	@Column(nullable = false)
	private String userName;

	@Column(name = "password", nullable = false)
	private String pwd;

	@Column(nullable = false)
	private String city;

	@Temporal(TemporalType.TIMESTAMP)
	private Date birthDayTime;

	@Temporal(TemporalType.DATE)
	private Date birthDay;

	@Temporal(TemporalType.TIME)
	private Date birthTime;
	// getter、setter、toString()省略
}
```

### find()
類似於hibernate中Session的`get()`，方法執行時馬上到資料庫查詢，查詢結果保存於內存中，可參考：[[⛄Hibernate_00_Overview#get()和load()]]
- 測試方法
	```java
	/**
	* 查
	*/
	@Test
	public void t07_01_find() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		// 通過主鍵查詢資料，類似於hibernate中Session的get()，方法執行時馬上到資料庫查詢，查詢結果保存於內存中
		T02_User user = entityManager.find(T02_User.class, 2);
		System.out.println("----------");
		System.out.println(user.toString());
	}
	```
- 打印結果
	```console
	Hibernate: select t1_0.id,t1_0.birthDay,t1_0.birthDayTime,t1_0.birthTime,t1_0.city,t1_0.password,t1_0.userName from jpa_t01_user t1_0 where t1_0.id=?
	----------
	T02_User [id=2, userName=mickey1, pwd=123456, city=Taipei, birthDayTime=2023-08-01 21:11:29.0, birthDay=2023-08-01, birthTime=21:11:29]
	```

### getReference()
類似於hibernate中Session的`load()`，變量調用時才會到資料庫查詢，懶加載，可參考：[[⛄Hibernate_00_Overview#get()和load()]]
- 測試方法
	```java
	/**
	* 查，懶加載
	*/
	@Test
	public void t07_02_getReference() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		// 通過主鍵查詢資料，類似於hibernate中Session的load()，變量調用時才會到資料庫查詢，懶加載
		T02_User user = entityManager.getReference(T02_User.class, 2);
		System.out.println("----------");
		System.out.println(user.toString());
	}
	```
- 打印結果
	```console
	----------
	Hibernate: select t1_0.id,t1_0.birthDay,t1_0.birthDayTime,t1_0.birthTime,t1_0.city,t1_0.password,t1_0.userName from jpa_t01_user t1_0 where t1_0.id=?
	T02_User [id=2, userName=mickey1, pwd=123456, city=Taipei, birthDayTime=2023-08-01 21:11:29.0, birthDay=2023-08-01, birthTime=21:11:29]
	```

### persist()
類似於hibernate中Session的`save()`，使對象由臨時狀態轉為持久化狀態，可參考：[[⛄Hibernate_00_Overview#測試]]
```java
/**
 * 增
 */
@Test
public void t07_03_persist() {
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	EntityManager entityManager = entityManagerFactory.createEntityManager();
	EntityTransaction entityTransaction = entityManager.getTransaction();
	entityTransaction.begin();

	// 持久化操作
	T02_User user = new T02_User();
	user.setUserName("mickey");
	user.setPwd("123456");
	user.setCity("Taipei");
	user.setBirthDayTime(new Date());
	user.setBirthDay(new Date());
	user.setBirthTime(new Date());

	System.out.println(user.toString()); // 存檔前主鍵不會生成
	entityManager.persist(user); // 保存持久化物備件
	System.out.println(user.toString()); // 存檔後會生成主鍵，並放入屬性
	entityTransaction.commit();
}
```

### remove()
- 類似於hibernate中Session的`delete()`，刪除資料庫中數據、持久化對象，無法刪除游離對象
- 游離對象：數據庫中有記錄，但對象實例沒有和Session有關聯

```java
/**
 * 刪
 */
@Test
public void t07_04_remove() {
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	EntityManager entityManager = entityManagerFactory.createEntityManager();
	EntityTransaction entityTransaction = entityManager.getTransaction();
	entityTransaction.begin();
	
	// 無法刪除游離對象，java.lang.IllegalArgumentException: Removing a detached instance com.mickey.entry.T02_User#2
//		T02_User user = new T02_User();
//		user.setId(2);
	
	T02_User user = entityManager.find(T02_User.class, 2);
	entityManager.remove(user);
	entityTransaction.commit();
}
```

### merge()
`merge()`主要處理實體類的同步，也就是數據的插入、更新，類似於hibernate中Session的`saveOrUpdate()`

- 若傳入臨時對象，對新建另一個實體類對象，並對新對象進行持久化操作後insert
	```java
	@Test
	public void t09_01_merge() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		EntityTransaction entityTransaction = entityManager.getTransaction();
		entityTransaction.begin();
		
		T02_User userBefore = new T02_User();
		userBefore.setUserName("mickey");
		userBefore.setPwd("123456");
		userBefore.setCity("Taipei");
		userBefore.setBirthDayTime(new Date());
		userBefore.setBirthDay(new Date());
		userBefore.setBirthTime(new Date());
	
		// 若傳入臨時對象，對新建另一個實體類對象，並對新對象進行持久化操作後insert
		T02_User userAfter = entityManager.merge(userBefore);
		entityTransaction.commit();
		
		System.out.println("userBefore id : " + userBefore.getId()); // userBefore id : null
		System.out.println("userAfter id : " + userAfter.getId()); // userAfter id : 202
	}
	```
- 若傳入游離對象，在`EntityManager`緩存中無此對象，且此對象在資料庫查無資料，則會執行insert；若有資料，則會執行update
	```java
	@Test
	public void t09_02_merge() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		EntityTransaction entityTransaction = entityManager.getTransaction();
		entityTransaction.begin();
		
		T02_User userBefore = new T02_User();
		userBefore.setUserName("anny");
		userBefore.setPwd("111222");
		userBefore.setCity("Taipei");
		userBefore.setBirthDayTime(new Date());
		userBefore.setBirthDay(new Date());
		userBefore.setBirthTime(new Date());
		userBefore.setId(302); // 游離對象：數據庫中有記錄，但對象實例沒有和Session有關聯
	
		// 若傳入游離對象
		// 在EntityManager緩存中無此對象，且此對象在資料庫查無資料，則會執行insert；若有資料，則會執行update
		// insert則會新增新的對象插入至數據庫
		T02_User userAfter = entityManager.merge(userBefore);
		entityTransaction.commit();
	
		System.out.println(userBefore == userAfter); // false
	}
	```
- 若傳入游離對象，在`EntityManager`緩存中有此對象，會更新數據庫，並將緩存對象指向更新後對象
```java
	@Test
	public void t09_03_merge() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		EntityTransaction entityTransaction = entityManager.getTransaction();
		entityTransaction.begin();
		
		T02_User userBefore = new T02_User();
		userBefore.setUserName("anny");
		userBefore.setPwd("111222");
		userBefore.setCity("Taipei");
		userBefore.setBirthDayTime(new Date());
		userBefore.setBirthDay(new Date());
		userBefore.setBirthTime(new Date());
		userBefore.setId(302);
	
		// 若傳入游離對象，在EntityManager緩存中有此對象，會更新數據庫，並將緩存對象指向更新後對象
		T02_User userSelect = entityManager.find(T02_User.class, 302); // 存入緩存
		T02_User userAfter = entityManager.merge(userBefore);
		entityTransaction.commit();
	
		System.out.println(userBefore == userAfter); // false
		System.out.println(userSelect == userAfter); // true
	}
	```

### flash()
類似於hibernate中Session的`flash()`，強制`EntityManager`中管理的所有Entity對應的資料表格與Entity的狀態同步
```java
@Test
public void t10_01_flash() {
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	EntityManager entityManager = entityManagerFactory.createEntityManager();
	EntityTransaction entityTransaction = entityManager.getTransaction();
	entityTransaction.begin();

	T02_User user = entityManager.find(T02_User.class, 302); // 存入緩存
	user.setUserName("Cindy");
	
	entityManager.flush(); // 會強制發動update指令，但數據不會更動
//		entityTransaction.commit(); // 默認情況下，提交事務才會刷新
}
```

### refresh()
類似於hibernate中Session的`refresh()`，將資料表格的更動載入Entity實例中
```java
@Test
public void t10_02_refresh() {
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	EntityManager entityManager = entityManagerFactory.createEntityManager();

	T02_User user = entityManager.find(T02_User.class, 302); // 存入緩存
	user.setUserName("Cindy");
	entityManager.refresh(user); // 此時會執行select SQL
	System.out.println(user.toString()); // 此時更新為資料庫的數據
}
```

### createQuery()、createNamedQuery()、createNativeQuery()
1. 實體類定義HQL語句，以供`createNamedQuery()`調用
	```java
	// 定義JPQL
	@NamedQueries ({
	    @NamedQuery(
	        name="QueryUserByUserName",
	        query="select u from T02_User u where u.userName=?1"),
	    @NamedQuery(
	        name="UpdateCityById",
	        query="update T02_User u set u.city=?1 where u.id=?2"
	    )
	})
	
	@Entity // 讓JPA知道這是一個實體類，也就是和數據表映射的類
	@Table(name = "jpa_t01_user") // 指定映射數據表，表名默認為類名小寫
	public class T02_User {
		// 省略
	}
	```
2. 測試方法
	```java
	@Test
	public void t11_01_query() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		
		// createQuery，使用HQL查詢
		Query query1 = entityManager.createQuery("select u from T02_User u where u.userName=?1");
		query1.setParameter(1, "mickey");
		query1.getResultList().forEach(result -> System.out.println(result.toString()));
		System.out.println("----------");
	
		// createNamedQuery，使用在實體類定義的HQL查詢
		Query query2 = entityManager.createNamedQuery("QueryUserByUserName");
		query2.setParameter(1, "mickey");
		query2.getResultList().forEach(result -> System.out.println(result.toString()));
		System.out.println("----------");
		
		// createNativeQuery，使用SQL查詢
		Query query3 = entityManager.createNativeQuery("select * from jpa_t01_user where userName=?;", T02_User.class);
		query3.setParameter(1, "mickey");
		query3.getResultList().forEach(result -> System.out.println(result.toString()));
	}
	```

## 🗄️EntityTransaction
`EntityTransaction`用於管理資源層實體管理器的事務操作
- `begin()`，啟動事務
- `commit()`，提交當前事務
- `rollback()`，回滾當前事務
- `setRollbackOnly()`，設置當前事務只能被撒銷
- `getRollbackOnly()`
- `isActive()`，查看當前事務是否啟動，事務啟動才可以調用`commit()`、`rollback()`

```java
@Test
public void t11_02_entityTransaction() {
	EntityManagerFactory entityManagerFactory = null;
	EntityManager entityManager = null;
	EntityTransaction entityTransaction = null;
	try {
		entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		entityManager = entityManagerFactory.createEntityManager();
		entityTransaction = entityManager.getTransaction();
		entityTransaction.begin(); // 啟動事務
		System.out.println(entityTransaction.isActive()); // 查看當前事務是否啟動，true

		// 省略
		
		entityTransaction.setRollbackOnly(); // 設置當前事務只能被撒銷
		System.out.println(entityTransaction.getRollbackOnly()); // true
		entityTransaction.commit(); // 提交當前事務
		System.out.println(entityTransaction.isActive()); // 查看當前事務是否啟動，false
	} catch (Exception e) {
		e.printStackTrace();
		entityTransaction.rollback(); // 回滾當前事務
	} finally {
		entityManager.close();
		entityManagerFactory.close();
	}
}
```