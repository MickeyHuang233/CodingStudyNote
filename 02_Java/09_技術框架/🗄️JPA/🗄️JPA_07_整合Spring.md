# JPA整合Spring
## 🗄️Maven引入依賴
```xml
<properties>
	<maven.compiler.target>12</maven.compiler.target>
	<maven.compiler.source>12</maven.compiler.source>
</properties>

<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.5.7</version>
</parent>

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.32</version>
		<scope>runtime</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

## 🗄️application.properties
```properties
spring.datasource.username=mickey
spring.datasource.password=mickey
spring.datasource.url=jdbc:mysql://192.168.56.100:3306/testdb?useUnicode=true&characterEncoding=UTF-8&rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.database-platform=org.hibernate.dialect.MariaDB53Dialect
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
```

## 🗄️實體類
```java
@Entity
@Table(name = "jpa_spring_user")
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	@Column(nullable = false)
	private String userName;

	@Column(name = "password", nullable = false)
	private String pwd;

	@Column(nullable = false)
	private String city;

	@Temporal(TemporalType.TIMESTAMP)
	private Date createDate;
	// 省略getter、setter、toString()
}
```

## 🗄️DAO
```java
@Repository
public interface UserDao extends JpaRepository<User, Long>{

}
```

## 🗄️SpringBoot啟動類
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 🗄️測試類
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class HelloWorldTest {
	Logger logger = Logger.getLogger(HelloWorldTest.class);
	
	@Autowired
	private UserDao userDao;
	
	@Test
	public void jpaTest() {
		User user = new User();
		user.setUserName("Mickey");
		user.setPwd("123456");
		user.setCity("Taipei");
		user.setCreateDate(new Date());
		userDao.save(user);
	}
}
```

## 🗄️測試Log
```console
2023-08-04 14:37:32.572  INFO 708 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-08-04 14:37:32.809  INFO 708 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-08-04 14:37:32.830  INFO 708 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.MariaDB53Dialect
Hibernate: drop table if exists jpa_spring_user
Hibernate: create table jpa_spring_user (id integer not null auto_increment, city varchar(255) not null, create_date datetime(6), password varchar(255) not null, user_name varchar(255) not null, primary key (id)) engine=InnoDB
2023-08-04 14:37:33.558  INFO 708 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2023-08-04 14:37:33.572  INFO 708 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2023-08-04 14:37:34.107  INFO 708 --- [           main] test.HelloWorldTest                      : Started HelloWorldTest in 4.065 seconds (JVM running for 5.508)
Hibernate: insert into jpa_spring_user (city, create_date, password, user_name) values (?, ?, ?, ?)
2023-08-04 14:37:34.586  INFO 708 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2023-08-04 14:37:34.594  INFO 708 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2023-08-04 14:37:34.620  INFO 708 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

## 🗄️目錄結構
![[JPA_07_整合Spring_01_目錄結構.png]]



