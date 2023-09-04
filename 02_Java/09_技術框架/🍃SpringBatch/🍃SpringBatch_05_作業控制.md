# Spring Batch作業啟動
## 🍃SpringBoot啟動
- [[🍃SpringBatch_01_HelloWorld]]中的寫法就是通過SpringBoot啟動批次作業
- SpringBoot啟動後，會調用`JobLauncherApplicationRunner.run()`，將操作委托給`SimpleJobLauncher.run()`執行。默認情況下，SpringBoot啟動就會馬上執行批次作業
- 可以在application.properties設置啟動不會馬上執行批次作業
```properties
# 停止SpringBoot啟動就會馬上執行批次作業
spring.batch.job.enabled: false
```

## 🍃Spring單元測試啟動
### Maven引入
```xml
<dependency>  
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

### application.properties
設置啟動不會馬上執行批次作業
```properties
# 停止SpringBoot啟動就會馬上執行批次作業
spring.batch.job.enabled: false
```

### 啟動類
```java
@SpringBootApplication
public class T40_AppStart {
    public static void main(String[] args) {
        SpringApplication.run(T40_AppStart.class, args);
    }
}
```

### 測試類
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class T40_HelloWorldTest {
    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Test
    public void test() throws Exception {
        this.jobLauncher.run( //
                this.job(), // 作業實例
                new JobParameters()); // 作業運行攜帶參數
    }

    public Tasklet tasklet() {
        return new Tasklet() {
            @Override
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                System.out.println("Do Something in Tasklet");
                return RepeatStatus.FINISHED;
            }
        };
    }

    public Step step01() {
        return this.stepBuilderFactory.get("step_01") //
                .tasklet(this.tasklet()) //
                .build();
    }

    public Job job() {
        return this.jobBuilderFactory.get("t40_job") //
                .start(this.step01()) //
                .build();
    }
}
```

## 🍃RESTful API啟動
### Maven引入
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### application.properties
設置啟動不會馬上執行批次作業
```properties
# 停止SpringBoot啟動就會馬上執行批次作業
spring.batch.job.enabled: false
```

### 啟動類
```java
@SpringBootApplication
public class T40_AppStart {
    public static void main(String[] args) {
        SpringApplication.run(T40_AppStart.class, args);
    }
}
```

### 配置類
```java
// 單獨配置作業相關操作
@EnableBatchProcessing
@Configuration
public class T41_BatchConfig {
    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

	@Bean
	public Step step01_41() {
		return this.stepBuilderFactory.get("step_01") //
				.tasklet(new Tasklet() {
					@Override
					public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
						chunkContext.getStepContext().getJobParameters() //
								.entrySet() //
								.forEach(entry -> System.out.println(entry.getKey() + " : " + entry.getValue()));
						return RepeatStatus.FINISHED; // 表示執行完畢
					}
				}) //
				.build();
	}

    @Bean
    public Job job_41() {
        return this.jobBuilderFactory.get("t41_job") //
                .start(this.step01_41()) //
                .incrementer(new RunIdIncrementer()) // 設置run.id自增增量器
                .build();
    }
}
```

### Controller
```java
@RestController()
@RequestMapping("/t41")
public class T41_HelloWorldStartJob {
    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job_41;

    // 不帶參數
    @GetMapping("/startJob")
    @ResponseBody
    public ExitStatus start() throws Exception {
        // 啟動Job作業
        JobExecution jobExecution = this.jobLauncher.run(this.job_41, new JobParameters());
        return jobExecution.getExitStatus();
    }

    // 帶參數
    @GetMapping("/startJobByParam")
    @ResponseBody
    public ExitStatus startByParam(
            String name
    ) throws Exception {
        // 作業參數
//        JobParameters jobParameters = new JobParametersBuilder() //
//                .addString("name", name) //
//                .toJobParameters() ;
        JobParameters jobParameters = new JobParametersBuilder(this.jobExplorer) //
                .getNextJobParameters(this.job_41) // 取得上一次run.id自增參數值
                .addString("name", name) //
                .toJobParameters() ;
        // 啟動Job作業
        JobExecution jobExecution = this.jobLauncher.run(this.job_41, jobParameters);
        return jobExecution.getExitStatus();
    }
}
```

### 測試
發起請求，起動作業
- http://127.0.0.1:8080/t41/startJob
	```console
	{"exitCode":"COMPLETED","exitDescription":"","running":false}
	```
- http://127.0.0.1:8080/t41/startJobByParam?name=mickey
	```console
	{"exitCode":"COMPLETED","exitDescription":"","running":false}
	```

# Spring Batch作業停止
作業停止有以下情況：
- 作業執行成功、正常停止，`COMPLETED`
- 因為各種意外導致作業停止，`FAILED`
- 某步驟數據結果不滿足下一步執行前提，手動讓其停止，`STOPED`

## 🍃使用步驟監聽器停止
### 建立全局變量
可使用上下文實現
```java
public class T43_ReasouseCount {
    public static int TOTAL_COUNT = 100;
    public static int READ_COUNT = 0;
}
```

### 建立監聽器
```java
// 自定義步驟執行監聽器
public class T43_StepStopListener implements StepExecutionListener {
    @Override
    public void beforeStep(StepExecution stepExecution) {
        System.out.println("T43_StepStopListener.beforeStep()");
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        System.out.println("T43_StepStopListener.afterStep()");
        // 不滿足條件處理
        if (T43_ReasouseCount.TOTAL_COUNT != T43_ReasouseCount.READ_COUNT){
            return ExitStatus.STOPPED;
        }

        return stepExecution.getExitStatus(); // 必要改動
    }
}
```

### 步驟設置監聽器
- `listener()`，設置監聽器
- `allowStartIfComplete(true)`，允許此步驟可重覆執行

```java
// 停止步驟監聽器
@SpringBootApplication
@EnableBatchProcessing
public class T43_StepStopTest {
    public static void main(String[] args) {
        SpringApplication.run(T43_StepStopTest.class, args);
    }

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    // 監聽器
    @Bean
    public T43_StepStopListener t43_StepStopListener(){
        return new T43_StepStopListener();
    }

    @Bean
    public Step step01_43() {
        return this.stepBuilderFactory.get("step_01") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("Do Something In Step 01...");
                        // TODO，模擬讀取資料
                        // 若不為100(不符合條件)，監聽器會將狀態改為STOPPED，並停止執行
                        // 若先前有被停止執行的作業，且此次數據符合條件(==100)，會接續執行上次的作業
                        T43_ReasouseCount.READ_COUNT += 100;
                        return RepeatStatus.FINISHED;
                    }
                }) //
                .listener(this.t43_StepStopListener()) // 設置監聽器
                .allowStartIfComplete(true) // 允許此步驟可重覆執行
                .build();
    }

    @Bean
    public Step step02_43() {
        return this.stepBuilderFactory.get("step_02") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("Do Something In Step 02...");
                        System.out.println( //
                                String.format("TOTAL_COUNT:%s, READ_COUNT:%s", //
                                        T43_ReasouseCount.TOTAL_COUNT, //
                                        T43_ReasouseCount.READ_COUNT
                                ));
                        return RepeatStatus.FINISHED;
                    }
                }) //
                .build();
    }

    @Bean
    public Job job_43() throws Exception {
        return this.jobBuilderFactory.get("t43_job") //
                .start(this.step01_43()) //
                .on("STOPPED").stopAndRestart(this.step01_43()) // 狀態碼為STOPPED時，重覆執行step_01
                .from(this.step01_43()).on("*").to(this.step02_43()) // 其他情況，正常執行
                .end() //
                .incrementer(new RunIdIncrementer()) //
                .build();
    }
}
```

## 🍃步驟設置停標記
```java
// 步驟設置停標記
@SpringBootApplication
@EnableBatchProcessing
public class T44_StepStopTest2 {
    public static void main(String[] args) {
        SpringApplication.run(T44_StepStopTest2.class, args);
    }

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step step01_44() {
        return this.stepBuilderFactory.get("step_01") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("Do Something In Step 01...");
                        // TODO，模擬讀取資料
                        // 若不為100(不符合條件)，監聽器會將狀態改為STOPPED，並停止執行
                        // 若先前有被停止執行的作業，且此次數據符合條件(==100)，會接續執行上次的作業
                        T43_ReasouseCount.READ_COUNT += 100;
                        // 不滿足條件處理，設置停止標記
                        if (T43_ReasouseCount.TOTAL_COUNT != T43_ReasouseCount.READ_COUNT){
                            chunkContext.getStepContext().getStepExecution().setTerminateOnly();
                        }
                        return RepeatStatus.FINISHED;
                    }
                }) //
                .allowStartIfComplete(true) // 允許此步驟可重覆執行
                .build();
    }

    @Bean
    public Step step02_44() {
        return this.stepBuilderFactory.get("step_02") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("Do Something In Step 02...");
                        System.out.println( //
                                String.format("TOTAL_COUNT:%s, READ_COUNT:%s", //
                                        T43_ReasouseCount.TOTAL_COUNT, //
                                        T43_ReasouseCount.READ_COUNT
                                ));
                        return RepeatStatus.FINISHED;
                    }
                }) //
                .build();
    }

    @Bean
    public Job job_44() throws Exception {
        return this.jobBuilderFactory.get("t44_job") //
                .start(this.step01_44()) //
                .next(this.step02_44()) //
                .incrementer(new RunIdIncrementer()) //
                .build();
    }
}
```

# Spring Batch作業重啟
默認情況下，只允許異常或終止狀態的步驟重啟

## 🍃禁止重啟
`preventRestart()`，如果執行失敗，不允許再次執行

1. 步驟設置停止標記，模擬數據不符合狀況
	```java
	@Bean
	public Step step01_45() {
		return this.stepBuilderFactory.get("step_01") //
				.tasklet(new Tasklet() {
					@Override
					public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
						System.out.println("Do Something In Step 01...");
						// 設置停止標記
						chunkContext.getStepContext().getStepExecution().setTerminateOnly();
						return RepeatStatus.FINISHED;
					}
				}) //
				.build();
	}
	```
2. 設置禁止重啟
	```java
	@Bean  
	public Job job_45() throws Exception {  
		return this.jobBuilderFactory.get("t45_job") //  
				.start(this.step01_45()) //  
				.next(this.step02_45()) //  
				.preventRestart() // 禁止重啟  
	//          .incrementer(new RunIdIncrementer()) // 使用自增器無法測出  
				.build();  
	}
	```
3. 第一次執行狀態為`STOPPED`時，再關閉停止標記(摸擬數據符合狀況)執行作業，會顯示錯誤
	```console
	Caused by: org.springframework.batch.core.repository.JobRestartException: JobInstance already exists and is not restartable
		at org.springframework.batch.core.launch.support.SimpleJobLauncher.run(SimpleJobLauncher.java:107) ~[spring-batch-core-4.3.4.jar:4.3.4]
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
	```

## 🍃限制重啟次數
`startLimit(次數)`，限制啟動次數

1. 步驟設置停止標記，模擬數據不符合狀況
	設置限制啟動次數
	```java
	@Bean
	public Step step01_46() {
		return this.stepBuilderFactory.get("step_01") //
				.tasklet(new Tasklet() {
					@Override
					public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
						System.out.println("Do Something In Step 01...");
						// 設置停止標記
						chunkContext.getStepContext().getStepExecution().setTerminateOnly();
						return RepeatStatus.FINISHED;
					}
				}) //
				.startLimit(2) // 限制重啟1次，第一次啟動就開始計數
				.build();
	}
	```
2. 當啟動次數超過設定的次數，會報錯
	```console
	org.springframework.batch.core.StartLimitExceededException: Maximum start limit exceeded for step: step_01StartMax: 2
		at org.springframework.batch.core.job.SimpleStepHandler.shouldStart(SimpleStepHandler.java:235) ~[spring-batch-core-4.3.4.jar:4.3.4]
		at org.springframework.batch.core.job.SimpleStepHandler.handleStep(SimpleStepHandler.java:128) ~[spring-batch-core-4.3.4.jar:4.3.4]
		at org.springframework.batch.core.job.AbstractJob.handleStep(AbstractJob.java:413) ~[spring-batch-core-4.3.4.jar:4.3.4]
	```

## 🍃無限重啟
相同的==作業名稱==、==作業標識參數==表示是同一次作業執行，再執行作業會直接返回狀態`NOOP`(執行無效)
`allowStartIfComplete(true)`，設定步驟允許無限重啟

```java
@Bean
public Step step01_47() {
	return this.stepBuilderFactory.get("step_01") //
			.tasklet(new Tasklet() {
				@Override
				public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
					System.out.println("Do Something In Step 01...");
					return RepeatStatus.FINISHED;
				}
			}) //
			.allowStartIfComplete(true) // 允許無限重啟
			.build();
}
```