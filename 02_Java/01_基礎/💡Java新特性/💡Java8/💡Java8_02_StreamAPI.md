# Java8_02_StreamAPI
`java.util.stream`
Java8 官方API文檔：
	1. [java.util.stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-frame.html)
	2. [Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)

## 💡介紹
- 概念：流(Stream)是數據渠道，用於操作數據源所産生的元素序列(著重計算)。
- 注意：
	1. Stream僅做數據處理，不會儲存
	2. Stream計算、篩選時不會改變原對象，而是返回新Stream
	3. Stream操作是延遲執行的，也就是說需要使用Stream計算結果時才會執行
-  Stream流使用流程：
 	1. Stream建立
 	2. 中間操作(篩選/計算)
 	3. 終止操作(終端操作)，必須觸發終止操作，否則Stream不會執行任何中間操作

## 💡建立Stream流
- 通過Collection提供`stream()`或`parallelStream()`
	- `stream()`，建立串行流
	- `parallelStream()`，建立并行流
- 通過靜態方法`Arrays.stream()`獲取
- 通過靜態方法`Stream.of()`獲取
- 建立無限流
	- `Stream.iterate(T t, Operator<T>)`，使用迭代建立無限流(內容會一直建立)
	- `Stream.generate(Supplier<T>)`，使用迭代建立無限流

```java
@Test
@DisplayName("Stream建立")
public void testcase01() {
	// 方法一：通過Collection提供stream()或parallelStream()
	List<String> list = new ArrayList<>();
	Stream<String> stream01 = list.stream();
	Stream<String> Stream02 = list.parallelStream();

	// 方法二：通過靜態方法Arrays.stream()獲取
	T06_Employee[] emps = new T06_Employee[10];
	Stream<T06_Employee> stream03 = Arrays.stream(emps);

	// 方法三：通過靜態方法Stream.of()獲取
	Stream<String> stream04 = Stream.of("aaa", "bbb", "ccc");

	// 方法四：使用迭代建立無限流(內容會一直建立)
	UnaryOperator<Integer> bo = (i) -> i + 2;
	Stream<Integer> stream05 = Stream.iterate(0, bo);// 從0開始建立偶數
	stream05.forEach(System.out::println);

	// 方法五：使用迭代建立無限流
	Supplier<Double> supplier = Math::random;
	Stream<Double> stream06 = Stream.generate(supplier);
	stream06.forEach(System.out::println);
}
```

## 💡Stream中間操作
### 篩選、切片
- `filter()`，從Stream中按條件篩選元素
- `limit()`，截斷流，只取前面幾個元素
- `skip()`，跳過元素，從第n個元素開始取(與`limit()`互補)，若流不足n個則返回空流
- `distinct()`，篩選，通過`hashCode()`和`equals()`去重，如果有特殊去重邏輯，需要重寫這兩個方法

```java
@Test
@DisplayName("Stream中間操作，篩選、切片")
public void testcase01() {
	// filter，從Stream中按條件篩選元素
	getEmployees().stream()//
			.filter((e) -> e.getAge() > 30)//
			.forEach(System.out::println);// 終止操作，一次性執行全部中間操作

	// limit，截斷流，只取前面幾個元素
	getEmployees().stream()//
			.limit(2)//
			.forEach(System.out::println);

	// skip，跳過元素，從第n個元素開始取，若流不足n個則返回空流
	// 與limit互補
	getEmployees().stream()//
			.skip(2)//
			.forEach(System.out::println);

	// distinct，篩選，通過hashCode()和equals()去重，如果有特殊去重邏輯，需要重寫這兩個方法
	getEmployees().stream()//
			.distinct()//
			.forEach(System.out::println);
}

	/**
 * 取得測試資料
 * 
 * @return
 */
private List<T07_Employee> getEmployees() {
	return Arrays.asList(//
			new T07_Employee("Mickey", 22, 30000),//
			new T07_Employee("Jack", 33, 55000),//
			new T07_Employee("Molly", 44, 45000),//
			new T07_Employee("Tai", 17, 25000),//
			new T07_Employee("Maple", 20, 36000),//
			new T07_Employee("John", 55, 38000),//
			new T07_Employee("Mickey", 22, 32000));
}
```

### 映射
- `map()`，將元素轉換成其他形式或提取信息
- `flatMap()`，接收一個函數，將流中的每個值轉為另一個流，再把所有的流連成一個流

```java
@Test
@DisplayName("Stream中間操作，映射")
public void testcase02() {
	// map，將元素轉換成其他形式或提取信息
	getEmployees().stream()//
			.map((e) -> e.getName().toUpperCase())//
			.forEach(System.out::println);

	// flatMap，接收一個函數，將流中的每個值轉為另一個流，再把所有的流連成一個流
	getEmployees().stream()//
			.map((e) -> e.getName())//
			.flatMap((str) -> {
				List<Character> chars = new ArrayList<>();
				for (Character c : str.toCharArray()) {
					chars.add(c);
				}
				return chars.stream();// 每個元素再返回Stream<Character>
			})//
			.forEach(System.out::println);
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T07_Employee> getEmployees() {
	return Arrays.asList(//
			new T07_Employee("Mickey", 22, 30000),//
			new T07_Employee("Jack", 33, 55000),//
			new T07_Employee("Molly", 44, 45000),//
			new T07_Employee("Tai", 17, 25000),//
			new T07_Employee("Maple", 20, 36000),//
			new T07_Employee("John", 55, 38000),//
			new T07_Employee("Mickey", 22, 32000));
}
```

### 排序
- `sorted()`，自然排序，Comparable
- `sorted(Comparator com)`，定制排序，Comparator

```java
@Test
@DisplayName("Stream中間操作，排序")
public void testcase03() {
	// sorted，自然排序，Comparable
	getEmployees().stream()//
			.map((e) -> e.getName())//
			.sorted()//
			.forEach(System.out::println);

	// sorted(Comparator com)，定制排序，Comparator
	getEmployees().stream()//
			.sorted((e1, e2) -> {
				if (e1.getAge() == e2.getAge()) {
					return (e1.getSalary() - e1.getSalary()) > 0 ? 1 : -1;
				} else {
					return e1.getAge() - e2.getAge();
				}
			})//
			.forEach(System.out::println);
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T07_Employee> getEmployees() {
	return Arrays.asList(//
			new T07_Employee("Mickey", 22, 30000),//
			new T07_Employee("Jack", 33, 55000),//
			new T07_Employee("Molly", 44, 45000),//
			new T07_Employee("Tai", 17, 25000),//
			new T07_Employee("Maple", 20, 36000),//
			new T07_Employee("John", 55, 38000),//
			new T07_Employee("Mickey", 22, 32000));
}
```

## 💡Stream終止操作
### 查找、匹配
- `allMatch()`，是否匹配所有元素
- `anyMatch()`，是否至少匹配一個元素
- `noneMatch()`，是否沒有匹配所有元素
- `findFirst()`，返回第一個元素
- `findAny()`，返回任意元素
- `count()`，返回流中元素個數
- `max()`，返回最大值
- `min()`，返回最小值
```java
@Test
@DisplayName("Stream終止操作，查找與匹配")
public void testcase01() {
	// allMatch，是否匹配所有元素
	boolean allBusy = getEmployees().stream()//
			.allMatch((e) -> e.getStatus().equals(Status.BUSY));
	System.out.println("allMatch ? " + allBusy);

	// anyMatch，是否至少匹配一個元素
	boolean anyBusy = getEmployees().stream()//
			.anyMatch((e) -> e.getStatus().equals(Status.BUSY));
	System.out.println("anyMatch ? " + anyBusy);

	// noneMatch，是否沒有匹配所有元素
	boolean noneMatch = getEmployees().stream()//
			.noneMatch((e) -> e.getAge() < 18);
	System.out.println("noneMatch ? " + noneMatch);

	// findFirst，返回第一個元素
	Optional<T11_Employee> topEmp = getEmployees().stream()//
			.sorted((e1, e2) -> (e1.getSalary() - e2.getSalary() > 0) ? -1 : 1)//
			.findFirst();
	System.out.println("findFirst ? " + topEmp.get().toString());

	// findAny，返回任意元素
	Optional<T11_Employee> findAny = getEmployees().parallelStream()// 使用并行流
			.filter((e) -> e.getStatus().equals(Status.FREE))//
			.findAny();
	System.out.println("findAny ? " + findAny.get().toString());

	// count，返回流中元素個數
	long count = getEmployees().stream()//
			.count();
	System.out.println("count ? " + count);

	// max，返回最大值
	Optional<T11_Employee> max = getEmployees().stream()//
			.max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
	System.out.println("max ? " + max.get().toString());

	// min，返回最小值
	Optional<T11_Employee> min = getEmployees().stream()//
			.min((e1, e2) -> Integer.compare(e1.getAge(), e2.getAge()));
	System.out.println("min ? " + min.get().toString());
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T11_Employee> getEmployees() {
	return Arrays.asList(//
			new T11_Employee("Mickey", 22, 30000, Status.BUSY),//
			new T11_Employee("Jack", 33, 55000, Status.VOCATION),//
			new T11_Employee("Molly", 44, 45000, Status.FREE),//
			new T11_Employee("Tai", 18, 25000, Status.BUSY),//
			new T11_Employee("Maple", 20, 36000, Status.FREE),//
			new T11_Employee("John", 55, 38000, Status.VOCATION),//
			new T11_Employee("Mickey", 22, 32000, Status.BUSY));
}
```

### 歸約
- 歸約：將流中元素反複結合，得到一個值
	- reduce(T identity, BinaryOperator)，identity為初始值
	- reduce(BinaryOperator)，因為無初始值，返回可能為空，故返回Operator

```java
@Test
@DisplayName("Stream終止操作，歸約")
public void testcase02() {
	// 歸約：將流中元素反複結合，得到一個值
	// reduce(T identity, BinaryOperator)，identity為初始值
	Double sum1 = getEmployees().stream()//
			.map((e) -> e.getSalary())// .map(T11_Employee::getSalary)
			.reduce(0D, Double::sum);// 
	System.out.println("salary sum1 : " + sum1);

	// reduce(BinaryOperator)，因為無初始值，返回可能為空，故返回Operator
	Optional<Double> sum2 = getEmployees().stream()//
			.map(T11_Employee::getSalary)//
			.reduce(Double::sum);
	System.out.println("salary sum2 : " + sum2.get());
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T11_Employee> getEmployees() {
	return Arrays.asList(//
			new T11_Employee("Mickey", 22, 30000, Status.BUSY),//
			new T11_Employee("Jack", 33, 55000, Status.VOCATION),//
			new T11_Employee("Molly", 44, 45000, Status.FREE),//
			new T11_Employee("Tai", 18, 25000, Status.BUSY),//
			new T11_Employee("Maple", 20, 36000, Status.FREE),//
			new T11_Employee("John", 55, 38000, Status.VOCATION),//
			new T11_Employee("Mickey", 22, 32000, Status.BUSY));
}
```

### 收集
#### 流轉為集合
- `collect(Collector col)`，將流轉換為其他集合，用於給Strem中元素做匯總的方法
	- `Collectors.toList()`
	- `Collectors.toSet()`
	- `Collectors.toMap()`
	- `Collectors.toCollection()`，用於建立Collection子類(可參考：[[🍀JavaSE_03_集合]])，如HashSet、ArrayList…

注意：Map無`stream()`方法使用，若要使用需轉為EntrySet，如：`map.entrySet().stream()`

```java
@Test
@DisplayName("Stream終止操作，收集，將流轉換為其他集合")
public void testcase03() {
	// collect(Collector col)，將流轉換為其他集合，用於給Strem中元素做匯總的方法
	// Collectors.toList()
	List<String> list = getEmployees().stream()//
			.map(T11_Employee::getName)//
			.collect(Collectors.toList());
	System.out.println(list.toString());

	// Collectors.toSet()
	Set<String> set = getEmployees().stream()//
			.map(T11_Employee::getName)//
			.collect(Collectors.toSet());
	System.out.println(set.toString());

	// Collectors.toMap()
	Map<String, Double> map = getEmployees().stream()//
			.limit(6)//
			.collect(Collectors.toMap(x -> x.getName(), x -> x.getSalary()));
	map.entrySet().stream()//
			.forEach(System.out::println);

	// Collectors.toCollection()，用於建立Collection子類，如HashSet、ArrayList…
	Collection<String> array = getEmployees().stream()//
			.map(T11_Employee::getName)//
			.collect(Collectors.toCollection(ArrayList::new));
	System.out.println(array.toString());
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T11_Employee> getEmployees() {
	return Arrays.asList(//
			new T11_Employee("Mickey", 22, 30000, Status.BUSY),//
			new T11_Employee("Jack", 33, 55000, Status.VOCATION),//
			new T11_Employee("Molly", 44, 45000, Status.FREE),//
			new T11_Employee("Tai", 18, 25000, Status.BUSY),//
			new T11_Employee("Maple", 20, 36000, Status.FREE),//
			new T11_Employee("John", 55, 38000, Status.VOCATION),//
			new T11_Employee("Mickey", 22, 32000, Status.BUSY));
}
```

#### 集合操作
-  `Collectors.counting()`，計算集合長度
-  `Collectors.averagingXxx()`，計算平均值
-  `Collectors.summingXxx()`，計算總和
-  `Collectors.maxBy()`，比較最大值，可返回相應集合
-  `Collectors.minBy()`比較最小值，可返回相應集合
-  `Collectors.joining()`，連接字符串
-  `Collectors.summarizingXxx()`，取得相應的計算信息，返回DoubleSummaryStatistics
	-  `getSum()`
	-  `getAverage()`
	-  `getCount()`
	-  `getMax()`
	-  `getMin()`
- `Collectors.groupingBy()`，分組，也可多級分組(用法參考下方例子)
- `partitioningBy()`，切片，返回Map分為符合條件、不符合條件

```java
@Test
@DisplayName("Stream終止操作，收集，集合操作")
public void testcase04() {
	// 計算集合長度
	Long collectorSize = getEmployees().stream()//
			.collect(Collectors.counting());
	System.out.println("collectorSize : " + collectorSize);

	// 計算平均值
	Double average = getEmployees().stream()//
			.collect(Collectors.averagingDouble(T11_Employee::getSalary));
	System.out.println("salary average : " + average);

	// 計算總和
	Double sum = getEmployees().stream()//
			.collect(Collectors.summingDouble(T11_Employee::getSalary));
	System.out.println("salary sum : " + sum);

	// 比較最大工資員工信息
	Optional<T11_Employee> maxEmp = getEmployees().stream()//
			.collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
	System.out.println("maxEmp : " + maxEmp.get());

	// 取得最小工資
	Optional<Double> minSalary = getEmployees().stream()//
			.map(T11_Employee::getSalary)//
			.collect(Collectors.minBy(Double::compare));
	System.out.println("minSalary : " + minSalary);

	// 連接字符串
	String nameJoin = getEmployees().stream()//
			.map(T11_Employee::getName)//
			.collect(Collectors.joining(", "));
	System.out.println("name join : " + nameJoin);

	// 取得相應的計算信息
	DoubleSummaryStatistics doubleSummary = getEmployees().stream()//
			.collect(Collectors.summarizingDouble(T11_Employee::getSalary));
	System.out.println("doubleSummary.getAverage() : " + doubleSummary.getAverage());
	System.out.println("doubleSummary.getCount() : " + doubleSummary.getCount());
	System.out.println("doubleSummary.getSum() : " + doubleSummary.getSum());
	System.out.println("doubleSummary.getMax() : " + doubleSummary.getMax());
	System.out.println("doubleSummary.getMin() : " + doubleSummary.getMin());

	// 分組
	Map<Status, List<T11_Employee>> groupMap = getEmployees().stream()//
			.collect(Collectors.groupingBy(T11_Employee::getStatus));
	groupMap.entrySet().stream()//
			.forEach(System.out::println);

	// 多級分組
	Map<Status, Map<String, List<T11_Employee>>> mutiGroupMap = getEmployees().stream()//
			.collect(Collectors.groupingBy(T11_Employee::getStatus, Collectors.groupingBy((e) -> {
				if (((T11_Employee) e).getSalary() <= 30000) {
					return "低薪";
				} else if (e.getSalary() <= 50000) {
					return "中薪";
				} else {
					return "高薪";
				}
			})));
	mutiGroupMap.entrySet().forEach((sub1) -> {
		System.out.println(sub1.getKey());
		sub1.getValue().entrySet().stream().forEach((sub2) -> System.out.println("\t" + sub2.toString()));
	});

	// 切片，分為符合條件、不符合條件
	Map<Boolean, List<T11_Employee>> partition01 = getEmployees().stream()//
			.collect(Collectors.partitioningBy((e) -> e.getSalary() > 30000));
	partition01.entrySet().stream()//
			.forEach(System.out::println);
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T11_Employee> getEmployees() {
	return Arrays.asList(//
			new T11_Employee("Mickey", 22, 30000, Status.BUSY),//
			new T11_Employee("Jack", 33, 55000, Status.VOCATION),//
			new T11_Employee("Molly", 44, 45000, Status.FREE),//
			new T11_Employee("Tai", 18, 25000, Status.BUSY),//
			new T11_Employee("Maple", 20, 36000, Status.FREE),//
			new T11_Employee("John", 55, 38000, Status.VOCATION),//
			new T11_Employee("Mickey", 22, 32000, Status.BUSY));
}
```

## 💡串行流、并行流
ForkJoinPool，參考：[[👩‍👩‍👧‍👦Java高并發編程_03_線程池]]
- `sequential()`，轉為串行流
- `parallel()`，轉為并行流，底層是使用ForkJoinPool

```java
/**
 * 串行和并行效能比較
 * 
 * @author 2106002
 *
 */
public class T14_ForkJoinPractice {
    long addStart = 0L;
    long addEnd = 6000000000L;

    @Test
    @DisplayName("一般for循環加總")
    public void testcase01() {
        Instant startTime = Instant.now();
        BigDecimal result = BigDecimal.ZERO;
        for (long i = addStart; i <= addEnd; i++) {
            result = result.add(new BigDecimal(i));
        }
        Instant endTime = Instant.now();

        System.out.println("testcase01 result : " + result);
        System.out.println("testcase01 spend time : " + Duration.between(startTime, endTime).toMillis());// 5939
    }

    @Test
    @DisplayName("ForkJoinPool加總")
    public void testcase02() {
        Instant startTime = Instant.now();
        ForkJoinPool pool = new ForkJoinPool();
        ForkJoinTask task = new ForkJoinTask(addStart, addEnd);
        pool.execute(task);
        BigDecimal result = task.join();
        Instant endTime = Instant.now();

        System.out.println("testcase02 result : " + result);
        System.out.println("testcase02 spend time : " + Duration.between(startTime, endTime).toMillis());// 6269
    }

    @Test
    @DisplayName("Java8 Stream串行流加總")
    public void testcase03() {
        Instant startTime = Instant.now();
        BigDecimal result = LongStream.rangeClosed(addStart, addEnd)//
                .sequential()//轉為并行流，底層是使用ForkJoinPool
                .mapToObj(BigDecimal::new)//
                .reduce(BigDecimal.ZERO, (x, y) -> x.add(y));

        Instant endTime = Instant.now();
        System.out.println("testcase03 result : " + result);
        System.out.println("testcase03 spend time : " + Duration.between(startTime, endTime).toMillis());// 470
    }

    @Test
    @DisplayName("Java8 Stream并行流加總")
    public void testcase04() {
        Instant startTime = Instant.now();
        BigDecimal result = LongStream.rangeClosed(addStart, addEnd)//
                .parallel()//轉為并行流，底層是使用ForkJoinPool
                .mapToObj(BigDecimal::new)//
                .reduce(BigDecimal.ZERO, (x, y) -> x.add(y));
        
        Instant endTime = Instant.now();
        System.out.println("testcase04 result : " + result);
        System.out.println("testcase04 spend time : " + Duration.between(startTime, endTime).toMillis());// 223
    }

    class ForkJoinTask extends RecursiveTask<BigDecimal> {
        private static final long serialVersionUID = -9005156455466905014L;
        final long MAX_NUM = 5000000000L;
        long start = 0;
        long end = 0;

        public ForkJoinTask(long start, long end) {
            this.start = start;
            this.end = end;
        }

        @Override
        protected BigDecimal compute() {
            BigDecimal sum = BigDecimal.ZERO;
            List<ForkJoinTask> taskList = new ArrayList<ForkJoinTask>();

            // 執行小任務
            if (end - start <= MAX_NUM) {
                for (long i = start; (end - i <= MAX_NUM) && (end - i >= 0); i++) {
                    sum = sum.add(new BigDecimal(i));
                }
                return sum;
            }
            // 拆分任務
            long lastStart = 0;
            for (long middle = start; end - middle > MAX_NUM; middle += MAX_NUM) {
                ForkJoinTask subTask = new ForkJoinTask((middle + 1), middle + MAX_NUM);
                subTask.fork();
                taskList.add(subTask);
                lastStart = middle + MAX_NUM + 1;
            }
            ForkJoinTask subTask = new ForkJoinTask(lastStart, end);
            subTask.fork();
            taskList.add(subTask);
            // 組合回大任務
            for (ForkJoinTask task : taskList) {
                sum.add(task.join());
            }
            return sum;
        }
    }
}
```