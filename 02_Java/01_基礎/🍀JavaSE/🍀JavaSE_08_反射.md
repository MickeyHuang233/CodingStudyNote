# JavaSE_08_反射
`java.lang.reflect`
[Java反射技术_反射教程](http://www.51gjie.com/java/82)
[Java反射獲取所有私有字段](https://code.i-harness.com/zh-TW/q/e9b1a8)
反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

## 🍀Class
`java.lang.Class`
[Java8 官方API文檔 Class](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)
```java
// 取得Class對象的方法
Class cls1 = "abc".getClass();
Class cls2 = String.class;
Class cls3 = Class.forName("java.lang.String");
// 所有的類在程序啟動時都會轉為Class類，因此取得同一個對象的Class對象都是一樣的對象。
System.out.println(clas1.getSimpleName());
System.out.println(cls1==cls2);//true
System.out.println(cls1==cls3);//true
```

## 🍀Method
[Java8 官方API文檔 Method](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html)
描述类的方法信息（包括：方法修饰符、方法名称、参数列表…等）。
- `getMethods()`，获得类的public类型的方法
- `getMethod(String name, Class[] params)`，获得类的特定方法，name参数指定方法的名字，params参数指定方法的参数类型
- `getDeclaredMethods()`，获取类中所有的方法(public、protected、default、private)
- `getDeclaredMethod(String name, Class[] params)`，获得类的特定方法，name参数指定方法的名字，params参数指定方法的参数类型
- `invoke()`，反射调用方法

```java
//使用反射第一步:获取操作类MethodDemoFieldDemo所对应的Class对象
Class<?> cls = Class.forName("com.testReflect.MethodDemo");
Object obj = cls.newInstance();

//获取public int addResult(int addNum)方法
Method addMethod = cls.getMethod("addResult", new Class[] {int.class});
System.out.println("修饰符: " + Modifier.toString(addMethod.getModifiers()));
System.out.println("返回值: " + addMethod.getReturnType());
System.out.println("方法名称: " + addMethod.getName());
System.out.println("参数列表: " + addMethod.getParameterTypes());
int result = (int) addMethod.invoke(obj, 2);
System.out.println("调用addResult后的运行结果:" + result);

//获取public String toString() 方法
Method toStringMethod = cls.getMethod("toString", new Class[] {});
System.out.println("修饰符: " + Modifier.toString(toStringMethod.getModifiers()));
System.out.println("返回值: " + toStringMethod.getReturnType());
System.out.println("方法名称: " + toStringMethod.getName());
System.out.println("参数列表: " + toStringMethod.getParameterTypes());
String msg = (String) toStringMethod.invoke(obj);
System.out.println("调用toString后的运行结果:" + msg);
```

权限总结
1.  public的static的方法：没有任何权限问题，`getMethod()`就可以取得。
2.  public的非静态的方法：没有任何权限问题，`getMethod()`就可以取得，但是，在invoke时，第一个参数必须是具体的某一个对象，static的可要可不要
3.  protected的非静态方法：必须使用`getDeclaredMethod()`，不能使用`getMethod()`，不用设置`setAccessiable(true)`
4.  friendly的非静态方法：必须使用`getDeclaredMethod()`，不能使用`getMethod()`，不用设置`setAccessiable(true)`
5.  private的非静态方法：必须使用`getDeclaredMethod()`，不能使用`getMethod()`，必须设置`setAccessiable(true)`

## 🍀Field
用于获取某个类的属性或该属性的属性值。
[Java8 官方API文檔 Field](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html)
- `Class.getDeclaredField(String name)`，该对象反映此 Class 对象所表示的类或接口的指定已声明字段（包括私有成员）
- `Class.getDeclaredField()`，该对象反映此 Class 对象所表示的类或接口的指定已声明字段（包括私有成员）
- `Class.getField(String name)`，反映此 Class 对象所表示的类或接口的指定公共成员字段
- `Class.getFields()`，该数组包含此 Class 对象所表示的类或接口的所有可访问公共字段
```java
public class Person{  
	public String name;  
	private Integer age;  
	private String sex;  
	//省略所有属性的set、get方法 
}
```

```java
public class Test{  
    public static void main(String[]args) throws NoSuchFieldException, SecurityException{  
        Person person = new Person();  
        //通过Class.getDeclaredField(String name)获取类或接口的指定已声明字段 
        Field f1 = person.getClass().getDeclaredField("name");  
        System.out.println("-----Class.getDeclaredField(String name)用法-------");  
        System.out.println(f1);
        //通过Class.getDeclaredFields()获取类或接口的指定已声明字段
        Field[] f2 = person.getClass().getDeclaredFields();
        System.out.println("-----Class.getDeclaredFields()用法-------");  
        for(Field field:f2)  
        {  
            System.out.println(field);  
        }
        //通过Class.getField(String name)返回一个类或接口的指定公共成员字段，私有成员报错。  
        Field f3 = person.getClass().getField("name");  
        System.out.println("-----Class.getField(String name)用法-------");  
        System.out.println(f3);  
        //如果获取age属性(私有成员) 则会报错  
        //Field f3=person.getClass().getField("name");  
        //通过Class.getField()，返回 Class 对象所表示的类或接口的所有可访问公共字段。  
        Field[] f4 = person.getClass().getFields();  
        System.out.println("-----Class.getFields()用法-------");  
        for(Field fields:f4)  
        {  
            //因为只有name属性为共有，因此只能遍历出name属性  
            System.out.println(fields);  
        }  
    }  
}
/*
-----Class.getDeclaredField(String name)用法-------  
public java.lang.String com.mao.test.Person.name  
-----Class.getDeclaredFields()用法-------  
public java.lang.String com.mao.test.Person.name  
private java.lang.Integer com.mao.test.Person.age  
private java.lang.String com.mao.test.Person.sex  
-----Class.getField(String name)用法-------  
public java.lang.String com.mao.test.Person.name  
-----Class.getFields()用法-------  
public java.lang.String com.mao.test.Person.name  
*/
```

## 🍀Constructor
代表某个类的构造方法，用来管理所有的构造函数的类。
[Java8 官方API文檔 Constructor](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html)
```java
class Car{
    public void run(){
        System.out.println("----running");
    }
    public Car() {
        System.out.println("Car()");
    }
    public Car(int i){
        System.out.println("Car(" + i + ")");
    }
}
```

```java
public class Temp1 {
    public static void main(String[] args) {
		// 取得constructor
		Constructor<Car> constructor = clazz.getConstructor();// 調用無參構造方法
		Constructor<Car> constructor2= clazz.getConstructor(int.class);
		// 建立相應的對象
		Car car =  constructor.newInstance();
		Car car2 = constructor2.newInstance(2);
		car.run();
		car2.run();
    }
}
/*
Car()
Car(i)
----running
----running
*/
```

```java
public static void main(String[] args) {
    //传统方式  
    String s = new String(new StringBuffer("abc"));

    //得到构造方法，参数类型为StringBuffer，得到方法的时候需要类型，用这个方法的时候需要传递同样类型的对象  
    Constructor constructor = String.class.getConstructor(StringBuffer.class);

    //构造String实例对象,编译的时候不知道具体对应哪个构造方法，所以需要传入具体类型的参数  
    //在编译的时候只知道它是一个构造方法，但不知道你是哪个类的构造方法，要做强制类型转换  
    //要理解编译时和运行时的区别  
    String str = (String) constructor.newInstance(new StringBuffer("HELLO"));

    System.out.println(s);
    System.out.println(str.charAt(4));
} 
```

## 🍀泛型 Generic
Java中泛型及給編譯器用，一互編譯完成，所有和泛型相關的類型全部刪除；為了迎合開發需要，java新增`ParameterizedType`、`GenericArrayType`、`TypeVariable`和`WildCardType`。
- [ParameterizedType](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/ParameterizedType.html)：表示參數化的類型，如`Collection<String>`
- [GenericArrayType](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/GenericArrayType.html)：表示一種元素類型是參數化類型或類型變量的數組類型。
- [TypeVariable](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/TypeVariable.html)：各種類型變量的公共父接口。
- [WildCardType](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/WildcardType.html)：表示一種通配符類型，如：`?`、`? extends Number`。

## 🍀reflect.Array
`java.lang.reflect.Array`
reflect.Array类位于java.lang.reflect包下,它是个反射工具包，全是静态方法
[Java8 官方API文檔 Array](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Array.html)
```java
int[] arr = {1,2,3,4,5};
Class<?> c = arr.getClass().getComponentType();

System.out.println("数组类型： " + c.getName());
int len = Array.getLength(arr);
System.out.println("数组长度： " + len);
System.out.print("遍历数组： ");
for (int i = 0; i < len; i++) {
	System.out.print(Array.get(arr, i) + " ");
}
System.out.println();
//修改数组
System.out.println("修改前的第一个元素： " + Array.get(arr, 0));
Array.set(arr, 0, 3);
System.out.println("修改后的第一个元素： " + Array.get(arr, 0));
```

## 🍀反射機制性能問題
`setAccessible()`，啟用(false)和禁用(true)訪問安全檢查的開關，若需要頻繁調用注解時且對硬體要求比較高，建議設置`setAccessible(ture)`以提高效率。

