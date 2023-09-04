# Spring Batch ItemReader
`ItemReader`是Spring Batch提供輸入的組件，如：文件、數據庫、JMS資源…等

```java
public interface ItemReader<T> {
    @Nullable
    T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;
}
```

## 🍃輸入純文本文件
Spring Batch默認使用`FlatFileItemReader`實現純文本文件輸入

1. 建立封裝的物件
	```java
	public class T49_User {
	    private Long id;
	    private String name;
	    private Long age;
	    private String address; // 在文本中由三個欄位組成，需自定義映射
		// getter、setter、toString()省略
	}
	```
2. 建立`FlatFileItemReaderBuilder`，並設定至Chunk，可參考：[[🍃SpringBatch_03_Step#🍃Chunk處理模式]]
	```java
	// 讀取純文本，封裝至對象中
	@EnableBatchProcessing
	@SpringBootApplication
	public class T49_ItemReaderTest {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T49_ItemReaderTest.class, args);
	    }
	
	    @Bean
	    public ItemWriter t49_itemWriter(){
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
		@Bean
		public FlatFileItemReader t49_itemReader() {
			// 自動封裝的方式，數據無需做加工直接賦值至屬性
		//     return new FlatFileItemReaderBuilder<>()
		//             .name("t49_itemReader") // 設置Reader名稱
		//             .resource(new ClassPathResource("T49_ReadFile.txt")) // 讀取文件路徑
		//             .delimited().delimiter("#") // 設置分隔符
		//             .names("id", "name", "age") // 設置欄位名稱
		//             .targetType(T49_User.class) // 讀取的數據封裝的物件類型，自動封裝
		//             .build();

		// 自定義封裝的方式
		return new FlatFileItemReaderBuilder<>()
				.name("t49_itemReader") // 設置Reader名稱
				.resource(new ClassPathResource("T49_ReadFile.txt")) // 讀取文件路徑
				.delimited().delimiter("#") // 設置分隔符
				.names("id", "name", "age", "country", "city", "road") // 設置欄位名稱
				.fieldSetMapper(new FieldSetMapper<Object>() { // 自定義封裝映射
					@Override
					public T49_User mapFieldSet(FieldSet fieldSet) throws BindException {
						T49_User user = new T49_User();
						user.setId(fieldSet.readLong("id"));
						user.setName(fieldSet.readString("name"));
						user.setAge(fieldSet.readLong("age"));
						String address =
								fieldSet.readString("country")
								+ fieldSet.readString("city")
								+ fieldSet.readString("road");
						user.setAddress(address);
						return user;
					}
				})
				.build();
		}
	
	    @Bean
	    public Step step01_49(){
	        return this.stepBuilderFactory.get("step_01") //
	                .<T49_User, T49_User>chunk(1) // 一次讀幾筆資料
	                .reader(this.t49_itemReader()) // 設置Reader
	                .writer(this.t49_itemWriter()) // 設置Writer
	                .build();
	    }
	
	    @Bean
	    public Job job49(){
	        return this.jobBuilderFactory.get("t49_job") //
	                .start(this.step01_49()) //
	                .build();
	    }
	}
	```
3. 建立測試純文本文件
	```console
	1#Mickey#233#Taiwan#Taipei#Test234  
	2#Molly#123#Taiwan#NewTaipeiCity#Abc000  
	3#Tai#111#US#NewYork#MyRoad123  
	4#Anny#222#US#NewYork#Test555  
	5#June#234#Taiwan#Taipei#Road123456
	```
4. 測試結果
	```console
	T49_User{id=1, name='Mickey', age=233, address='TaiwanTaipeiTest234'}
	T49_User{id=2, name='Molly', age=123, address='TaiwanNewTaipeiCityAbc000'}
	T49_User{id=3, name='Tai', age=111, address='USNewYorkMyRoad123'}
	T49_User{id=4, name='Anny', age=222, address='USNewYorkTest555'}
	T49_User{id=5, name='June', age=234, address='TaiwanTaipeiRoad123456'}
	```

## 🍃輸入JSON文件
使用`JsonItemReader`實現JSON文件輸入

1. 建立封裝的物件
	```java
	public class T51_User {
	    private Long id;
	    private String name;
	    private Long age;
	    private String address;
		// getter、setter、toString()省略
	}
	```
2. 建立`JsonItemReaderBuilder`，並設定至Chunk，可參考：[[🍃SpringBatch_03_Step#🍃Chunk處理模式]]
	```java
	// 讀取JSON文本，封裝至對象中
	@EnableBatchProcessing
	@SpringBootApplication
	public class T51_ItemReaderTest2 {
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    public static void main(String[] args) {
	        SpringApplication.run(T51_ItemReaderTest2.class, args);
	    }
	
	    @Bean
	    public ItemWriter t51_itemWriter() {
	        return new ItemWriter() {
	            @Override
	            public void write(List list) throws Exception {
	                list.forEach(System.out::println);
	            }
	        };
	    }
	
	    @Bean
	    public JsonItemReader t51_itemReader() {
	        return new JsonItemReaderBuilder<T51_User>()
	                .name("t51_itemReader") // 設置Reader名稱
	                .resource(new ClassPathResource("T51_ReadJsonFile.json")) // 讀取文件路徑
	                .jsonObjectReader(
	                        // 實現JsonObjectReader，可用不同的JSON解析框架，並指定目標解析類型
	//                        new GsonJsonObjectReader<>(T51_User.class)
	                        new JacksonJsonObjectReader<>(T51_User.class)
	                )
	                .build();
	    }
	
	    @Bean
	    public Step step01_51() {
	        return this.stepBuilderFactory.get("step_01") //
	                .<T51_User, T51_User>chunk(1) // 一次讀幾筆資料
	                .reader(this.t51_itemReader()) // 設置Reader
	                .writer(this.t51_itemWriter()) // 設置Writer
	                .build();
	    }
	
	    @Bean
	    public Job job51() {
	        return this.jobBuilderFactory.get("t51_job") //
	                .start(this.step01_51()) //
	                .incrementer(new RunIdIncrementer()) //
	                .build();
	    }
	}
	```
3. 建立測試JSON文件
	```json
	[  
	  {"id":1, "name":"Mickey", "age":233, "address":"TaiwanTaipeiTest234"},  
	  {"id":2, "name":"Molly", "age":123, "address":"TaiwanNewTaipeiCityAbc000"},  
	  {"id":3, "name":"Tai", "age":111, "address":"USNewYorkMyRoad123"},  
	  {"id":4, "name":"Anny", "age":222, "address":"USNewYorkTest555"},  
	  {"id":5, "name":"June", "age":234, "address":"TaiwanTaipeiRoad123456"}  
	]
	```
4. 測試結果
	```console
	T49_User{id=1, name='Mickey', age=233, address='TaiwanTaipeiTest234'}
	T49_User{id=2, name='Molly', age=123, address='TaiwanNewTaipeiCityAbc000'}
	T49_User{id=3, name='Tai', age=111, address='USNewYorkMyRoad123'}
	T49_User{id=4, name='Anny', age=222, address='USNewYorkTest555'}
	T49_User{id=5, name='June', age=234, address='TaiwanTaipeiRoad123456'}
	```

## 🍃輸入數據庫
1. 建立相應資料表
	```sql
	CREATE TABLE testdb.t52_user (
		id INT auto_increment PRIMARY KEY,
		name varchar(100) NOT NULL,
		age INT NOT NULL,
		address varchar(100) NULL
	) ENGINE = MYISAM DEFAULT CHARSET = utf8
	```
2. 設置測試資料
	```console
	id|name  |age|address                  |
	--+------+---+-------------------------+
	 1|Mickey|233|TaiwanTaipeiTest234      |
	 2|Molly |123|TaiwanNewTaipeiCityAbc000|
	 3|Tai   |111|USNewYorkMyRoad123       |
	 4|Anny  |222|USNewYorkTest555         |
	 5|June  |234|TaiwanTaipeiRoad123456   |
	```
3. 建立封裝的物件
	```java
	public class T51_User {
	    private Long id;
	    private String name;
	    private Long age;
	    private String address;
		// getter、setter、toString()省略
	}
	```

### 使用遊標方式讀取
`JdbcCursorItemReader`，單筆讀取資料

```java
// 讀取數據庫，封裝至對象中
@EnableBatchProcessing
@SpringBootApplication
public class T52_JdbcItemReaderTest {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;
    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        SpringApplication.run(T52_JdbcItemReaderTest.class, args);
    }

    @Bean
    public ItemWriter t52_itemWriter() {
        return new ItemWriter() {
            @Override
            public void write(List list) throws Exception {
                list.forEach(System.out::println);
            }
        };
    }

    @Bean
    public JdbcCursorItemReader<T52_User> t52_itemReader() {
        // 使用JDBC讀取單筆資料
        return new JdbcCursorItemReaderBuilder<T52_User>()
                .name("t52_itemReader") // 設置Reader名稱
                .dataSource(this.dataSource) // 設置數據源，與數據庫連線，Spring容器實現
                .sql("SELECT * FROM t52_user WHERE age>=?") // 通過SQL查詢數據
                .preparedStatementSetter( // 拼接參數
                        new ArgumentPreparedStatementSetter(new Object[]{200})
                )
                .rowMapper( // 數據和對象一一映射
                        new RowMapper<T52_User>() {
                            @Override
                            public T52_User mapRow(ResultSet rs, int rowNum) throws SQLException {
                                T52_User user = new T52_User();
                                user.setId(rs.getLong("id"));
                                user.setName(rs.getString("name"));
                                user.setAge(rs.getLong("age"));
                                user.setAddress(rs.getString("address"));
                                return user;
                            }
                        }
                )
                .build();
    }

    @Bean
    public Step step01_52() {
        return this.stepBuilderFactory.get("step_01") //
                .<T52_User, T52_User>chunk(1) // 一次讀幾筆資料
                .reader(this.t52_itemReader()) // 設置Reader
                .writer(this.t52_itemWriter()) // 設置Writer
                .build();
    }

    @Bean
    public Job job52() {
        return this.jobBuilderFactory.get("t52_job") //
                .start(this.step01_52()) //
                .incrementer(new RunIdIncrementer()) //
                .build();
    }
}
```

### 使用分頁方式讀取
`JdbcPagingItemReader`，取得每一頁資料

```java
// 讀取數據庫，封裝至對象中
@EnableBatchProcessing
@SpringBootApplication
public class T53_JdbcItemReaderTest2 {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;
    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        SpringApplication.run(T53_JdbcItemReaderTest2.class, args);
    }

    @Bean
    public ItemWriter t53_itemWriter() {
        return new ItemWriter() {
            @Override
            public void write(List list) throws Exception {
                list.forEach(System.out::println);
            }
        };
    }

    @Bean
    public PagingQueryProvider t53_pagingQueryProvider() throws Exception {
        SqlPagingQueryProviderFactoryBean factoryBean = new SqlPagingQueryProviderFactoryBean();
        factoryBean.setDataSource(this.dataSource); // 設置數據源
        factoryBean.setSelectClause("SELECT *"); // 選擇列
        factoryBean.setFromClause("FROM t52_user"); // 選擇表
        factoryBean.setWhereClause("WHERE  age >= :age"); // 選擇條件，:age表示占位符
        factoryBean.setSortKey("id"); // 排序欄位名
        return factoryBean.getObject();
    }

    @Bean
    public JdbcPagingItemReader<T52_User> t53_itemReader() throws Exception {
        // sql條件
        Map<String, Object> sqlCondition = new HashMap<>();
        sqlCondition.put("age", 200);
        // 使用JDBC讀取分頁資料
        // SELECT * FROM t52_user limit ?,pageSize
        return new JdbcPagingItemReaderBuilder<T52_User>()
                .name("t53_itemReader") // 設置Reader名稱
                .dataSource(this.dataSource) // 設置數據源，與數據庫連線，Spring容器實現
                .queryProvider(this.t53_pagingQueryProvider()) // 分頁邏輯
                .parameterValues(sqlCondition) // sql條件
                .pageSize(2) // 一頁有多少數據
                .rowMapper( // 數據和對象一一映射
                        new RowMapper<T52_User>() {
                            @Override
                            public T52_User mapRow(ResultSet rs, int rowNum) throws SQLException {
                                T52_User user = new T52_User();
                                user.setId(rs.getLong("id"));
                                user.setName(rs.getString("name"));
                                user.setAge(rs.getLong("age"));
                                user.setAddress(rs.getString("address"));
                                return user;
                            }
                        }
                )
                .build();
    }

    @Bean
    public Step step01_53() throws Exception {
        return this.stepBuilderFactory.get("step_01") //
                .<T52_User, T52_User>chunk(1) // 一次讀幾筆資料
                .reader(this.t53_itemReader()) // 設置Reader
                .writer(this.t53_itemWriter()) // 設置Writer
                .build();
    }

    @Bean
    public Job job53() throws Exception {
        return this.jobBuilderFactory.get("t53_job") //
                .start(this.step01_53()) //
                .incrementer(new RunIdIncrementer()) //
                .build();
    }
}
```

## 🍃異常處理
### 跳過異常記錄
```java
@Bean
public Step step01_53() throws Exception {
	// 跳過異常記錄
	return this.stepBuilderFactory.get("step_01") //
			.<T52_User, T52_User>chunk(1) // 一次讀幾筆資料
			.reader(this.t53_itemReader()) // 設置Reader
			.writer(this.t53_itemWriter()) // 設置Writer
			.faultTolerant() // 容錯
			.skip(RuntimeException.class) // 跳過異常類型
			.skipLimit(10) // 跳過異常次數
			.skipPolicy(
					new SkipPolicy() {
						@Override
						public boolean shouldSkip(
								Throwable throwable, // 異常信息
								int i // 跳過次數
						) throws SkipLimitExceededException {
							// 定制跳過異常、異常次數
							return false;
						}
					}
			)
			.build();
}
```

### 異常記錄日誌
1. 建立監聽器，實現`ItemReadListener`
	```java
	public class T55_ItemReaderListener implements ItemReadListener {
	    @Override
	    public void beforeRead() {
	
	    }
	
	    @Override
	    public void afterRead(Object o) {
	
	    }
	
	    @Override
	    public void onReadError(Exception e) {
	        System.out.println("將異常信息記錄");
	    }
	}
	```
2. 設置Reader監聽器
	```java
	@Bean
	public Step step01_53() throws Exception {
		// 異常記錄日誌
		return this.stepBuilderFactory.get("step_01") //
				.<T52_User, T52_User>chunk(1) // 一次讀幾筆資料
				.reader(this.t53_itemReader()) // 設置Reader
				.writer(this.t53_itemWriter()) // 設置Writer
				.listener(new T55_ItemReaderListener()) // Reader監聽器
				.build();
	}
	```



