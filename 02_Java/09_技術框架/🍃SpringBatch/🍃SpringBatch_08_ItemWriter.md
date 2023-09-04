# Spring Batch ItemWriter
## 🍃輸出純文本文件
`FlatFileItemWriter`，實現純文本文件輸出

```java
// 輸出純文本文件
@EnableBatchProcessing
@SpringBootApplication
public class T62_ItemWriterTest {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    public static void main(String[] args) {
        SpringApplication.run(T62_ItemWriterTest.class, args);
    }

    @Bean
    public FlatFileItemReader t62_itemReader() {
        return new FlatFileItemReaderBuilder<>()
                .name("t62_itemReader")
                .resource(new ClassPathResource("T62_ReadFile.txt"))
                .delimited().delimiter("#")
                .names("id", "name", "age")
                .targetType(T62_User.class)
                .build();
    }

    @Bean
    public FlatFileItemWriter t62_latFileItemWriter(){
        return new FlatFileItemWriterBuilder<T62_User>()
                .name("t62_itemWriter") // 設置Writer名稱
                .resource(new PathResource("D:\\T62_WriteFile.txt")) // 輸出文件路徑
                .formatted() // 輸出格式
                .format("id=%s, name=%s, age=%s") // 數據輸出格式
                .names("id", "name", "age") // 占位符對應欄位名稱
//                .shouldDeleteIfEmpty(true) // 若輸入數據為空，輸出時創建的文件將直接刪除
//                .shouldDeleteIfExists(true) // 若輸出文件已存在，則刪除
//                .append(true) // 若輸出文件已存在，將資料追加到現在文件中
                .build();
    }

    @Bean
    public Step step01_62() {
        return this.stepBuilderFactory.get("step_01")
                .<T62_User, T62_User>chunk(1)
                .reader(this.t62_itemReader())
                .writer(this.t62_latFileItemWriter()) // 設置Writer
                .build();
    }

    @Bean
    public Job job62() {
        return this.jobBuilderFactory.get("t62_job") //
                .incrementer(new RunIdIncrementer()) //
                .start(this.step01_62()) //
                .build();
    }
}
```

## 🍃輸出JSON文件
- `JsonFileItemWriter`，實現JSON文件輸出
- Spring Batch默認提供Json對象調度器
	- `JacksonJsonObjectMarshaller`
	- `GsonJsonObjectMarshaller`

```java
// 輸出JSON文件
@Bean
public JsonFileItemWriter t63_jsonFileItemWriter(){
	return new JsonFileItemWriterBuilder<T62_User>()
			.name("t63_itemWriter") // 設置Writer名稱
			.resource(new PathResource("D:\\T63_WriteFile.json")) // 輸出文件路徑
			.jsonObjectMarshaller(new JacksonJsonObjectMarshaller<>()) // json對象調度器
			.build();
}

@Bean
public Step step01_62() {
	return this.stepBuilderFactory.get("step_01")
			.<T62_User, T62_User>chunk(1)
			.reader(this.t62_itemReader())
//                .writer(this.t62_latFileItemWriter()) // 設置Writer(純文本文件)
			.writer(this.t63_jsonFileItemWriter()) // 設置Writer(JSON文件)
			.build();
}
```

## 🍃輸出數據庫
`JdbcBatchItemWriter`，實現數據庫輸出

```java
@Autowired
private DataSource dataSource;

// 輸出數據庫
@Bean
public JdbcBatchItemWriter<T62_User> t64_jdbcBatchItemWriter(){
	return new JdbcBatchItemWriterBuilder<T62_User>()
			.dataSource(this.dataSource) // 設置數據源
			.sql("INSERT INTO t52_user(name, age) VALUES(?, ?)") // 插入數據sql
			.itemPreparedStatementSetter(new ItemPreparedStatementSetter<T62_User>() { // 設置占位符參數
				@Override
				public void setValues(T62_User user, PreparedStatement ps) throws SQLException {
					ps.setString(1, user.getName());
					ps.setLong(2, user.getAge());
				}
			})
			.build();
}

@Bean
public Step step01_62() {
	return this.stepBuilderFactory.get("step_01")
			.<T62_User, T62_User>chunk(1)
			.reader(this.t62_itemReader())
//                .writer(this.t62_latFileItemWriter()) // 設置Writer(純文本文件)
//                .writer(this.t63_jsonFileItemWriter()) // 設置Writer(JSON文件)
			.writer(this.t64_jdbcBatchItemWriter()) // 設置Writer(數據庫)
			.build();
}
```

## 🍃輸出多終端
`CompositeItemWriter`，實現多終端輸出

```java
@EnableBatchProcessing
@SpringBootApplication
public class T62_ItemWriterTest {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;
    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        SpringApplication.run(T62_ItemWriterTest.class, args);
    }

    @Bean
    public FlatFileItemReader t62_itemReader() {
        return new FlatFileItemReaderBuilder<>()
                .name("t62_itemReader")
                .resource(new ClassPathResource("T62_ReadFile.txt"))
                .delimited().delimiter("#")
                .names("id", "name", "age")
                .targetType(T62_User.class)
                .build();
    }

    // 輸出純文本文件
    @Bean
    public FlatFileItemWriter t62_latFileItemWriter(){
        return new FlatFileItemWriterBuilder<T62_User>()
                .name("t62_itemWriter") // 設置Writer名稱
                .resource(new PathResource("D:\\T62_WriteFile.txt")) // 輸出文件路徑
                .formatted() // 輸出格式
                .format("id=%s, name=%s, age=%s") // 數據輸出格式
                .names("id", "name", "age") // 占位符對應欄位名稱
//                .shouldDeleteIfEmpty(true) // 若輸入數據為空，輸出時創建的文件將直接刪除
//                .shouldDeleteIfExists(true) // 若輸出文件已存在，則刪除
//                .append(true) // 若輸出文件已存在，將資料追加到現在文件中
                .build();
    }

    // 輸出JSON文件
    @Bean
    public JsonFileItemWriter t63_jsonFileItemWriter(){
        return new JsonFileItemWriterBuilder<T62_User>()
                .name("t63_itemWriter") // 設置Writer名稱
                .resource(new PathResource("D:\\T63_WriteFile.json")) // 輸出文件路徑
                .jsonObjectMarshaller(new JacksonJsonObjectMarshaller<>()) // json對象調度器
                .build();
    }

    // 輸出數據庫
    @Bean
    public JdbcBatchItemWriter<T62_User> t64_jdbcBatchItemWriter(){
        return new JdbcBatchItemWriterBuilder<T62_User>()
                .dataSource(this.dataSource) // 設置數據源
                .sql("INSERT INTO t52_user(name, age) VALUES(?, ?)") // 插入數據sql
                .itemPreparedStatementSetter(new ItemPreparedStatementSetter<T62_User>() { // 設置占位符參數
                    @Override
                    public void setValues(T62_User user, PreparedStatement ps) throws SQLException {
                        ps.setString(1, user.getName());
                        ps.setLong(2, user.getAge());
                    }
                })
                .build();
    }

    // 輸出多終端
    @Bean
    public CompositeItemWriter<T62_User> t65_ompositeItemWriter(){
        return new CompositeItemWriterBuilder<T62_User>()
                .delegates(Arrays.asList( // 設置多組Writer
                        this.t62_latFileItemWriter(),
                        this.t63_jsonFileItemWriter(),
                        this.t64_jdbcBatchItemWriter()
                ))
                .build();
    }

    @Bean
    public Step step01_62() {
        return this.stepBuilderFactory.get("step_01")
                .<T62_User, T62_User>chunk(1)
                .reader(this.t62_itemReader())
//                .writer(this.t62_latFileItemWriter()) // 設置Writer(純文本文件)
//                .writer(this.t63_jsonFileItemWriter()) // 設置Writer(JSON文件)
//                .writer(this.t64_jdbcBatchItemWriter()) // 設置Writer(數據庫)
                .writer(this.t65_ompositeItemWriter()) // 設置Writer(多終端)
                .build();
    }

    @Bean
    public Job job62() {
        return this.jobBuilderFactory.get("t62_job") //
                .incrementer(new RunIdIncrementer()) //
                .start(this.step01_62()) //
                .build();
    }
}
```

