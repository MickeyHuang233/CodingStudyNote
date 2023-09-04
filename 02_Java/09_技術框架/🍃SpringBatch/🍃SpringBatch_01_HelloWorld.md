# Spring Batch HelloWorld
## 🍃信息持久化存入內存
如：H2、HSQLDB，這些數據庫將數據緩存至內存中，當批處理結束後就會被清除，一般用於單元測試

### Maven引入
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.5.7</version>
</parent>

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-batch</artifactId>
	</dependency>
	<!-- in memory database -->
	<dependency>
		<groupId>com.h2database</groupId>
		<artifactId>h2</artifactId>
	</dependency>
</dependencies>
```

### 啟動類
```java
@SpringBootApplication // 設置為啟動類    
// 啟動SpringBatch相關注解，要求配置在配置類或啟動類  
// 配置完成後，SpringBoot會自動加載JobLauncher、JobBuilderFactory、StepBuilderFactory並交由容器管理，需要使用直接注入即可  
@EnableBatchProcessing
public class App {  
    public static void main(String[] args) {  
        SpringApplication.run(App.class, args);  
    }  
  
    // Job啟動器，由Spring Boot進行調用  
    @Autowired  
    private JobLauncher jobLauncher;  
  
    // Job構造器工廠  
    @Autowired  
    private JobBuilderFactory jobBuilderFactory;  
  
    // Step構造器工廠  
    @Autowired  
    private StepBuilderFactory stepBuilderFactory;  
  
    // Tasklet，Step中具體執行的邏輯  
    @Bean  
    public Tasklet tasklet() {  
        return new Tasklet() {  
            @Override  
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {  
                System.out.println("Do Something in Tasklet");  
                return RepeatStatus.FINISHED; // 表示執行完畢
                // return RepeatStatus.CONTINUABLE; // 表示還要重覆執行此Step
            }  
        };  
    }  
  
    @Bean  
    public Step step01() {  
        return this.stepBuilderFactory.get("step_01") //  
                .tasklet(this.tasklet()) //  
                .build();  
    }  
    
	// 1. SpringBoot啟動會去從容器加載Job對象  
	// 2. 將此Job交由JobLauncherApplicationRunner  
	// 3. JobLuncher實現Job的執行
    @Bean  
    public Job job() {  
        return this.jobBuilderFactory.get("t01_job") //  
                .start(this.step01()) //  
                .build();  
    }  
}
```

## 🍃信息持久化存入數據庫
### 建立DB環境
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
	- Host：192.168.56.100
	- Port：3306
	- Database：testdb
	- Username：mickey
	- Password：mickey

### Maven引入
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.5.7</version>
</parent>

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-batch</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.32</version>
		<scope>runtime</scope>
	</dependency>
</dependencies>
```

### 建立persistence.xml配置檔
```properties
# DB連線信息  
spring.datasource.username=mickey  
spring.datasource.password=mickey  
spring.datasource.url=jdbc:mysql://192.168.56.100:3306/testdb?useUnicode=true&characterEncoding=UTF-8&rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai  
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver  
  
# 初始化數據庫，文件放在依賴jar中  
spring.sql.init.schema-locations: classpath:org/springframework/batch/core/schema-mysql.sql  
# 第一次執行，建立資料表  
#spring.sql.init.mode: always  
spring.sql.init.mode: never
```

### 啟動類
同[[🍃SpringBatch_01_HelloWorld#🍃信息持久化存入內存#啟動類]]



