# Spring Batch批處理數據表
![[SpringBatch_04_批處理數據表_01_表關系](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%8D%83SpringBatch/images/SpringBatch_04_%E6%89%B9%E8%99%95%E7%90%86%E6%95%B8%E6%93%9A%E8%A1%A8_01_%E8%A1%A8%E9%97%9C%E7%B3%BB.png?raw=true)

- ==BATCH_JOB_INSTANCE==，執行一次作業時就會産生一筆資料
	- `JOB_EXECUTION_ID`，作業執行實例主鍵
	- `VERSION`，樂觀鎖控制的版本號
	- `JOB_NAME`，作業名稱
	- `JOB_KEY`，作業名與作業標識參數的哈希值，能標識唯一的Job實例
- ==BATCH_JOB_EXECUTION==，記錄每次作業執行的狀態，如：啟動時間、完成時間…等
	- `JOB_EXECUTION_ID`，作業執行實例主鍵
	- `VERSION`，樂觀鎖控制的版本號
	- `JOB_INSTANCE_ID`，對應BATCH_JOB_INSTANCE
	- `CREATE_TIME`，記錄創建時間
	- `START_TIME`，作業執行開始時間
	- `END_TIME`，作業執行結束時間
	- `STATUS`，作業執行的批處理狀態，對應[[🍃SpringBatch_03_Step#🍃ExitStatus作業狀態]]
	- `EXIT_CODE`，作業執行的退出碼，對應[[🍃SpringBatch_03_Step#🍃ExitStatus作業狀態]]
	- `EXIT_MESSAGE`，作業執行的退出信息
	- `LAST_UPDATED`，最後一次更新記錄時間
	- `JOB_CONFIGURATION_LOCATION`
- ==BATCH_JOB_EXECUTION_CONTEXT==，記錄作業上下文對象
	- `JOB_EXECUTION_ID`，作業執行實例主鍵
	- `SHORT_CONTEXT`，ExecutionContext序列化後字符串縮減版
	- `SERIALIZED_CONTEXT`，ExecutionContext序列化後字符串
- ==BATCH_JOB_EXECUTION_PARAMS==，記錄作業執行參數
	- `JOB_EXECUTION_ID`，作業執行實例主鍵
	- `TYPE_CD`，標記參數類型
	- `KEY_NAME`，參數名
	- `STRING_VAL`，當參數類型為String時的值
	- `DATE_VAL`，當參數類型為Date時的值
	- `LONG_VAL`，當參數類型為Long時的值
	- `DOUBLE_VAL`，當參數類型為Double時的值
	- `IDENTIFYING`，用於標記該參數是否為標識性參數
- BATCH_JOB_EXECUTION_SEQ
- BATCH_JOB_SEQ
- ==BATCH_STEP_EXECUTION==，記錄步驟執行狀態
	- `STEP_EXECUTION_ID`，步驟執行對象id
	- `VERSION`，樂觀鎖控制的版本號
	- `STEP_NAME`，步驟名稱
	- `JOB_EXECUTION_ID`，作業執行實例主鍵
	- `START_TIME`，步驟執行的開始時間
	- `END_TIME`，步驟執行的結束時間
	- `STATUS`，步驟批存理狀態，對應[[🍃SpringBatch_03_Step#🍃ExitStatus作業狀態]]
	- `COMMIT_COUNT`，步驟提交事務次數
	- `READ_COUNT`，讀入條目數量
	- `FILTER_COUNT`，由於`ItemProcessor`返回null而過濾的條目數
	- `WRITE_COUNT`，寫入條目數量
	- `READ_SKIP_COUNT`，由於`ItemReader`中拋出異常而跳過的條目數量
	- `WRITE_SKIP_COUNT`，由於`ItemWriter`中拋出異常而跳過的條目數量
	- `PROCESS_SKIP_COUNT`，由於`ItemProcessor`中拋出異常而跳過的條目數量
	- `ROLLBACK_COUNT`，在步驟執行中被回滾的事務數量
	- `EXIT_CODE`，步驟的退出碼
	- `EXIT_MESSAGE`，步驟執行返回信息
	- `LAST_UPDATED`，最後一次更新記錄時間
- BATCH_STEP_EXECUTION_CONTEXT
	- `STEP_EXECUTION_ID`，步驟執行對象id
	- `SHORT_CONTEXT`，ExecutionContext序列化後字符串縮減版
	- `SERIALIZED_CONTEXT`，ExecutionContext序列化後字符串
- BATCH_STEP_EXECUTION_SEQ
