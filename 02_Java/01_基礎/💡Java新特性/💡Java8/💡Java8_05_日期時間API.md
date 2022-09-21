# Java8_05_日期時間API
`java.time`
[Java8 官方API文檔 java.time](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)

- 相較於之前`Date`、`Calendars`、`SimpleDateFormat`…所在的包一致，且API的命名規則也有規律
- Java7及之前的日期時間API是線程不安全的，而Java8的則是線程安全的

## 💡介紹
### 線程不安全錯誤
```java
@Test
@DisplayName("原日期時間API，線程不安全問題")
public void testcase01() throws Exception {
	SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
	ExecutorService pool = Executors.newFixedThreadPool(5);
	Callable<Date> task = () -> sdf.parse("20210923");
	List<Future<Date>> results = new ArrayList<>();

	// 使用線程池執行
	for (int i = 0; i < 10; i++) {
		results.add(pool.submit(task));
	}
	// 打印結果
	for (Future<Date> result : results) {
		System.out.println(result.get().toLocaleString());
	}

	pool.shutdown();
}
```

### Java8前解決方法
```java
public class T17_DateTimeAPI {
    @Test
    @DisplayName("原日期時間API，解決線程不安全")
    public void testcase02() throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(5);
        Callable<Date> task = () -> SolveThreadNotSave.convert("20210923");
        List<Future<Date>> results = new ArrayList<>();

        // 使用線程池執行
        for (int i = 0; i < 10; i++) {
            results.add(pool.submit(task));
        }
        // 打印結果
        for (Future<Date> result : results) {
            System.out.println(result.get().toLocaleString());
        }

        pool.shutdown();
    }

    /**
     * 解決Java8前日期時間API線程不安全的方法
     * 
     * @author 2106002
     *
     */
    static class SolveThreadNotSave {
        // 線程中的局部變量
        private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>() {
            protected DateFormat initialValue() {
                return new SimpleDateFormat("yyyyMMdd");
            }
        };

        public static Date convert(String source) throws ParseException {
            return threadLocal.get().parse(source);
        }
    }
}
```

### Java8後解決方法
```java
@Test
@DisplayName("Java8日期時間API，線程安全")
public void testcase03() throws Exception {
	ExecutorService pool = Executors.newFixedThreadPool(5);
	DateTimeFormatter dtf = DateTimeFormatter.BASIC_ISO_DATE;
	Callable<LocalDate> task = () -> LocalDate.parse("20210923", dtf);// 因為多線程都會新建實例，所以線程安全
	List<Future<LocalDate>> results = new ArrayList<>();

	// 使用線程池執行
	for (int i = 0; i < 10; i++) {
		results.add(pool.submit(task));
	}
	// 打印結果
	for (Future<LocalDate> result : results) {
		System.out.println(result.get().toString());
	}
}
```

## 💡本地日期時間 LocalDateTime
LocalDate、LocalTime、LocalDateTime的實例是==不可變的對象==，使用ISO-8601格式，但不包含時區的信息

- `LocalDateTime.now()`，取得當前日期時間
- `of()`，取得指定日期時間
- `plusXxx()`，加日期時間，無論做什麼改變都會返回新的對象
- `minusXxx()`，減日期時間
- 單獨取得信息
	- `getYear()`
	- `getMonth().getValue()`
	- `getDayOfYear()`
	- `getHour()`
	- `getMinute()`
	- `getSecond()`

```java
// LocalDate、LocalTime、LocalDateTime使用方式都一樣
// LocalDateTime.now()，取得當前日期時間
LocalDateTime ldt1 = LocalDateTime.now();
System.out.println(ldt1);

// of()，取得指定日期時間
LocalDateTime ldt2 = LocalDateTime.of(2021, 12, 23, 5, 45, 34, 0);
System.out.println(ldt2);

// plusXxx()，加日期時間，無論做什麼改變都會返回新的對象
LocalDateTime ldt3 = ldt2.plusYears(1);
System.out.println(ldt3);

// minusXxx()，減日期時間
LocalDateTime ldt4 = ldt3.minusMonths(5);
System.out.println(ldt4);

// 單獨取得信息
System.out.println("Year : " + ldt1.getYear());
System.out.println("Month : " + ldt1.getMonth().getValue());
System.out.println("Day : " + ldt1.getDayOfYear());
System.out.println("Hour : " + ldt1.getHour());
System.out.println("Minute : " + ldt1.getMinute());
System.out.println("Second : " + ldt1.getSecond());
```

## 💡時間戳 Instant
- `Instant.now()`，取得當前日期時間信息，默認取UTC時區
- `atOffset(ZoneOffset.ofHours(n))`，加入時區信息
- `toEpochMilli()`，轉為時間戳，以UNIX元年(1970/01/01)到指定時間的毫秒數
- `ofEpochSecond()`，以秒數指定時間

```java
// 以UNIX元年(1970/01/01)到指定時間的毫秒數
Instant ins1 = Instant.now();
System.out.println(ins1.toString());// 默認取UTC時區

// 轉為不同時區
OffsetDateTime atOffset = ins1.atOffset(ZoneOffset.ofHours(8));// 加八小時的時區
System.out.println(atOffset.toString());

// 轉為時間戳(毫秒值)
System.out.println(ins1.toEpochMilli());

// ofEpochSecond()，以秒數指定時間
Instant ins4 = Instant.ofEpochSecond(1000);// 1970-01-01T00:16:40Z
System.out.println(ins4);
```

## 💡時間日期間隔
### 日期間隔 Period
- `Period.between(ld1, ld2)`，計算兩個日期的間隔
- `getXxx()`，取得相應日期間隔信息

```java
// Period，日期間隔
LocalDate ld1 = LocalDate.now();
LocalDate ld2 = LocalDate.of(2005, 12, 25);
Period period = Period.between(ld1, ld2);
System.out.println(period.getYears());
System.out.println(period.getMonths());
System.out.println(period.getDays());
```

### 時間間隔 Duration
- `Duration.between(ins1, ins2)`，計算兩個時間的間隔
- `toXxx()`，取得相應時間間隔信息

```java
// Duration，時間間隔
Instant ins1 = Instant.now();
LocalTime lt1 = LocalTime.now();
Thread.sleep(2000);
Instant ins2 = Instant.now();
LocalTime lt2 = LocalTime.now();
System.out.println("Instant : " + Duration.between(ins1, ins2).toMillis());// 取得毫秒顯示
System.out.println("LocalTime : " + Duration.between(lt1, lt2).toMillis());
```

## 💡時間校正器 TemporalAdjuster
[Java8 官方API文檔 TemporalAdjusters](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalAdjusters.html)

```java
LocalDateTime ldt1 = LocalDateTime.now();
System.out.println("Today : \t" + ldt1);

// withXxx()，取得指定日期時間
LocalDateTime ldt2 = ldt1.withDayOfMonth(10);
System.out.println("10 day of month : " + ldt2);

// with(TemporalAdjusters.xxx())，取得相應邏輯的日期時間
LocalDateTime ldt3 = ldt1.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
System.out.println("next monday : " + ldt3);

// 自定義日期時間邏輯
// 因為TemporalAdjuster是函數式接口，因此可通過實現此接口自定義
// 計算下一個工作日
LocalDateTime ldt4 = ldt1.with((d) -> {
	LocalDateTime temp = (LocalDateTime) d;
	DayOfWeek dow = temp.getDayOfWeek();
	if (dow.equals(DayOfWeek.FRIDAY)) {
		return temp.plusDays(3);
	} else if (dow.equals(DayOfWeek.SATURDAY)) {
		return temp.plusDays(2);
	} else {
		return temp.plusDays(1);
	}
});
System.out.println("next workday : " + ldt4);
```

## 💡日期時間格式化 DateTimeFormatter
- 格式設置
	- `DateTimeFormatter.Xxx`，使用自帶格式
	- `DateTimeFormatter.ofPattern(String s)`，自定義格式
- `format(DateTimeFormatter dtf)`，LocalDateTime格式化為String
- `LocalDateTime.parse()`，String解析為LocalDateTime
	注意formatter不可使用hh(12小時制)，會出現`DateTimeParseException`

```java
LocalDateTime ldt1 = LocalDateTime.now();
System.out.println("Original : " + ldt1);

// 使用DateTimeFormatter.Xxx自帶的格式
// format()，LocalDateTime格式化為String
DateTimeFormatter dtf1 = DateTimeFormatter.ISO_DATE;
String str1 = ldt1.format(dtf1);
System.out.println("Format : " + str1);

// DateTimeFormatter.ofPattern(String s)，自定義格式
DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
String str2 = ldt1.format(dtf2);
System.out.println("Format : " + str2);

// LocalDateTime.parse()，String解析為LocalDateTime
LocalDateTime ldt3 = LocalDateTime.parse(str2, dtf2);
System.out.println("Parse : " + ldt3);
```

## 💡時區 ZonedDateTime
ZonedDateTime、ZonedDate、ZonedTime

- `ZoneId.getAvailableZoneIds()`，取得所有時區
- `LocalDateTime.now(ZoneId zone)`，取得指定時區的當前時間
- `atZone(ZoneId zone)`，加上時差標記，但時間不會改

```java
// ZonedDateTime、ZonedDate、ZonedTime
// 所有所有時區
Set<String> zoneSet = ZoneId.getAvailableZoneIds();
zoneSet.forEach(System.out::println);

// LocalDateTime.now(ZoneId zone)，取得指定時區的當前時間
LocalDateTime ldt1 = LocalDateTime.now(ZoneId.of("Pacific/Majuro"));
System.out.println("Original : " + LocalDateTime.now());
System.out.println(ldt1);

// atZone(ZoneId zone)，加上時差標記，但時間不會改
LocalDateTime ldt2 = LocalDateTime.now();
ZonedDateTime zdt2 = ldt2.atZone(ZoneId.of("Pacific/Majuro"));
System.out.println(zdt2);
```