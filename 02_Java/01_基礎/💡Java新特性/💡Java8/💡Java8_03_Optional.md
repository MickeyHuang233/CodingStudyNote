# Java8_03_Optional
`java.util.Optional`為容器類，在建立Optional或取出時，會報出NullPointerException，因此此類的目地在於當出現`java.lang.NullPointerException`時，比較好debug。

## 💡建立Optional
- `Optional.of(T t)`，建立一個Optional實例
- `Optional.empty()`，建立空的Optional實例
- `Optional.ofNullable(T t)`，若t不為null則建立實例，否則建立空實例
	相當於`of()`和`empty()`的綜合使用

```java
@Test
@DisplayName("建立Optional實例")
public void testcase01() {
	// Optional.of(T t)，建立一個Optional實例
	Optional<T15_Employee> opt1 = Optional.of(new T15_Employee("test"));
	T15_Employee emp1 = opt1.get();
	System.out.println(emp1.toString());

	// Optional.empty()，建立空的Optional實例
	Optional<T15_Employee> opt2 = Optional.empty();
	System.out.println(opt2.get().toString());

	// Optional.ofNullable(T t)，若t不為null則建立實例，否則建立空實例
	// 相當於of()和empty()的綜合使用
	Optional<T15_Employee> opt3 = Optional.ofNullable(null);
	System.out.println(opt3.get().toString());
}
```

## 💡操作Optional
- `isPresent()`，判斷是否有值
- `orElse(T t)`，若調用對象為null則返回t，反之則正常使用
- `orElseGet(Supplier s)`，若調用對象為null則返回Supplier建立的對象，反之則正常使用
- `map(Function f)`，有值則對其處理，反之則不處理
- `flatMap(Function f)`，和`map()`類似，但一定要返回Optional

```java
@Test
@DisplayName("操作Optional")
public void testcase02() {
	// isPresent()，判斷是否有值
	Optional<T15_Employee> opt1 = Optional.ofNullable(null);
	System.out.println("opt1 is NOT null : " + opt1.isPresent());

	// orElse(T t)，若調用對象為null則返回t，反之則正常使用
	T15_Employee emp2 = opt1.orElse(new T15_Employee("NullEmp"));
	System.out.println(emp2.toString());

	// orElseGet(Supplier s)，若調用對象為null則返回Supplier建立的對象，反之則正常使用
	T15_Employee emp3 = opt1.orElseGet(T15_Employee::new);
	System.out.println(emp3.toString());

	// map(Function f)，有值則對其處理，反之則不處理
	//        Optional<T15_Employee> opt4 = Optional.ofNullable(null);
	Optional<T15_Employee> opt4 = Optional.ofNullable(new T15_Employee("Mickey", 233, 233333.5, T15_Employee.Status.BUSY));
	Optional<String> mapName = opt4.map(T15_Employee::getName);
	System.out.println("mapName : " + mapName.get());

	// flatMap(Function f)，和map()類似，但一定要返回Optional
	Optional<String> flatMapName = opt4.flatMap((e) -> Optional.of(e.getName()));
	System.out.println("flatMapName : " + flatMapName.get());
}
```
