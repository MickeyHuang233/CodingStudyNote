# JPA_04_映射關聯關系
## 🗄️單向多對一
### 實體類映射關系
![JPA_04_映射關聯關系_01_單向多對一](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_04_%E6%98%A0%E5%B0%84%E9%97%9C%E8%81%AF%E9%97%9C%E7%B3%BB_01_%E5%96%AE%E5%90%91%E5%A4%9A%E5%B0%8D%E4%B8%80.png?raw=true)

- 1端
	```java
	@Entity
	@Table(name = "jpa_t12_customer")
	public class T12_Customer {
		// id|createDayTime|customerName|
		// --+-------------+------------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String customerName;
	
		@Temporal(TemporalType.TIMESTAMP)
		private Date createDayTime;
		// 省略getter、setter、toString()
	}
	```
- N端
	```java
	@Entity
	@Table(name = "jpa_t12_order")
	public class T12_Order {
		//	customerId|id|orderName|
		//	----------+--+---------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String orderName;
	
		@JoinColumn(name = "customerId") // 映射外鍵欄位名
	//	@ManyToOne // 映射多對一的關聯關系
		@ManyToOne(fetch = FetchType.LAZY) // 映射多對一的關聯關系，懶加載
		private T12_Customer customer;
		// 省略getter、setter、toString()
	}
	```

### insert
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t12_01_insert() {
		// 建立單筆顧客信息
		T12_Customer customer = new T12_Customer();
		customer.setCustomerName("Mickey");
		customer.setCreateDayTime(new Date());
		
		// 建立多筆訂單，並關聯顧客
		T12_Order order1 = new T12_Order();
		order1.setOrderName("Order_01");
		order1.setCustomer(customer);
		T12_Order order2 = new T12_Order();
		order2.setOrderName("Order_02");
		order2.setCustomer(customer);
		
		// 執行保存，保存1:N關聯時，建議先保存1端對象，再保存N端對象，以避免執行多余update語句
		this.entityManager.persist(customer);
		this.entityManager.persist(order1);
		this.entityManager.persist(order2);
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
	insert into jpa_t12_customer (createDayTime,customerName) values (?,?)
	insert into jpa_t12_order (customerId,orderName) values (?,?)
	insert into jpa_t12_order (customerId,orderName) values (?,?)
	```

### select
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t12_02_select() {
		T12_Order order = this.entityManager.find(T12_Order.class, 1);
		System.out.println(order.toString());
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- `@ManyToOne`，默認情況下，使用左外連接的方式取得N端對象和其關聯的1端對象
	```sql
	select t1_0.id,c1_0.id,c1_0.createDayTime,c1_0.customerName,t1_0.orderName 
	from jpa_t12_order t1_0 
	left join jpa_t12_customer c1_0 
	on c1_0.id=t1_0.customerId 
	where t1_0.id=?
	```
- `@ManyToOne(fetch = FetchType.LAZY)`，懶加載會分別執行兩次查詢
	```sql
	select t1_0.id,t1_0.customerId,t1_0.orderName from jpa_t12_order t1_0 where t1_0.id=?
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t12_customer t1_0 where t1_0.id=?
	```

### delete
因為有關聯，無法刪除1端的資料
```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t12_03_delete() {
	// 刪除N端數據
//		T12_Order order = this.entityManager.find(T12_Order.class, 1);
//		entityManager.remove(order);
	
	// 因為有關聯，無法刪除1端的資料
	// java.sql.SQLIntegrityConstraintViolationException: Cannot delete or update a parent row: a foreign key constraint fails
	T12_Customer customer = this.entityManager.find(T12_Customer.class, 1);
	entityManager.remove(customer);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

### update
可以通過N端更新1端的數據
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t12_04_update() {
		// 通過N端更新1端的數據
		T12_Order order = this.entityManager.find(T12_Order.class, 1);
		order.getCustomer().setCustomerName("Merry");
		this.entityManager.persist(order);
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
	select t1_0.id,t1_0.customerId,t1_0.orderName from jpa_t12_order t1_0 where t1_0.id=?
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t12_customer t1_0 where t1_0.id=?
	update jpa_t12_customer set createDayTime=?,customerName=? where id=?
	```

## 🗄️單向一對多
### 實體類映射關系
![JPA_04_映射關聯關系_02_單向一對多](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_04_%E6%98%A0%E5%B0%84%E9%97%9C%E8%81%AF%E9%97%9C%E7%B3%BB_02_%E5%96%AE%E5%90%91%E4%B8%80%E5%B0%8D%E5%A4%9A.png?raw=true)

- 1端
	```java
	@Entity
	@Table(name = "jpa_t13_customer")
	public class T13_Customer {
	//	id|createDayTime|customerName|
	//	--+-------------+------------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String customerName;
	
		@Temporal(TemporalType.TIMESTAMP)
		private Date createDayTime;
	
		@JoinColumn(name = "customerId") // 映射外鍵欄位名
	//	@OneToMany // 映射一對多的關聯關系
	//	@OneToMany(fetch = FetchType.LAZY) // 映射一對多的關聯關系，懶加載
		@OneToMany(cascade = CascadeType.REMOVE) // 可設置若1端刪除，則對應的N端也一并刪除
		private Set<T13_Order> orders = new HashSet<>();
		// 省略getter、setter、toString()
	}
	```
- N端
	```java
	@Entity
	@Table(name = "jpa_t13_order")
	public class T13_Order {
	//	customerId|id|orderName|
	//	----------+--+---------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String orderName;
		// 省略getter、setter、toString()
	}
	```

### insert
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t13_01_insert() {
		// 建立多筆訂單
		T13_Order order1 = new T13_Order();
		order1.setOrderName("Order_01");
		T13_Order order2 = new T13_Order();
		order2.setOrderName("Order_02");
	
		// 建立單筆顧客信息，並關聯訂單
		T13_Customer customer = new T13_Customer();
		customer.setCustomerName("Mickey");
		customer.setCreateDayTime(new Date());
		HashSet<T13_Order> orders = new HashSet<T13_Order>();
		orders.add(order1);
		orders.add(order2);
		customer.setOrders(orders);
		
		// 執行保存
		this.entityManager.persist(customer);
		this.entityManager.persist(order1);
		this.entityManager.persist(order2);
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
	insert into jpa_t13_customer (createDayTime,customerName) values (?,?)
	insert into jpa_t13_order (orderName) values (?)
	insert into jpa_t13_order (orderName) values (?)
	update jpa_t13_order set customerId=? where id=?
	update jpa_t13_order set customerId=? where id=?
	```

### select
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t13_02_select() {
		T13_Customer customer = this.entityManager.find(T13_Customer.class, 1);
		System.out.println(customer.toString());
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 執行SQL語句，無論是否用懶加載，都要執行兩次查詢語句
	```sql
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t13_customer t1_0 where t1_0.id=?
	select o1_0.customerId,o1_0.id,o1_0.orderName from jpa_t13_order o1_0 where o1_0.customerId=?
	```

### delete
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t12_03_delete() {
		// 刪除1端數據，會先將N端的外鍵欄位更新為null後再刪除
		T13_Customer customer = this.entityManager.find(T13_Customer.class, 1);
		entityManager.remove(customer);
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- 刪除1端數據，默認會先將N端的外鍵欄位更新為null後再刪除
	```sql
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t13_customer t1_0 where t1_0.id=?
	update jpa_t13_order set customerId=null where customerId=?
	delete from jpa_t13_customer where id=?
	```
- 可通過`@OneToMany`的`cascade`屬性修改刪除策略
	```sql
	-- @OneToMany(cascade = CascadeType.REMOVE)
	-- 若1端刪除，則對應的N端也一并刪除
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t13_customer t1_0 where t1_0.id=?
	select o1_0.customerId,o1_0.id,o1_0.orderName from jpa_t13_order o1_0 where o1_0.customerId=?
	update jpa_t13_order set customerId=null where customerId=?
	delete from jpa_t13_order where id=?
	delete from jpa_t13_order where id=?
	delete from jpa_t13_customer where id=?
	```

### update
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t13_04_update() {
		// 通過1端更新N端的數據
		T13_Customer customer = this.entityManager.find(T13_Customer.class, 3);
		customer.getOrders().stream().forEach(order -> order.setOrderName("testName"));
		this.entityManager.persist(customer);
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
	select t1_0.id,t1_0.createDayTime,t1_0.customerName from jpa_t13_customer t1_0 where t1_0.id=?
	select o1_0.customerId,o1_0.id,o1_0.orderName from jpa_t13_order o1_0 where o1_0.customerId=?
	update jpa_t13_order set orderName=? where id=?
	update jpa_t13_order set orderName=? where id=?
	```

## 🗄️雙向多對一
### 實體類映射關系
![JPA_04_映射關聯關系_03_雙向多對一](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_04_%E6%98%A0%E5%B0%84%E9%97%9C%E8%81%AF%E9%97%9C%E7%B3%BB_03_%E9%9B%99%E5%90%91%E5%A4%9A%E5%B0%8D%E4%B8%80.png?raw=true)

- 1端
	```java
	@Entity
	@Table(name = "jpa_t14_customer")
	public class T14_Customer {
	//	id|createDayTime|customerName|
	//	--+-------------+------------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String customerName;
	
		@Temporal(TemporalType.TIMESTAMP)
		private Date createDayTime;
	
		@JoinColumn(name = "customerId") // 映射外鍵欄位名
		@OneToMany // 映射一對多的關聯關系，兩邊外鍵欄位名需要一致
	//	@OneToMany(fetch = FetchType.LAZY) // 映射一對多的關聯關系，懶加載
		private Set<T14_Order> orders = new HashSet<>();
		// 省略getter、setter、toString()
	}
	```
- N端
	```java
	@Entity
	@Table(name = "jpa_t14_order")
	public class T14_Order {
	//	customerId|id|orderName|
	//	----------+--+---------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
	
		private String orderName;
	
		@JoinColumn(name = "customerId") // 映射外鍵欄位名，兩邊外鍵欄位名需要一致
		@ManyToOne // 映射多對一的關聯關系
	//	@ManyToOne(fetch = FetchType.LAZY) // 映射多對一的關聯關系，懶加載
		private T14_Customer customer;
		// 省略getter、setter、toString()
	}
	```

### insert
可分別使用[[🗄️JPA_04_映射關聯關系#🗄️單向多對一#insert]]和[[🗄️JPA_04_映射關聯關系#🗄️單向一對多#insert]]的方式添加資料
```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t14_01_insert() {
	// 建立單筆顧客信息
	T14_Customer customer = new T14_Customer();
	customer.setCustomerName("Mickey");
	customer.setCreateDayTime(new Date());
	
	// 建立多筆訂單，並關聯顧客
	T14_Order order1 = new T14_Order();
	order1.setOrderName("Order_01");
	order1.setCustomer(customer);
	T14_Order order2 = new T14_Order();
	order2.setOrderName("Order_02");
	order2.setCustomer(customer);
	
	// 執行保存
	this.entityManager.persist(customer);
	this.entityManager.persist(order1);
	this.entityManager.persist(order2);
}

@Test
public void t14_02_insert() {
	// 建立多筆訂單
	T14_Order order1 = new T14_Order();
	order1.setOrderName("Order_01");
	T14_Order order2 = new T14_Order();
	order2.setOrderName("Order_02");

	// 建立單筆顧客信息，並關聯訂單
	T14_Customer customer = new T14_Customer();
	customer.setCustomerName("Mickey");
	customer.setCreateDayTime(new Date());
	HashSet<T14_Order> orders = new HashSet<T14_Order>();
	orders.add(order1);
	orders.add(order2);
	customer.setOrders(orders);
	
	// 執行保存
	this.entityManager.persist(customer);
	this.entityManager.persist(order1);
	this.entityManager.persist(order2);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

## 🗄️雙向一對一
### 實體類映射關系
![JPA_04_映射關聯關系_04_雙向一對一](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_04_%E6%98%A0%E5%B0%84%E9%97%9C%E8%81%AF%E9%97%9C%E7%B3%BB_04_%E9%9B%99%E5%90%91%E4%B8%80%E5%B0%8D%E4%B8%80.png?raw=true)

- 維護外鍵一端
	```java
	@Entity
	@Table(name = "jpa_t15_department")
	public class T15_Department {
	//	employeeId|id|departmentName|
	//	----------+--+--------------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
		
		private String departmentName;
		
		// 兩張表只要有一張表有標示@JoinColumn就可以
		// 1對1關系需要添加unique = true
		@JoinColumn(name = "employeeId", unique = true)
		// @OneToOne // 映射1對1關聯關系
		@OneToOne(fetch = FetchType.LAZY) // 映射1對1關聯關系，懶加載
		private T15_Employee employee;
		// 省略getter、setter、toString()
	}
	```
- 不維護外鍵另一端
	```java
	@Entity
	@Table(name = "jpa_t15_employee")
	public class T15_Employee {
	//	id|EmployeeName|
	//	--+------------+
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
		
		private String EmployeeName;
		
		@OneToOne(mappedBy = "employee") // 指定對方的關系維護屬性
		private T15_Department department;
		// 省略getter、setter、toString()
	}
	```

### insert
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t14_01_insert() {
		// 建立員工信息
		T15_Employee employee = new T15_Employee();
		employee.setEmployeeName("Monkey");
		
		// 建立部門信息
		T15_Department department = new T15_Department();
		department.setDepartmentName("Marketing");
		
		// 關聯兩邊資料
		employee.setDepartment(department);
		department.setEmployee(employee);
		
		// 執行保存，建議先保存不維護外鍵的一方，以避免執行多余的update語法
		this.entityManager.persist(employee);
		this.entityManager.persist(department);
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
	insert into jpa_t15_department (departmentName,employeeId) values (?,?)
	insert into jpa_t15_employee (EmployeeName) values (?)
	-- 建議先保存不維護外鍵的一方，以避免執行多余的update語法
	-- update jpa_t15_department set departmentName=?,employeeId=? where id=?
	```

### select
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t15_02_select() {
		// 默認情況下，會使用左外連接取得關連對象
	//		T15_Employee employee = this.entityManager.find(T15_Employee.class, 2);
	//		System.out.println(employee.toString());
		
		T15_Department department = this.entityManager.find(T15_Department.class, 2);
		System.out.println(department.toString());
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
- `@OneToOne`，默認情況下，會使用左外連接取得關連對象
	```sql
	select t1_0.id,t1_0.EmployeeName,d1_0.id,d1_0.departmentName 
	from jpa_t15_employee t1_0 
	left join jpa_t15_department d1_0 
	on t1_0.id=d1_0.employeeId 
	where t1_0.id=?
	```
- `@OneToOne(fetch = FetchType.LAZY)`，懶加載會分別執行兩次查詢
	```sql
	select t1_0.id,t1_0.departmentName,t1_0.employeeId from jpa_t15_department t1_0 where t1_0.id=?
	select t1_0.id,t1_0.EmployeeName,d1_0.id,d1_0.departmentName from jpa_t15_employee t1_0 left join jpa_t15_department d1_0 on t1_0.id=d1_0.employeeId where t1_0.id=?
	```

### delete
建議從維護外鍵的對象進行操作

```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t15_03_delete() {
	// 刪除不維護外鍵一端數據，因為有關聯關系，無法刪除
	// jakarta.persistence.RollbackException: Error while committing the transaction
//		T15_Employee employee = this.entityManager.find(T15_Employee.class, 1);
//		entityManager.remove(employee);
	
	// 刪除維護外鍵一端數據
	T15_Department department = this.entityManager.find(T15_Department.class, 1);
	entityManager.remove(department);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

### update
建議從維護外鍵的對象進行操作

```java
@BeforeEach
public void init() {
	this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	this.entityManager = entityManagerFactory.createEntityManager();
	this.entityTransaction = entityManager.getTransaction();
	this.entityTransaction.begin();
}

@Test
public void t15_04_update() {
	// 通過不維護外鍵的表修改數據
	T15_Employee employee = this.entityManager.find(T15_Employee.class, 2);
	employee.getDepartment().setDepartmentName("BOSS");
	this.entityManager.persist(employee);
	
	// 通過維護外鍵的表修改數據
	T15_Department department = this.entityManager.find(T15_Department.class, 2);
	department.getEmployee().setEmployeeName("Mickey");
	this.entityManager.persist(department);
}

@AfterEach
public void destory() {
	this.entityTransaction.commit();
	this.entityManager.close();
	this.entityManagerFactory.close();
}
```

## 🗄️雙向多對多
### 實體類映射關系
![JPA_04_映射關聯關系_05_雙向多對多](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_04_%E6%98%A0%E5%B0%84%E9%97%9C%E8%81%AF%E9%97%9C%E7%B3%BB_05_%E9%9B%99%E5%90%91%E5%A4%9A%E5%B0%8D%E5%A4%9A.png?raw=true)

- 維護外鍵一端
	```java
	@Entity
	@Table(name = "jpa_t16_employee")
	public class T16_Employee {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
		
		private String employeeName;
		
		@JoinTable(name="jpa_t16_employee_work", // 中間表名
			joinColumns = {@JoinColumn(name = "employeeId", referencedColumnName = "id")}, // 設置維護主鍵，指向當表的主鍵
			inverseJoinColumns = {@JoinColumn(name = "worksId", referencedColumnName = "id")}) // 關聯的對象id，指向關聯表的主鍵
		// @ManyToMany // 映射多對多的關聯關系
		@ManyToMany(cascade = CascadeType.REMOVE) // 設置將沒關聯的資料刪除
		private Set<T16_Work> works = new HashSet<>();
		// 省略getter、setter、toString()
	}
	```
- 不維護外鍵另一端
	```java
	@Entity
	@Table(name = "jpa_t16_work")
	public class T16_Work {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
		
		private String workName;
		
		@ManyToMany(mappedBy = "works") // 指定對方的關系維護屬性
		private Set<T16_Employee> employees = new HashSet<>();
		// 省略getter、setter、toString()
	}
	```

### insert
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t16_01_insert() {
		// 建立員工信息
		T16_Employee employee1 = new T16_Employee();
		employee1.setEmployeeName("Monkey");
		T16_Employee employee2 = new T16_Employee();
		employee2.setEmployeeName("Mickey");
		
		// 建立工作信息
		T16_Work work1 = new T16_Work();
		work1.setWorkName("work01");;
		T16_Work work2 = new T16_Work();
		work2.setWorkName("work02");;
		
		// 關聯兩邊資料
		Set<T16_Work> works1 = new HashSet<>();
		works1.add(work1);
		works1.add(work2);
		Set<T16_Work> works2 = new HashSet<>();
		works2.add(work1);
		employee1.setWorks(works1);
		employee2.setWorks(works2);
		
		Set<T16_Employee> employees1 = new HashSet<>();
		employees1.add(employee1);
		employees1.add(employee2);
		Set<T16_Employee> employees2 = new HashSet<>();
		employees2.add(employee1);
		work1.setEmployees(employees1);
		work2.setEmployees(employees2);
		
		// 執行保存，建議先保存不維護外鍵的一方，以避免執行多余的update語法
		this.entityManager.persist(employee1);
		this.entityManager.persist(employee2);
		this.entityManager.persist(work1);
		this.entityManager.persist(work2);
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
	insert into jpa_t16_employee (employeeName) values (?)
	insert into jpa_t16_employee (employeeName) values (?)
	insert into jpa_t16_work (workName) values (?)
	insert into jpa_t16_work (workName) values (?)
	insert into jpa_t16_employee_work (employeeId,worksId) values (?,?)
	insert into jpa_t16_employee_work (employeeId,worksId) values (?,?)
	insert into jpa_t16_employee_work (employeeId,worksId) values (?,?)
	```

### select
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t16_02_select() {
		// 從維護外鍵表查詢
	//		T16_Employee employee = this.entityManager.find(T16_Employee.class, 1);
	//		System.out.println(employee.toString());
		
		// 從不維護外鍵表查詢
		T16_Work work = this.entityManager.find(T16_Work.class, 1);
		System.out.println(work.toString());
		work.getEmployees().forEach(employee -> System.out.println(employee.toString()));
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
	-- 從維護外鍵表查詢
	select t1_0.id,t1_0.employeeName from jpa_t16_employee t1_0 where t1_0.id=?
	
	select w1_0.employeeId,w1_1.id,w1_1.workName 
	from jpa_t16_employee_work w1_0 
	join jpa_t16_work w1_1 
	on w1_1.id=w1_0.worksId 
	where w1_0.employeeId=?
	
	-- 從不維護外鍵表查詢
	select t1_0.id,t1_0.workName from jpa_t16_work t1_0 where t1_0.id=?
	select e1_0.worksId,e1_1.id,e1_1.employeeName from jpa_t16_employee_work e1_0 join jpa_t16_employee e1_1 on e1_1.id=e1_0.employeeId where e1_0.worksId=?
	select w1_0.employeeId,w1_1.id,w1_1.workName from jpa_t16_employee_work w1_0 join jpa_t16_work w1_1 on w1_1.id=w1_0.worksId where w1_0.employeeId=?
	select w1_0.employeeId,w1_1.id,w1_1.workName from jpa_t16_employee_work w1_0 join jpa_t16_work w1_1 on w1_1.id=w1_0.worksId where w1_0.employeeId=?
	```

### delete
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t16_03_delete() {
		// 刪除不維護外鍵一端數據，因為有關聯關系，無法刪除
		// jakarta.persistence.RollbackException: Error while committing the transaction
	//		T16_Work work = this.entityManager.find(T16_Work.class, 1);
	//		this.entityManager.remove(work);
		
		// 刪除維護外鍵一端數據
		T16_Employee employee = this.entityManager.find(T16_Employee.class, 2);
		entityManager.remove(employee);
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
	-- 刪除不維護外鍵一端數據
	select t1_0.id,t1_0.workName from jpa_t16_work t1_0 where t1_0.id=?
	delete from jpa_t16_work where id=?
	
	-- 刪除維護外鍵一端數據
	select t1_0.id,t1_0.employeeName from jpa_t16_employee t1_0 where t1_0.id=?
	delete from jpa_t16_employee_work where employeeId=?
	delete from jpa_t16_employee where id=?
	```

### update
- 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t16_04_update() {
		// 通過不維護外鍵的表修改數據
		T16_Work work = this.entityManager.find(T16_Work.class, 3);
		work.getEmployees().forEach(e -> e.setEmployeeName("testEmployee"));
		this.entityManager.persist(work);
		
		// 通過維護外鍵的表修改數據
	//		T16_Employee employee = this.entityManager.find(T16_Employee.class, 3);
	//		employee.getWorks().forEach(w -> w.setWorkName("testWork"));
	//		this.entityManager.persist(employee);
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
	select t1_0.id,t1_0.workName from jpa_t16_work t1_0 where t1_0.id=?
	select e1_0.worksId,e1_1.id,e1_1.employeeName from jpa_t16_employee_work e1_0 join jpa_t16_employee e1_1 on e1_1.id=e1_0.employeeId where e1_0.worksId=?
	update jpa_t16_employee set employeeName=? where id=?
	update jpa_t16_employee set employeeName=? where id=?
	
	select t1_0.id,t1_0.employeeName from jpa_t16_employee t1_0 where t1_0.id=?
	select w1_0.employeeId,w1_1.id,w1_1.workName from jpa_t16_employee_work w1_0 join jpa_t16_work w1_1 on w1_1.id=w1_0.worksId where w1_0.employeeId=?
	update jpa_t16_work set workName=? where id=?
	update jpa_t16_work set workName=? where id=?
	```