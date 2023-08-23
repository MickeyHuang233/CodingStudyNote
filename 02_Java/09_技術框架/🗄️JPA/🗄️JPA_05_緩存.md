# JPA緩存
## 🗄️JPA一級緩存
默認情況下會啟動一級緩存，在不關閉`EntityManager`，針對同一筆資料進行多次查詢，實際上只會在資料庫查詢一次
```java
@Test
public void t17_01_firstCache() {
	EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
	EntityManager entityManager = entityManagerFactory.createEntityManager();
	
	// 默認情況下會啟動一級緩存，在不關閉EntityManager，針對同一筆資料進行多次查詢，實際上只會在資料庫查詢一次
	T17_Cache cache01 = entityManager.find(T17_Cache.class, 1);
	T17_Cache cache02 = entityManager.find(T17_Cache.class, 1);
}
```

## 🗄️JPA二級緩存
查詢的資料可跨`EntityManager`取得，默認情況下不會啟動，配置參考：https://howtodoinjava.com/hibernate/configuring-ehcache-3-with-hibernate-6/

1. Maven引入hibernate級存包
	```xml
	<!-- 使用ehcache實現緩存 -->
	<dependency>
		<groupId>org.ehcache.modules</groupId>
		<artifactId>ehcache-core</artifactId>
		<version>3.10.4</version>
	</dependency>
	<dependency>
		<groupId>org.ehcache</groupId>
		<artifactId>ehcache</artifactId>
		<version>3.9.11</version>
	</dependency>
	<!-- hibernate緩存相關依賴 -->
	<dependency>
		<groupId>org.hibernate</groupId>
		<artifactId>hibernate-ehcache</artifactId>
		<version>5.2.2.Final</version>
	</dependency>
	<dependency>
		<groupId>org.hibernate.orm</groupId>
		<artifactId>hibernate-jcache</artifactId>
		<version>6.2.5.Final</version>
	</dependency>
	<!-- 從Java11開始，JAXB已從JDK發行版中刪除，用於解析ehcache.xml -->
	<dependency>
		<groupId>javax.xml.bind</groupId>
		<artifactId>jaxb-api</artifactId>
		<version>2.1</version>
	</dependency>
	<dependency>
		<groupId>com.sun.xml.bind</groupId>
		<artifactId>jaxb-impl</artifactId>
		<version>2.0.1</version>
	</dependency>
	```
2. persistence.xml配置，`shared-cache-mode`，二級緩存策略，可參考官方文檔：[Hibernate_User_Guide caching-mappings](https://docs.jboss.org/hibernate/orm/6.2/userguide/html_single/Hibernate_User_Guide.html#caching-mappings)
	- ALL，所有實體類都需要緩存
	- NONE，所有實體類都不需要緩存
	- DISABLE_SELECTIVE，標識@Cacheable(true)的實體類需要緩存
	- ENABLE_SELECTIVE，除標識@Cacheable(false)的實體類需要緩存
	- UNSPECIFIED，默認
	```xml
	<persistence xmlns="http://java.sun.com/xml/ns/persistence"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
		version="2.0">
		<persistence-unit name="myUnit" transaction-type="RESOURCE_LOCAL">
	        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
			
			<!-- 
				二級緩存策略配置
				ALL，所有實體類都需要緩存
				NONE，所有實體類都不需要緩存
				DISABLE_SELECTIVE，默認，標識@Cacheable(true)的實體類需要緩存
				ENABLE_SELECTIVE，除標識@Cacheable(false)的實體類需要緩存
			-->
			<shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
			
			<properties>
				<property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
				<property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://192.168.56.100:3306/testdb?useSSL=false" />
				<property name="jakarta.persistence.jdbc.user" value="mickey" />
				<property name="jakarta.persistence.jdbc.password" value="mickey" />
	
				<property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect" />
				<property name="hibernate.hbm2ddl.auto" value="update" />
				<property name="hibernate.show_sql" value="true" />
				
				<!-- 啟動二級緩存 -->
				<property name="hibernate.cache.use_second_level_cache" value="true"/>
				<property name="hibernate.cache.use_query_cache" value="true"/>
				<property name="hibernate.generate_statistics" value="true"></property>
				<!-- 指定緩存工廠 -->
				<property name="hibernate.cache.region.factory_class" value="jcache"/>
				<!-- ehcache緩存配置 -->
				<property name="hibernate.javax.cache.provider" value="org.ehcache.jsr107.EhcacheCachingProvider"/>
				<property name="hibernate.javax.cache.uri" value="ehcache.xml"/>
			</properties>
		</persistence-unit>
	</persistence>
	```
3. 新建src/main/resources/ehcache.xml，配置說明可參考官方文檔：[Ehcache 3.0 sample](https://www.ehcache.org/documentation/3.0/examples.html)
	```xml
	<ehcache:config xmlns:ehcache="http://www.ehcache.org/v3"
		xmlns:jcache="http://www.ehcache.org/v3/jsr107">
		<ehcache:service>
			<jcache:defaults>
				<jcache:cache name="invoices"
					template="myDefaultTemplate" />
			</jcache:defaults>
		</ehcache:service>
		<ehcache:cache-template name="myDefaultTemplate">
			<ehcache:expiry>
				<ehcache:none />
			</ehcache:expiry>
		</ehcache:cache-template>
	</ehcache:config>
	```
4. 實體類配置`@Cacheable`啟動緩存
	```java
	@Entity
	@Table(name = "jpa_t17_cache")
	@Cacheable // 實例對象啟動二級緩存
	@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
	public class T17_Cache {
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;
		
		private String cacheName;
		
		@Temporal(TemporalType.TIMESTAMP)
		private Date createDate;
	}
	```
5. 測試程式
	```java
	@BeforeEach
	public void init() {
		this.entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		this.entityManager = entityManagerFactory.createEntityManager();
		this.entityTransaction = entityManager.getTransaction();
		this.entityTransaction.begin();
	}
	
	@Test
	public void t17_02_secondCache() {
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("myUnit");
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		
		T17_Cache cache01 = entityManager.find(T17_Cache.class, 1);
		
		entityManager.clear();
		
		// 重啟EntityManager後，若無在persistence.xml配置二級緩存，則會在資料庫查詢
		T17_Cache cache02 = entityManager.find(T17_Cache.class, 1);
	}
	
	@AfterEach
	public void destory() {
		this.entityTransaction.commit();
		this.entityManager.close();
		this.entityManagerFactory.close();
	}
	```
6. 執行結果，只會出現一次查詢SQL
	```console
	Hibernate: select t1_0.id,t1_0.cacheName,t1_0.createDate from jpa_t17_cache t1_0 where t1_0.id=?
	```
