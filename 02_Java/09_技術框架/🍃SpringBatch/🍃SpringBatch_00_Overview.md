# Spring Batch
- 批次處理，將數據分批次進行處理的過程，如：銀行對賬邏輯、跟系統數據同步…等
	![SpringBatch_00_Overview_01_常規批次步驟](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%8D%83SpringBatch/images/SpringBatch_00_Overview_01_%E5%B8%B8%E8%A6%8F%E6%89%B9%E6%AC%A1%E6%AD%A5%E9%A9%9F.png?raw=true)
	- 自動執行，正常情況無需人為介入
	- 數據量大
	- 定時執行
- Spring Batch
	- Spring的子項目，基於Spring框架為基礎的輕量級的批處理框架
	- Spring Batch提供大量可重用組件，如：日誌、追蹤、事務、任務作業統計、任務重啟、跳過、重複、資源管理…等
	- Spring Batch是批次處理應用框架，不提供調度框架，如果需要定時處理批次需要額外引入，如：Quartz
- Spring Batch架構，主要實現高內聚、低耦合、擴展性強的設計
	- 應用層(Application)，工程師需要自定義代碼實現邏輯的批次處理作業
	- 核心層(Batch Core)，提供Spring Batch啟動、控制所需要的核心類，如：`JobLauncher`、`Job`、`Step`…等
	- 基礎架構層(Batch Infrastructure)，提供通用的讀、寫、服務處理

## 🍃預備知識
- [[🍃SpringBoot_00_Overview]]
- [[🗄️JPA_00_Overview]]

## 🍃批次處理流程
![SpringBatch_00_Overview_02_批次處理流程](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/02_Java/09_%E6%8A%80%E8%A1%93%E6%A1%86%E6%9E%B6/%F0%9F%8D%83SpringBatch/images/SpringBatch_00_Overview_02_%E6%89%B9%E6%AC%A1%E8%99%95%E7%90%86%E6%B5%81%E7%A8%8B.png?raw=true)

- `JobLauncher`，作業調度器，批次作業啟動入口
- `Job`，批次作業，執行批次任務邏輯
- `Step`，批次作業步驟，一個批次作業由多個步驟組成，完成所有的步驟後批次作業才算執行完畢
- `ItemReader`，從數據源(文件、數據庫、隊列…)中讀取`Item`數據記錄
- `ItemProcessor`，`Item`數據加工邏輯，如：數據清洗、數據轉換、數據過濾、數據校驗…
- `ItemWriter`，將`Item`數據記錄寫入數據源(文件、數據庫、隊列…)
- `JobRepository`，保存`Job`/檢索`Job`信息，此時需要持久化`Job`，而`JobRepository`就是持久化的接口
