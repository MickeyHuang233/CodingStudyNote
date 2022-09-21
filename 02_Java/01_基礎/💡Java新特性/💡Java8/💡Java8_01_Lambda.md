# Java8_01_Lambda
## 💡基礎語法
Java8中引入新的操作符`->`，Lambda操作符
- `->`左側：Lambda表達式參數，對應接口中的抽象方法的參數列表
- `->`右側：Lambda表達式執行功能，對應抽象方法功能的實現，也稱"Lambda體"
-  Lambda表達式需要"函數式接口"的支持
 	函數式接口：只有一個方法的接口，可用`@FunctionalInterface`檢查是否為函數式接口

### 無參數，無返回值
```java
@Test
@DisplayName("語法格式一：無參數，無返回值")
public void testcase01() {
	// 注意：在匿名內部類山使用同級別的函數，必須是final(java8後省略final)
	int num = 0;
	// 原本寫法
	Runnable run1 = new Runnable() {
		@Override
		public void run() {
			System.out.println("Hello Original! : " + num);
		}
	};
	new Thread(run1).run();
	// Lambda寫法
	Runnable run2 = () -> System.out.println("Hello Lambda! : " + num);
	new Thread(run2).run();
}
```

### 一個參數，無返回值
```java
@Test
@DisplayName("語法格式二：有一個參數，無返回值")
public void testcase02() {
	// 習慣寫法
	Consumer<T03_Employee> consumer1 = (e) -> System.out.println(e.toString());
	// 只有一個參數時，小括號可省略
	Consumer<T03_Employee> consumer2 = e -> System.out.println(e.toString());
	T03_Employee emp = new T03_Employee("Mickey", 233, 45000);
	consumer1.accept(emp);
}
```

### 兩個以上參數，有返回值
```java
@Test
@DisplayName("語法格式三：有兩個(含)以上參數，有返回值")
public void testcase03() {
	// Lambda可省略參數數據類型，因為JVM可根據上下文推斷數據類型
	// 如：List<String> strs = new ArrayList<>();
	// Lambda體中有多條語句，要用{}包起來
	Comparator<Integer> comparator1 = (o1, o2) -> {
		System.out.println("Do Something Else");
		return Integer.compare(o1, o2);
	};
	// Lambda體只有一條語句可省略{}、return
	Comparator<Integer> comparator2 = (o1, o2) -> Integer.compare(o1, o2);
}
```

## 💡內置核心函數式接口
Lambda表達式主要是用於實現函數式接口，但每次使用都要自己建立太過麻煩，所以Java8已經建立好幾個常用的接口使用。

| Main Interface | Primitive Interface                                                         | Function                                                                                 |
| -------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Consumer       | IntConsumer<br>LongConsumer<br>DoubleConsumer                               | void accept(int value);<br>void accept(long value);<br>void accept(double value)         |
| Function       | IntFunction\<R\><br>LongFunction\<R\><br>DoubleFunction\<R\>                | R apply(int value);<br>R apply(long value);<br>R apply(double value);                    |
| Function       | ToIntFunction\<T\>//回傳int<br>ToLongFunction\<T\><br>ToDoubleFunction\<T\> | int applyAsInt(T value);<br>long applyAsLong(T value);<br>double applyAsDouble(T value); |
| Supplier       | IntSupplier<br>LongSupplier<br>DoubleSupplier                               | int getAsInt();<br>long getAsLong();<br>double getAsDouble();                            |
| Predicate      | IntPredicate<br>LongPredicate<br>DoublePredicate                            | boolean test(int value);<br>boolean test(long value);<br>boolean test(double value);     |

### Consumer消費型接口
`Consumer<T>`，`void accept(T t)`
[Java8 官方API文檔 Consumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html)

```java
@Test
@DisplayName("消費型接口")
public void testcase01() {
	Consumer<List<T05_Employee>> consumer = (emps) -> emps.forEach(e -> System.out.println(e.getName()));
	consumer.accept(getEmployees());
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T05_Employee> getEmployees() {
	return Arrays.asList(//
			new T05_Employee("Mickey", 22, 30000),//
			new T05_Employee("Jack", 33, 55000),//
			new T05_Employee("Molly", 44, 45000),//
			new T05_Employee("Tai", 17, 25000),//
			new T05_Employee("Maple", 20, 36000),//
			new T05_Employee("John", 55, 38000));
}
```

###  Supplie供給型接口
`Supplier<T>`，`T get()`
[Java8 官方API文檔 Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)

```java
@Test
@DisplayName("供給型接口")
public void testcase02() {
	Supplier<Integer> supplier = () -> (int) (Math.random() * 100);
	T05_Employee emp = new T05_Employee("Alex", supplier.get(), 20000);
	System.out.println(emp.toString());
}
```

### Function函數型接口
`Function<T, R>`，`R apply(T t)`
[Java8 官方API文檔 Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html)

```java
@Test
@DisplayName("函數型接口")
public void testcase03() {
	Function<T05_Employee, Integer> function = (e) -> (int) (e.getAge() * e.getSalary());
	T05_Employee emp = new T05_Employee("Alex", (int) (Math.random() * 100), 20000);
	System.out.println(emp.toString());
	System.out.println("result : " + function.apply(emp));
}
```

### Predicate斷言型接口
`Predicate<T>`，`boolean test(T t)`
[Java8 官方API文檔 Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)

```java
@Test
@DisplayName("斷言型接口")
public void testcase04() {
	Predicate<T05_Employee> predicate = (e) -> e.getAge() >=30;
	List<T05_Employee> result = new ArrayList<>();
	for(T05_Employee emp:getEmployees()) {
		if(predicate.test(emp)) {
			result.add(emp);
		}
	}
	result.forEach((e) -> System.out.println(e.toString()));
}

/**
 * 取得測試資料
 * 
 * @return
 */
private List<T05_Employee> getEmployees() {
	return Arrays.asList(//
			new T05_Employee("Mickey", 22, 30000),//
			new T05_Employee("Jack", 33, 55000),//
			new T05_Employee("Molly", 44, 45000),//
			new T05_Employee("Tai", 17, 25000),//
			new T05_Employee("Maple", 20, 36000),//
			new T05_Employee("John", 55, 38000));
}
```


## 💡內置運算函數式接口
Binary，二元運算，Unary的擴充
Unary，一元運算

| Interface                                                                                               | Function                |
| ------------------------------------------------------------------------------------------------------- | ----------------------- |
| [BiConsumer\<T,U\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiConsumer.html)       | void accept(T t, U u);  |
| [BiPredicate\<T,U\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiPredicate.html)     | boolean test(T t, U u); |
| [BiFunction\<T,U,R\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BiFunction.html)     | R apply(T t, U u);      |
| [BinaryOperator\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html) | => BiFunction\<T,T,T\>  |
| [UnaryOperator\<T\>](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html)   | => Function\<T,T\>      |

```java
List<Person> ps = Arrays.asList(new Person(10), new Person(20), new Person(30));
UnaryOperator<Person> older = p -> {
    p.addAge(1);//一元運算內容，類似age = age + 1, 回傳age
    return p;
};
ps.stream().map(older).forEach(p -> System.out.println(p));
```

```java
List<String> words = Arrays.asList("hello", "hi", "Good morning");
BiPredicate<String, Integer> pre = (s, length) -> s.length() >= length;//二元運算內容
words.stream().filter(w -> pre.test(w, 5)).forEach(w -> System.out.println(w));//將符合pre物件的值都打印出來
```

## 💡方法引用
若Lambda體中可以引用已經實現的方法，則可以省略寫法
前提條件：要實現的函數式接口的參數、返回值類型，必須和已實現的方法參數、返回值類型一致

1. 對象::實例方法名
	```java
	@Test
	@DisplayName("語法格式一，對象::實例方法名")
	public void testcase01() {
		// Lambda實現函數式接口
		Consumer<String> consumer1 = (s) -> System.out.println(s);
		consumer1.accept("Hello World!");

		// Lambda方法引用
		PrintStream ps = System.out;// 實現對象
		Consumer<String> consumer2 = ps::println;
		consumer2.accept("Hello Method Ref");
	}
	```
2. 類::靜態方法名
	```java
	@Test
	@DisplayName("語法格式二，類::靜態方法名")
	public void testcase02() {
		// Lambda實現函數式接口
		Supplier<Double> supplier1 = () -> Math.random();
		System.out.println("Random : " + supplier1.get());

		// Lambda方法引用
		Supplier<Double> supplier2 = Math::random;
		System.out.println("Random : " + supplier2.get());
	}
	```
3. 類::實例方法名
	```java
	@Test
	@DisplayName("語法格式三，類::實例方法名")
	public void testcase03() {
		// Lambda實現函數式接口
		BiPredicate<String, String> bp1 = (s1, s2) -> s1.equals(s2);
		System.out.println(bp1.test("John", "Jack"));

		// Lambda方法引用
		// 如果第一個參數是此方法的調用者，第二個參數是此方法的參數時，才可使用
		BiPredicate<String, String> bp2 = String::equals;
		System.out.println(bp2.test("Mickey", "Molly"));
	}
	```

## 💡構造器引用
```java
@Test
@DisplayName("Lambda構造器引用，類名::new")
public void testcase01() {
	// Lambda實現函數式接口，無參數
	Supplier<T06_Employee> supplier1 = () -> new T06_Employee();
	System.out.println("supplier1 : " + supplier1.get().toString());

	// Lambda構造器引用，無參數
	Supplier<T06_Employee> supplier2 = T06_Employee::new;
	System.out.println("supplier2 : " + supplier2.get().toString());

	// Lambda實現函數式接口，一個參數
	Function<String, T06_Employee> function1 = (str) -> new T06_Employee(str);
	System.out.println("function1 : " + function1.apply("Mickey"));

	// Lambda構造器引用，一個參數
	// 要調用哪個構造器主要看Function傳入參數的型別
	Function<String, T06_Employee> function2 = T06_Employee::new;
	System.out.println("function2 : " + function2.apply("Maple"));

	// Lambda實現函數式接口，兩個參數
	BiFunction<String, Integer, T06_Employee> bf1 = (str, num) -> new T06_Employee(str, num);
	System.out.println("bf1 : " + bf1.apply("Tai", 34));

	// Lambda構造器引用，一個參數
	// 構造器的參數列表要與實現函數式的參數列表一致(型別、順序)
	BiFunction<String, Integer, T06_Employee> bf2 = T06_Employee::new;
	System.out.println("bf2 : " + bf2.apply("Jack", 45));
}
```

## 💡數組引用
```java
@Test
@DisplayName("Lambda數組引用，Type::new")
public void testcase02() {
	// Lambda實現函數式接口
	Function<Integer, String[]> function1 = (size) -> new String[size];
	System.out.println("function1 : size=" + function1.apply(10).length);

	// Lambda數組引用
	Function<Integer, String[]> function2 = String[]::new;
	System.out.println("function2 : size=" + function2.apply(20).length);
}
```