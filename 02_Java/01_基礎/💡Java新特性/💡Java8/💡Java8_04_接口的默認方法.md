# Java8接口的默認方法
```java
@Test
@DisplayName("接口的默認方法")
public void testcase01() {
	System.out.println("getClassName() : " + new SubClass01().getClassName());
	System.out.println("getClassType() : " + new SubClass01().getClassType());
}

interface Interface01 {
	default String getClassName() {
		return Interface01.class.getSimpleName() + ".test()";
	}

	default String getClassType() {
		return Interface01.class.getTypeName();
	}
}

interface Interface02 {
	default String getClassType() {
		return Interface02.class.getTypeName();
	}
}

class SubClass01 implements Interface01, Interface02 {
	// 如果沒複寫接口的默認方法，調用時會直接使用接口的默認方法
	// 如果有複寫，則會調用複寫後的方法
	@Override
	public String getClassName() {
		return SubClass01.class.getSimpleName() + ".test()";
	}

	// 如果繼承多個接口有相同方法，一定要選擇要複寫哪一個接口的方法
	@Override
	public String getClassType() {
		return Interface02.super.getClassType();
	}
}
```
