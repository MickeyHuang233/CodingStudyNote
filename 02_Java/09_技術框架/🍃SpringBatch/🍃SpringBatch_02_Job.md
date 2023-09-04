# Spring Batch Job
- Job表示能有序並獨立執行的步驟(Step)列表
- 相關類說明
	- `Job`，用於定義每次作業執行的規范、步驟
	- `JobInstance`(作業實例)，表示一次Job的運行，每當作業運行時都會創建，並通過==作業名稱==、==作業標識參數==來區分不同的作作實例
	- `JobExecution`(作業執行器)，用於記錄Job執行情況，如：開始執行時間、執行完成時間、處理狀態…等。同一個作業實例可能會執行多次，因此可能也會對應多筆作業執行器
	- `JobParameters`，主要維護`Map<String, JobParameter>`作業參數
	- `JobParameter`
		- `parameter`，參數值
		- `parameterType`，參數類型
		- `identifying`，是否做區分

## 🍃作業標識參數
### 設置作業參數
1. Edit Configurations...
	![[SpringBatch_02_Job_01_設置作業參數.png]]
2. `+` --> Application --> 新增Job參數
	![[SpringBatch_02_Job_02_設置作業參數.png]]
3. 執行剛剛新增的Application
	![[SpringBatch_02_Job_03_設置作業參數.png]]
4. 執行成功後，可在資料表`BATCH_JOB_EXECUTION_PARAMS`中看到剛剛設定的參數
	```console
	JOB_EXECUTION_ID|TYPE_CD|KEY_NAME|STRING_VAL|DATE_VAL           |LONG_VAL|DOUBLE_VAL|IDENTIFYING|
	----------------+-------+--------+----------+-------------------+--------+----------+-----------+
	               4|STRING |name    |mickey    |1970-01-01 08:00:00|       0|       0.0|Y          |
	```
5. 注意：Spring Batch中，相同Job名稱、相同標識參數只能成功執行一次，因此若參數不變，會報錯
	```console
	2023-08-24 11:17:02.481 ERROR 7548 --- [           main] o.s.boot.SpringApplication               : Application run failed
	
	java.lang.IllegalStateException: Failed to execute ApplicationRunner
		at org.springframework.boot.SpringApplication.callRunner(SpringApplication.java:796) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.callRunners(SpringApplication.java:783) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:345) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:1354) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:1343) ~[spring-boot-2.5.7.jar:2.5.7]
		at mickey.study.T06_JobParamTest.main(T06_JobParamTest.java:24) ~[classes/:na]
	Caused by: org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException: A job instance already exists and is complete for parameters={name=mickey}.  If you want to run this job again, change the parameters.
		at org.springframework.batch.core.repository.support.SimpleJobRepository.createJobExecution(SimpleJobRepository.java:139) ~[spring-batch-core-4.3.4.jar:4.3.4]
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
		at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
		at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
		at java.base/java.lang.reflect.Method.invoke(Method.java:567) ~[na:na]
		at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344) ~[spring-aop-5.3.13.jar:5.3.13]
		at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198) ~[spring-aop-5.3.13.jar:5.3.13]
	```

### 取得作業參數
- 通過`ChunkContext`取得作業參數
	```java
	@Bean  
	public Tasklet tasklet_12() {  
	    return new Tasklet() {  
	        @Override  
	        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {  
	            Map<String, Object> jobParameters = chunkContext //  
	                    .getStepContext() // 取得步驟上下文  
	                    .getJobParameters();// 取得作業參數  
	            jobParameters.entrySet().forEach(entry -> System.out.println(entry.getKey() + " : " + entry.getValue()));  
	            return RepeatStatus.FINISHED;
	        }  
	    };  
	}
	```
- 使用`@StepScope`延時獲取
	```java
	@Bean  
	@StepScope // 表示啟動專案時，不加載該Step步驟Bean，等到被調用時才加載，也就是延時獲取
	public Tasklet tasklet_12(  
	        @Value("#{jobParameters['name']}") String name  
	) {  
	    return new Tasklet() {  
	        @Override  
	        public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {  
	            System.out.println("name : " + name); // 取得參數  
	            System.out.println("Do Something in Tasklet");  
	            return RepeatStatus.FINISHED;  
	        }  
	    };  
	}  
	  
	@Bean  
	public Step step01_12() {  
	    return this.stepBuilderFactory.get("step_01") //  
	            .tasklet(this.tasklet_12(null)) // 參數通過注入，所以不需要帶值  
	            .build();  
	}
	```

## 🍃參數校驗器
### 自定義參數校驗器
1. 新建參數校驗器，實現`JobParametersValidator`接口
	```java
	public class T15_ParamValidator implements JobParametersValidator {  
	    @Override  
	    public void validate(JobParameters jobParameters) throws JobParametersInvalidException {  
	        String name = jobParameters.getString("name");  
	        // 校驗參數不能為空或空字符串  
	        if(!StringUtils.hasText(name)){  
	            throw new JobParametersInvalidException("name can't null");  
	        }  
	        // 若正常執行完畢，表示校驗都通過  
	    }  
	}
	```
2. 在建立Job前，設置參數校驗器
	```java
	@Bean  
	public Job job_15() {  
	    return this.jobBuilderFactory.get("t15_job") //  
	            .start(this.step01_12()) //  
	            .validator(this.t15_paramValidator()) // 設置自定義參數校驗器  
	            .build();  
	}
	```
3. 當執行不帶參數時，就會報錯
	```console
	java.lang.IllegalStateException: Failed to execute ApplicationRunner
		at org.springframework.boot.SpringApplication.callRunner(SpringApplication.java:796) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.callRunners(SpringApplication.java:783) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:345) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:1354) ~[spring-boot-2.5.7.jar:2.5.7]
		at org.springframework.boot.SpringApplication.run(SpringApplication.java:1343) ~[spring-boot-2.5.7.jar:2.5.7]
		at mickey.study.T06_JobParamTest.main(T06_JobParamTest.java:29) ~[classes/:na]
	Caused by: org.springframework.batch.core.JobParametersInvalidException: name can't null
		at mickey.study.job.T15_ParamValidator.validate(T15_ParamValidator.java:14) ~[classes/:na]
	```

### 默認參數校驗器
- `DefaultJobParametersValidator`
	- `requiredKeys`，必填參數keys
	- `optionalKeys`，選填參數keys

```java
// 默認參數校驗器
@Bean
public DefaultJobParametersValidator t16_DefaultJobParametersValidator(){
	DefaultJobParametersValidator defaultJobParametersValidator = new DefaultJobParametersValidator();
	// 必填參數keys
	defaultJobParametersValidator.setRequiredKeys(new String[]{"name"});
	// 選填參數keys
	defaultJobParametersValidator.setOptionalKeys(new String[]{"age"});
	return defaultJobParametersValidator;
}
	
@Bean
public Job job_16() {
	return this.jobBuilderFactory.get("t16_job") //
			.start(this.step01_12()) //
//          .validator(this.t15_paramValidator()) // 設置自定義參數校驗器
			.validator(this.t16_DefaultJobParametersValidator()) // 設置默認參數校驗器
			.build();
}
```

### 組合參數校驗器
`CompositeJobParametersValidator`，將多個參數校驗器依序執行

```java
// 自定義參數校驗器
public T15_ParamValidator t15_paramValidator(){
	return new T15_ParamValidator();
}

// 默認參數校驗器
@Bean
public DefaultJobParametersValidator t16_DefaultJobParametersValidator(){
	DefaultJobParametersValidator defaultJobParametersValidator = new DefaultJobParametersValidator();
	defaultJobParametersValidator.setRequiredKeys(new String[]{"name"});
	defaultJobParametersValidator.setOptionalKeys(new String[]{"age"});
	return defaultJobParametersValidator;
}

// 組合參數校驗器
public CompositeJobParametersValidator t17_compositeJobParametersValidator() throws Exception {
	CompositeJobParametersValidator compositeJobParametersValidator = new CompositeJobParametersValidator();
	// 將多個參數校驗器放至列表
	compositeJobParametersValidator.setValidators(Arrays.asList(
			this.t15_paramValidator(), // 自定義參數校驗器
			this.t16_DefaultJobParametersValidator() // 默認參數校驗器
	));
	// 保證參數校驗器中至少有一個不為空
	compositeJobParametersValidator.afterPropertiesSet();
	return compositeJobParametersValidator;
}

@Bean
public Job job_16() throws Exception {
	return this.jobBuilderFactory.get("t17_job") //
			.start(this.step01_12()) //
//          .validator(this.t15_paramValidator()) // 設置自定義參數校驗器
//          .validator(this.t16_DefaultJobParametersValidator()) // 設置默認參數校驗器
			.validator(this.t17_compositeJobParametersValidator()) // 設置組合參數校驗器
			.build();
}
```

## 🍃作業參數增量器
`JobParametersIncrementer`，使標識參數每次執行都會變動

- `RunIdIncrementer`，維護`run.id`的標識參數，每次啟動時自增1
	```java
	@Bean
	public Job job_18() throws Exception {
		return this.jobBuilderFactory.get("t18_job") //
				.start(this.step01_18()) //
				.incrementer(new RunIdIncrementer()) // 設置run.id自增增量器
				.build();
	}
	```
- 自定義作業參數增量器
	1. 建立自定義作業參數增量器，實現`JobParametersIncrementer`接口
		```java
		public class T19_DailyTimerstampParamIncrementer implements JobParametersIncrementer {  
		    @Override  
		    public JobParameters getNext(JobParameters jobParameters) {  
		        return new JobParametersBuilder(jobParameters) //  
		                .addLong("daily.id", new Date().getTime()) // 添加時間戳參數  
		                .toJobParameters();  
		    }  
		}
		```
	2. 設置自定義參數增量器
		```java
		@Bean
		public Job job_18() throws Exception {
			return this.jobBuilderFactory.get("t18_job") //
					.start(this.step01_18()) //
		//          .incrementer(new RunIdIncrementer()) // 設置run.id自增增量器
					.incrementer(this.t19_DailyTimerstampParamIncrementer()) // 設置自定義參數增量器
					.build();
		}
		```

## 🍃作業監聽器
`JobExecutionListener`，作業監聽器，用於監聽作業執行過程的邏輯
- `beforeJob`，作業執行前，一般用於初始化操作，如：建立連線、線程池初始化…等
- `afterJob`，作業執行後，業務執行完成釋放資源…等操作

### 接口實現方式
1. 建立作業監聽器，實現`JobExecutionListener`接口
	```java
	// 自定義作業監聽器
	public class T21_JobStateListener implements JobExecutionListener {
	    // 作業執行前
	    @Override
	    public void beforeJob(JobExecution jobExecution) {
	        System.out.println("T21_JobStateListener.beforeJob()，作業執行狀態：" + jobExecution.getStatus()); // STARTED
	    }
	
	    // 作業執行後
	    @Override
	    public void afterJob(JobExecution jobExecution) {
	        System.out.println("T21_JobStateListener.afterJob()，作業執行狀態：" + jobExecution.getStatus()); // COMPLETED
	    }
	}
	```
2. 設置自定義監聽器
	```java
	@SpringBootApplication
	@EnableBatchProcessing
	public class T21_JobListenerTest {
	    public static void main(String[] args) {
	        SpringApplication.run(T21_JobListenerTest.class, args);
	    }
	
	    @Autowired
	    private JobLauncher jobLauncher;
	
	    @Autowired
	    private JobBuilderFactory jobBuilderFactory;
	
	    @Autowired
	    private StepBuilderFactory stepBuilderFactory;
	
	    // 自定義監聽器
	    @Bean
	    public T21_JobStateListener t21_JobStateListener(){
	        return new T21_JobStateListener();
	    }
	
	    @Bean
	    public Tasklet tasklet_21() {
	        return new Tasklet() {
	            @Override
	            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
	                // 步驟執行過程
	                JobExecution jobExecution = stepContribution.getStepExecution().getJobExecution();
	                System.out.println("作業執行狀態：" + jobExecution.getStatus()); // STARTED
	
	                Map<String, Object> jobParameters = chunkContext //
	                        .getStepContext() // 取得步驟上下文
	                        .getJobParameters();// 取得作業參數
	                jobParameters.entrySet().forEach(entry -> System.out.println(entry.getKey() + " : " + entry.getValue()));
	                System.out.println("Do Something in Tasklet");
	                return RepeatStatus.FINISHED;
	            }
	        };
	    }
	
	    @Bean
	    public Step step01_21() {
	        return this.stepBuilderFactory.get("step_01") //
	                .tasklet(this.tasklet_21())
	                .build();
	    }
	
	    @Bean
	    public Job job_21() throws Exception {
	        return this.jobBuilderFactory.get("t21_job") //
	                .start(this.step01_21()) //
	                .listener(this.t21_JobStateListener()) // 設置自定義監聽器
	                .build();
	    }
	}
	```

### 注解方式
1. 建立作業監聽器，使用`@BeforeJob`、`@AfterJob`標識方法
	```java
	// 注解實現
	public class T22_JobListenerTest {
	    // 作業執行前
	    @BeforeJob
	    public void beforeJob(JobExecution jobExecution) {
	        System.out.println("T21_JobStateListener.beforeJob()，作業執行狀態：" + jobExecution.getStatus());
	    }
	
	    // 作業執行後
	    @AfterJob
	    public void afterJob(JobExecution jobExecution) {
	        System.out.println("T21_JobStateListener.afterJob()，作業執行狀態：" + jobExecution.getStatus());
	    }
	}
	```
2. 設置自定義監聽器
	```java
	@Bean
	public Job job_21() throws Exception {
		return this.jobBuilderFactory.get("t22_job") //
				.start(this.step01_21()) //
	//           .listener(new T22_JobListenerTest()) // 設置自定義監聽器
				.listener(JobListenerFactoryBean.getListener(new T22_JobListenerTest()))
				.build();
	}
	```

## 🍃執行上下文
- Spring時常用`XxxContext`表示上下文，如：`SpringApplicationContext`
	- Job Context，作業級的上下文環境，維護`JobExecution`對象，實現作業收尾工作、處理各種作業回調邏輯
	- Step Context，步驟級的上下文環境，維護`StepExecution`對象，實現步驟收尾工作、處理各種作業回調邏輯
- Execution Context，執行上下文，主要作用為數據共享
	![[SpringBatch_02_Job_04_執行上下文.png]]
	- Job Execution Context，用於所有Step間的數據共享
	- Step Execution Context，用於同一步驟間`ItemReader`、`ItemProcessor`、`ItemWriter`組件間的數據共享
- 引用鏈
	```mermaid
	graph LR 
	Job-->JobInstance;
	JobInstance-->JobExecution;
	JobExecution-->JobExecutionContext;
	Job-->Step;
	Step-->StepContext;
	StepContext-->StepExecution;
	StepExecution-->StepExecutionContext;
	```

### 程式碼
```java
@SpringBootApplication
@EnableBatchProcessing
public class T27_JobContextTest {
    public static void main(String[] args) {
        SpringApplication.run(T27_JobContextTest.class, args);
    }

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step step01_27() {
        return this.stepBuilderFactory.get("step_01") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        // 步驟上下文
                        StepContext stepContext = chunkContext.getStepContext();

                        // 步驟執行器
                        StepExecution stepExecution = stepContext.getStepExecution();
                        // 步驟執行上下文，只讀
                        Map<String, Object> stepExecutionContext = stepContext.getStepExecutionContext();
                        // 步驟執行上下文，可讀可寫
                        ExecutionContext stepEC = stepContext.getStepExecution().getExecutionContext();
                        stepEC.put("step01_key", "step01_value");
                        
                        // 作業執行上下文，只讀
                        Map<String, Object> jobExecutionContext = stepContext.getJobExecutionContext();
                        // 作業執行上下文，可讀可寫
                        ExecutionContext jobEC = stepContext.getStepExecution().getJobExecution().getExecutionContext();
                        jobEC.put("job_key", "job_value");

                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Step step02_27() {
        return this.stepBuilderFactory.get("step_02") //
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        // 步驟上下文
                        StepContext stepContext = chunkContext.getStepContext();

                        // 步驟執行上下文，可讀可寫
                        ExecutionContext stepEC = stepContext.getStepExecution().getExecutionContext();
                        System.out.println(stepEC.get("step01_key")); // 其他步驟的上下文無法取得

                        // 作業執行上下文，可讀可寫
                        ExecutionContext jobEC = stepContext.getStepExecution().getJobExecution().getExecutionContext();
                        System.out.println(jobEC.get("job_key")); // 作業上下文，可在不同步驟取得

                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Job job_27() throws Exception {
        return this.jobBuilderFactory.get("t27_job") //
                .start(this.step01_27()) //
                .next(this.step02_27()) //
                .incrementer(new RunIdIncrementer()) // 自增器
                .build();
    }
}
```

### 資料表
- `BATCH_JOB_EXECUTION_CONTEXT`
	```console
	{"@class":"java.util.HashMap","job_key":"job_value"}
	```
- `BATCH_STEP_EXECUTION_CONTEXT`
	```console
	步驟一：{"@class":"java.util.HashMap","batch.taskletType":"mickey.study.T27_JobContextTest$1","step01_key":"step01_value","batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep"}
	步驟二：{"@class":"java.util.HashMap","batch.taskletType":"mickey.study.T27_JobContextTest$2","batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep"}
	```





