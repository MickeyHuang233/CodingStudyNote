# JavaSE_07_注解
JDK5.0開始引用的新技術
- 作用：
	1. 不是程序本身，但可對程序作解釋
	2. 可被其他程序讀取
- 使用的地方：可附加在package、class、method、field…等上面
[Java8 官方API文檔 Annotation](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/Annotation.html)

## 🍀內置注解
- `@Override`：表示方法聲明打算重寫父類中的另一個方法聲明。
- `@Deprecated`：表示不鼓勵程序員使用。
- `@SuppressWarnings`：用來抑制編譯時的警告信息；[@SuppressWarnings註釋內可用的記號說明](https://www.ibm.com/support/knowledgecenter/zh-tw/SSQ2R2_9.1.1/org.eclipse.jdt.doc.user/tasks/task-suppress_warnings.htm)
```java
@SuppressWarnings("unchecked")
@SuppressWarnings(value={"unchecked", "deprecation"})
```

## 🍀元注解
作用：對注解作出解釋
```java
// 自定義注解
@ Target(value={ElementType.TYPE, ElementType.METHOD})
@Retention=(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface MyAnnotation{
	// 定義參數名及其類型，並定義默認值
	// 注意：若默認值為負數(如-1)時，表示這個值不存在
	String studentName() default "";
	int studentAge default -1;
	String[] class() default {"English", "Math"};
}
```

- `@Target`：描述使用范圍，其中ElementType中定義以下范圍：
	1. TYPE：適用 class, interface, enum
	2. FIELD：適用 field
	3. METHOD：適用 method
	4. PARAMETER：適用 method 上之 parameter
	5. CONSTRUCTOR：適用 constructor
	6. LOCAL_VARIABLE：適用區域變數
	7. ANNOTATION_TYPE：適用 annotation 型態
	8. PACKAGE：適用 package
- `@Retention`：描述注解保留策略，其中RetentionPolicy定義以下策略：
	1. SOURC，標註資訊留在原始碼（不會儲存至.class檔案）
	2. CLASS，標註資訊會儲存至.class檔案，但執行時期無法讀取
	3.  RUNTIME，標註資訊會儲存至.class檔案，但執行時期可以讀取，可通過反射讀取
- `@Documented`：要求為API 文件的一部份。
- `@Inherited`：要求子類的Annotation繼承父類。
