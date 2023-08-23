# JPA_01_HelloWorld
JPA + SpringBoot，可參考：[[🍃SpringBoot1_05_數據訪問#🍃整合JPA]]

## 🗄️數據庫安裝(MySQL容器)
1. 安裝至容器
	可參考：[[🐧Linux_RH134_10_Container管理]]、[Docker官方文件](https://hub.docker.com/_/mariadb)
	```bash
	# 建立MySQL容器
	podman run -d -p 3306:3306 --name=mydb \
	-e MYSQL_USER=mickey -e MYSQL_PASSWORD=mickey -e MYSQL_DATABASE=testdb \
	registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest
	# 打開防火墻端口號，並重啟
	sudo firewall-cmd --permanent --add-port=3306/tcp --zone=public
	sudo firewall-cmd --reload
	```
2. 使用資料庫連線工具連線
	![JPA_01_HelloWorld_01_資料庫連線工具參數](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_01_HelloWorld_01_%E8%B3%87%E6%96%99%E5%BA%AB%E9%80%A3%E7%B7%9A%E5%B7%A5%E5%85%B7%E5%8F%83%E6%95%B8.png?raw=true)
3. 建立資料表
	```sql
	create table jpa_t01_user(
	    id int(5) primary key auto_increment,
	    username varchar(50) not null,
	    password varchar(50) not null,
	    city varchar(20) not null
	);
	```

## 🗄️建立JPA專案
### Maven引入
```xml
<!-- DB Driver -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.32</version>
</dependency>
<!-- ORM框架 -->
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-core</artifactId>
	<version>6.2.5.Final</version>
</dependency>
```

### 建立實體類
```java
@Entity // 讓JPA知道這是一個實體類，也就是和數據表映射的類
@Table(name = "jpa_t01_user") // 指定映射數據表，表名默認為類名小寫
public class T02_User {
	@Id // 表示為主鍵
	@GeneratedValue(strategy = GenerationType.IDENTITY) // 自增主鍵
	private Integer id;
	
	@Column(nullable = false)
	private String userName;
	
	@Column(name = "password", nullable = false) // 默認欄位名默認和屬性名一致
	private String pwd;
	
	@Column(nullable = false)
	private String city;
	// 省略getter、setter
}
```

### 建立persistence.xml配置檔
- 建立路徑：java/META-INF/persistence.xml
- [persistence.xml sample - github](https://gist.github.com/erdinckocaman/11039632)

```xml
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
	version="2.0">
	<persistence-unit name="myUnit" transaction-type="RESOURCE_LOCAL">
		<!-- 
			Persistence provider，指定ORM框架作為JPA實現
			1. 就是指定jakarta.persistence.spi.PersistenceProvider實現類
			2. 如果JPA專案中只有一個ORM産品，也可以不用配置
		-->
		<!--
			hibernate版本高於4.3使用org.hibernate.jpa.HibernatePersistenceProvider
			hibernate版本低於4.2使用org.hibernate.ejb.HibernatePersistence
		-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
		
		<!--
			添加實體類
			教學有添加，但會出現Class "com.mickey.entry.T02_User" is listed in the persistence.xml file, but is not annotated
		-->
		<!--<class>com.mickey.entry.T02_User</class>-->
		
		<properties>
			<!-- DB連線資訊 -->
			<property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
			<property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://192.168.56.100:3306/testdb?useSSL=false" />
			<property name="jakarta.persistence.jdbc.user" value="mickey" />
			<property name="jakarta.persistence.jdbc.password" value="mickey" />

			<!-- 配置hibernate基本屬性 -->
			<property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect" />
			<property name="hibernate.hbm2ddl.auto" value="update" />
			<property name="hibernate.show_sql" value="true" />
		</properties>
	</persistence-unit>
</persistence>
```

### 建立測試類
```java
public class T02_HelloWorld {
	@Test
	public void helloWorld() {
		EntityManagerFactory entityManagerFactory = null;
		EntityManager entityManager = null;
		EntityTransaction entityTransaction = null;
		try {
			// 建立EntitymanagerFactory
			String persistenceUnitName = "myUnit"; // 持久化單元名稱，定義於persistence.xml中persistence-unit的name屬性
			entityManagerFactory = Persistence.createEntityManagerFactory(persistenceUnitName);
			
			// 建立EntityManager，類似於Hibernate中的SessionFactory，用於新增對象、事務管理的二級緩存
			entityManager = entityManagerFactory.createEntityManager();
			
			// 建立並啟動事務
			entityTransaction = entityManager.getTransaction();
			entityTransaction.begin();
			
			// 持久化操作
			T02_User user = new T02_User();
			user.setUserName("mickey");
			user.setPwd("123456");
			user.setCity("Taipei");
			entityManager.persist(user); // 保存持久化物備件
			
			// 提交事務
			entityTransaction.commit();
		} catch (Exception e) {
			e.printStackTrace();
			entityTransaction.rollback();
		} finally {
			// 關閉資源
			entityManager.close();
			entityManagerFactory.close();
		}
	}
}
```

## 🗄️目錄結構
![JPA_01_HelloWorld_01_目錄結構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%97%84%EF%B8%8FJPA/images/JPA_01_HelloWorld_01_%E7%9B%AE%E9%8C%84%E7%B5%90%E6%A7%8B.png?raw=true)

