# Java8_06_注解
## 💡重覆注解
### 注解類
```java
// 需要標示容器類
@Repeatable(T21_Annotations.class)
@Retention(RUNTIME)
@Target({ TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, ANNOTATION_TYPE, PACKAGE, TYPE_PARAMETER, TYPE_USE })
public @interface T21_Annotation {
    String value() default "mickey";
}
```

### 容器類
```java
@Retention(RUNTIME)
@Target({ TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, ANNOTATION_TYPE, PACKAGE, TYPE_PARAMETER, TYPE_USE })
public @interface T21_Annotations {
    // 重複其他注解的容器類，需要新增與被重複注解變量;
    T21_Annotation[] value();
}
```

### 使用重複注解
```java
public class T21_MutiAnnotation {
    @Test
    @DisplayName("取得注解以反射")
    public void testcase() throws Exception {
        Arrays.stream(T21_MutiAnnotation.class//
        .getMethod("test")//
        .getAnnotationsByType(T21_Annotation.class))//
        .forEach((a) ->System.out.println(a.value()));
    }

    // 設置完成後，相同注解就可以使用多次
    @T21_Annotation(value = "Hello")
    @T21_Annotation(value = "World")
    public void test() {}
}
```

## 💡類型注解
注解類`@Target`需要另外聲明`ElementType.TYPE_PARAMETER`後就可使用，類型注解會應用在編釋時檢查類型值(如是否不為null)。

```java
public void test(@T21_Annotation(value = "test type paramter") String str) {}
```

