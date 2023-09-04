# Spring Batch ItemProcesser
## 🍃默認處理器
### 校驗處理器
將不符合要求的數據去除

1. Maven引入參數校驗框架
	```xml
	<!-- 參數校驗框架 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-validation</artifactId>
	</dependency>
	```
2. 建立實體類，並設置欄位條件
	```java
	public class T56_User {
	    private Long id;
	    @NotBlank(message = "username can't be empty")
	    private String name;
	    private Long age;
		// getter、setter、toString()省略
	}
	```
3. 設置測試資料
	```console
	1#Mickey#233  
	2#Molly#123  
	3#Tai#111  
	4#Anny#222  
	5#June#234  
	6##35
	```
4. 步驟設置校驗處理器`BeanValidatingItemProcessor`
	```java
	// 校驗處理器
	@EnableBatchProcessing
	@SpringBootApplication
	public class T56_ItemProcessorTest {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T56_ItemProcessorTest.class, args);
	    }
	
	    @Bean
	    public FlatFileItemReader t56_itemReader() {
	        return new FlatFileItemReaderBuilder<>()
	                .name("t56_itemReader")
	                .resource(new ClassPathResource("T56_ReadFile.txt"))
	                .delimited().delimiter("#")
	                .names("id", "name", "age")
	                .targetType(T56_User.class)
	                .build();
	    }
	
	    @Bean
	    public BeanValidatingItemProcessor<T56_User> t56_beanValidatingItemProcessor(){
	        BeanValidatingItemProcessor<T56_User> processor = new BeanValidatingItemProcessor<>();
	        processor.setFilter(true); // 是否將不滿足條件的數據直接丟棄
	        return processor;
	    }
	
	    @Bean
	    public ItemWriter t56_itemWriter() {
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
	    @Bean
	    public Step step01_56() {
	        return this.stepBuilderFactory.get("step_01")
	                .<T56_User, T56_User>chunk(1)
	                .reader(this.t56_itemReader())
	                .processor(this.t56_beanValidatingItemProcessor()) // 設置校驗處理器
	                .writer(this.t56_itemWriter())
	                .build();
	    }
	
	    @Bean
	    public Job job56() {
	        return this.jobBuilderFactory.get("t56_job") //
	                .incrementer(new RunIdIncrementer()) //
	                .start(this.step01_56()) //
	                .build();
	    }
	}
	```

### 適配器處理器
使用現有的處理邏輯類對數據進行處理

1. 建立實體類
	```java
	public class T57_User {
	    private Long id;
	    private String name;
	    private Long age;
		// getter、setter、toString()省略
	}
	```
2. 設置測試資料
	```console
	1#Mickey#233  
	2#Molly#123  
	3#Tai#111  
	4#Anny#222  
	5#June#234  
	```
3. 現有的處理邏輯類
	```java
	@Service
	public class T57_UserService {
	    // 將用戶名轉為大寫
	    public T57_User nameToUpperCase(T57_User user){
	        user.setName(user.getName().toUpperCase());
	        return user;
	    }
	}
	```
4. 注入處理邏輯類，並設置設置適配器處理器`ItemProcessorAdapter`
	```java
	// 適配器處理器
	@EnableBatchProcessing
	@SpringBootApplication
	public class T57_ItemProcessorTest {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	    @Autowired
	    private T57_UserService t57_userService;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T57_ItemProcessorTest.class, args);
	    }
	
	    @Bean
	    public FlatFileItemReader t57_itemReader() {
	        return new FlatFileItemReaderBuilder<>()
	                .name("t57_itemReader")
	                .resource(new ClassPathResource("T57_ReadFile.txt"))
	                .delimited().delimiter("#")
	                .names("id", "name", "age")
	                .targetType(T57_User.class)
	                .build();
	    }
	
		// 設置適配器處理器
	    @Bean
	    public ItemProcessorAdapter<T57_User, T57_User> t57_itemProcessorAdapter(){
	        ItemProcessorAdapter<T57_User, T57_User> adapter = new ItemProcessorAdapter<T57_User, T57_User>();
	        adapter.setTargetMethod("nameToUpperCase"); // 調用邏輯類方法名
	        adapter.setTargetObject(this.t57_userService); // 設置適配的邏輯類
	        return adapter;
	    }
	
	    @Bean
	    public ItemWriter t57_itemWriter() {
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
	    @Bean
	    public Step step01_57() {
	        return this.stepBuilderFactory.get("step_01")
	                .<T57_User, T57_User>chunk(1)
	                .reader(this.t57_itemReader())
	                .processor(this.t57_itemProcessorAdapter()) // 設置適配器處理器
	                .writer(this.t57_itemWriter())
	                .build();
	    }
	
	    @Bean
	    public Job job57() {
	        return this.jobBuilderFactory.get("t57_job") //
	                .incrementer(new RunIdIncrementer()) //
	                .start(this.step01_57()) //
	                .build();
	    }
	}
	```
5. 測試結果
	```console
	T57_User{id=1, name='MICKEY', age=233}
	T57_User{id=2, name='MOLLY', age=123}
	T57_User{id=3, name='TAI', age=111}
	T57_User{id=4, name='ANNY', age=222}
	T57_User{id=5, name='JUNE', age=234}
	```

### 腳本處理器
將邏輯使用js腳本實現

1. 建立js腳本
	```js
	// src\main\resources\T59_UserScript.js
	// item是約定的單詞，表示ItemReader讀出來的數據
	item.setName(item.getName().toUpperCase());  
	item;
	```
2. 設置測試資料
	```console
	1#Mickey#233  
	2#Molly#123  
	3#Tai#111  
	4#Anny#222  
	5#June#234  
	```
3. 設置設置適配器處理器`ScriptItemProcessor`，需要加載JS檔，並處理檔案中的邏輯
	```java
	// 腳本處理器
	@EnableBatchProcessing
	@SpringBootApplication
	public class T59_ItemProcessorTest3 {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T59_ItemProcessorTest3.class, args);
	    }
	
	    @Bean
	    public FlatFileItemReader t59_itemReader() {
	        return new FlatFileItemReaderBuilder<>()
	                .name("t59_itemReader")
	                .resource(new ClassPathResource("T59_ReadFile.txt"))
	                .delimited().delimiter("#")
	                .names("id", "name", "age")
	                .targetType(T59_User.class)
	                .build();
	    }
	
	    @Bean
	    public ScriptItemProcessor<T59_User, T59_User> t59_scriptItemProcessor(){
	        ScriptItemProcessor<T59_User, T59_User> processor = new ScriptItemProcessor<>();
	        processor.setScript(new ClassPathResource("T59_UserScript.js")); // 加載JS檔，並處理檔案中的邏輯
	        return processor;
	    }
	
	    @Bean
	    public ItemWriter t59_itemWriter() {
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
	    @Bean
	    public Step step01_59() {
	        return this.stepBuilderFactory.get("step_01")
	                .<T57_User, T57_User>chunk(1)
	                .reader(this.t59_itemReader())
	                .processor(this.t59_scriptItemProcessor()) // 設置腳本處理器
	                .writer(this.t59_itemWriter())
	                .build();
	    }
	
	    @Bean
	    public Job job59() {
	        return this.jobBuilderFactory.get("t59_job") //
	                .incrementer(new RunIdIncrementer()) //
	                .start(this.step01_59()) //
	                .build();
	    }
	}
	```
4. 測試結果
	```console
	T59_User{id=1, name='MICKEY', age=233}
	T59_User{id=2, name='MOLLY', age=123}
	T59_User{id=3, name='TAI', age=111}
	T59_User{id=4, name='ANNY', age=222}
	T59_User{id=5, name='JUNE', age=234}
	```

### 組合處理器
`CompositeItemProcessor`用於組合多個`ItemProcessor`，類似於過濾器鏈

```java
// 組合處理器
@EnableBatchProcessing
@SpringBootApplication
public class T60_ItemProcessorTest4 {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    public static void main(String[] args) {
        SpringApplication.run(T60_ItemProcessorTest4.class, args);
    }

    @Bean
    public FlatFileItemReader t60_itemReader() {
        return new FlatFileItemReaderBuilder<>()
                .name("t60_itemReader")
                .resource(new ClassPathResource("T60_ReadFile.txt"))
                .delimited().delimiter("#")
                .names("id", "name", "age")
                .targetType(T60_User.class)
                .build();
    }

    // 處理器一，參數校驗
    @Bean
    public BeanValidatingItemProcessor<T60_User> t60_beanValidatingItemProcessor(){
        BeanValidatingItemProcessor<T60_User> processor = new BeanValidatingItemProcessor<>();
        processor.setFilter(true);
        return processor;
    }

    // 處理器二，腳本處理器
    @Bean
    public ScriptItemProcessor<T60_User, T60_User> t60_scriptItemProcessor(){
        ScriptItemProcessor<T60_User, T60_User> processor = new ScriptItemProcessor<>();
        processor.setScript(new ClassPathResource("T59_UserScript.js")); // 加載JS檔，並處理檔案中的邏輯
        return processor;
    }

    @Bean
    public CompositeItemProcessor<T60_User, T60_User> t60_compositeItemProcessor(){
        CompositeItemProcessor<T60_User, T60_User> processor = new CompositeItemProcessor<>();
        processor.setDelegates(Arrays.asList( // 設置多組處理器
                this.t60_beanValidatingItemProcessor(),
                this.t60_scriptItemProcessor()
        ));
        return processor;
    }

    @Bean
    public ItemWriter t60_itemWriter() {
        return new ItemWriter() {
            @Override
            public void write(List list) throws Exception {
                list.forEach(System.out::println);
            }
        };
    }

    @Bean
    public Step step01_60() {
        return this.stepBuilderFactory.get("step_01")
                .<T60_User, T60_User>chunk(1)
                .reader(this.t60_itemReader())
                .processor(this.t60_compositeItemProcessor()) // 設置組合處理器
                .writer(this.t60_itemWriter())
                .build();
    }

    @Bean
    public Job job60() {
        return this.jobBuilderFactory.get("t60_job") //
                .incrementer(new RunIdIncrementer()) //
                .start(this.step01_60()) //
                .build();
    }
}
```

## 🍃自定義處理器
![[SpringBatch_07_ItemProcesser_01_自定義處理器.png]]

1. 建立自定義處理器，實作`ItemProcessor`
	```java
	public class T61_CustomizeProcessor implements ItemProcessor<T61_User, T61_User> {
	    @Override
	    public T61_User process(T61_User user) throws Exception {
	        // 自定義處理器邏輯
	        return user.getId()%2 == 0 ? user : null;
	    }
	}
	```
2. 設置自定義處理器
	```java
	// 自定義處理器
	@EnableBatchProcessing
	@SpringBootApplication
	public class T61_ItemProcessorTest5 {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T61_ItemProcessorTest5.class, args);
	    }
	
	    @Bean
	    public FlatFileItemReader t61_itemReader() {
	        return new FlatFileItemReaderBuilder<>()
	                .name("t61_itemReader")
	                .resource(new ClassPathResource("T61_ReadFile.txt"))
	                .delimited().delimiter("#")
	                .names("id", "name", "age")
	                .targetType(T61_User.class)
	                .build();
	    }
	
	    @Bean
	    public T61_CustomizeProcessor t61_customizeProcessor(){
	        return new T61_CustomizeProcessor();
	    }
	
	    @Bean
	    public ItemWriter t61_itemWriter() {
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
	    @Bean
	    public Step step01_61() {
	        return this.stepBuilderFactory.get("step_01")
	                .<T61_User, T61_User>chunk(1)
	                .reader(this.t61_itemReader())
	                .processor(this.t61_customizeProcessor()) // 設置自定義處理器
	                .writer(this.t61_itemWriter())
	                .build();
	    }
	
	    @Bean
	    public Job job61() {
	        return this.jobBuilderFactory.get("t61_job") //
	                .incrementer(new RunIdIncrementer()) //
	                .start(this.step01_61()) //
	                .build();
	    }
	}
	```
