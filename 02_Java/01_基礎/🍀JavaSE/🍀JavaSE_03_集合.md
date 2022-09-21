#💻編程/🌠Java/01_Java基礎
# Collection
```java
import java.util
```
![[JavaSE_03_集合_01.png]]

## 🍀List
[\[Java筆記\] 詳解 Collection-List](https://andy6804tw.github.io/2017/11/08/java-collection-intro-2/)

### ArrayList
數組，線程不安全，查詢快，修改慢
```java  
import java.util.ArrayList;

public class fruit{
	public static void main(String[] args){
		ArrayList<String>  fruit = new ArrayList<String>();
		fruit.add("Apple");//新增
		fruit.add("Banana");
		String desired= "Coconut";
		fruit.set(1,desired);//修改
		int num=fruit.size();
		for(int i=0;i<num;i++)//遍歷打印
		System.out.println(fruit.get(i));
	}
}
```

### LinkedList
鍵表，線程不安全，查詢慢，修改快
Node(Node previous, Object obj, Node next)
![[JavaSE_03_集合_02.png]]

ArrayList與LinkedList比較
|Compare|Array|LinkList|
|---|---|---|
|Add & Delete|需要做大量的資料搬移|及需改變Pointer|
|Memory Space|較浪費|需要才用|
|Data Stroage|可以對資料做直接存取|適合資料的循序存取|

## 🍀Map
用一個對象找另一個對象，Key是使用Set的方式儲存，而Value是用List的方式儲存。
![[JavaSE_03_集合_03.png]]

### HashMap
無順序，HashMap 在 put 时是根据 key 的 hashcode 进行 hash 然后放入对应的地方；效率高，線程不安全
    
```java
import java.util.Map;  
import java.util.HashMap;  

public class HashMapDemo {  
	public static void main(String args\[\]) {  
		Map<String, String> map = new HashMap<String, String>();  
		String key1 = "leon";// 宣告 key  
		String key2 = "godleon";  
		map.put(key1, "leon 的資料");// 將資料存入 Map container 中  
		map.put(key2, "godleon 的資料");  
		System.out.println(map.get(key1));// 將資料從 Map container 取出  
		System.out.println(map.get(key2));  
	}  
}
```

### HashTable
無順序，鍵不可重覆；效率低，線程安全

### TreeMap
有順序，鍵的自然順序進行排序

### LinkedHashMap
有序的 HashMap

### 自定義Map
ArrayList(key.hashCode()%arr.length) + LinkList，更有效率
Map + LisnkedList，以hashCode為索引，更有效率

## 🍀Set
集合，底實現用Map
### HashSet
無次序，不可重複，線程不安全
```java
public class HashSetDemo {
    public static void main(String args[]) {
        Set<String> set = new HashSet<String>();
        set.add("bad");
        set.add("godleon");
        set.add("good");
        set.add("godleon"); //重複的元素，在 Set 中不會存兩份
 
        // 請注意，實際上元素印出來的順序並非當初元素加入的順序
        // 那是因為 Set 有自己一套的排序方式
        Iterator iterator = set.iterator();
        while(iterator.hasNext())
            System.out.print(iterator.next() + " ");
        System.out.println();
 
        //也是可以利用 for each 的方式將元素印出
        set.remove("good");
        for(String s : set)
            System.out.print(s + " ");
        System.out.println();
    }
}
```

### LinkedHashSet
有次序，不可重複

### TreeSet
有次序(自訂排序)，不可重複，排序常用，線程不安全
```java
public class TreeSetDemo {
    public static void main(String args[]) {
        Set<String> set = new TreeSet<String>();
        set.add("godleon");
        set.add("good");
        set.add("bad");
        // 由於存入字串，因此元素會以字典的順序加以排序
        // 若要修改排序方式，可實作 java.util.Comparator interface 後
        // 在 new TreeSet 時當作其 Constructor 的參數帶入
        for(String s : set)
            System.out.print(s + " ");
        System.out.println();
    }
}
```

TreeSet，自定義順序
```java
//自定义数据类型，并在自定义的数据类型中实现CompareTo方法
class Teacher implements Comparable {
	int num;
	String name;
	Teacher(String name, int num) {
		this.num = num;
		this.name = name;
	}
	public String toString() {
		return "学号:" + num + " 姓名:" + name;
	}
	/*
		自定義的排序方法
		此字符串小于字符串参数，则返回一个小于 0 的值
		此字符串大于字符串参数，则返回一个大于 0 的值
	*/
	public int compareTo(Object o) {
		Teacher ss = (Teacher) o;
		int result = num > ss.num ? 1 : (num == ss.num ? 0 : -1);
		if (result == 0) {
			result = name.compareTo(ss.name);
		}
		return result;
	}
}

public class TreeSetTest {
	public static void main(String[] args) {
		Set<Teacher> treeSet = new TreeSet<Teacher>();
		treeSet.add(new Teacher("zhangsan", 2));
		treeSet.add(new Teacher("lisi", 1));
		treeSet.add(new Teacher("wangwu", 3));
		treeSet.add(new Teacher("mazi", 3));
		System.out.println(treeSet);//直接输出
		Iterator itTSet = treeSet.iterator();//遍历输出
		while(itTSet.hasNext())
			System.out.print(itTSet.next() + "\t");
		System.out.println();
	}
}
```

## 🍀Stack & Queue
Stack（先進後出）、Queue（先進先出）

## 🍀迭代器 Interator
hasNext()，next()，remove()
```java
for(Iterator iter = set.iterstor(); iter.hasNext();){  
	Object obj = iter.next();  
	System.out.println(obj);  
}
```











