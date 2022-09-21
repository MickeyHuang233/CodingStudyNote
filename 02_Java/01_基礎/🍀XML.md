###### tags: `📆2021年`
###### tags: `💻編程/🚋共用`
###### tags: `🗂Overview`

# XML
可擴展標記語言
作用：編寫配置文件(框架)、數據傳輸
```xml
<?xml version="1.0" encoding="UTF-8"?><!-- 聲明，一定要在第一行 -->
<goodslist><!-- 商品列表；ctrl + shift + C 註釋-->
	<good id="1001"><!-- 商品 -->
		<name>banana</name>
		<price>12</price>
		<place>guangzhao</place>
	</good>
	<good id="1002">
		<name>apple</name>
		<price>10</price>
		<place>shanghai</place>
	</good>
	<good id="1003">
		<name>mango</name>
		<price>20</price>
		<place>taiwan</place>
	</good>
</goodslist>
```

## 🍀XML約束
一般不會去寫，都是框架中出現的約束
### DTD
```DTD
<!-- 約束文檔，約束規則 -->
<?xml version="1.0"?>
<!DOCTYPE note [
  <!-- note下有四個子節點 -->
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
```

```xml
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd"> 
<!-- 可和XML放在一起(內部引入)，一般另外放(外部引入：本地、網絡) -->
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>
```

### Schema
1.  .xsd，本身也是XML文檔，比DTD勵害，用來替代DTD
2.  語法更容易閱讀
3.  功能更強大，類型更完善
4.  有命名空間
```xml
<?xml version="1.0"?>
<!-- 
targetNamespace, xmlns，命名空間
一個XML文檔可以遵守多個約束文檔，當約束使用同一個標記時可區分，相當於java的包名
 -->
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
targetNamespace="http://www.w3school.com.cn"
xmlns="http://www.w3school.com.cn"
elementFormDefault="qualified">

<xs:element name="note">
    <xs:complexType>
      <xs:sequence>
	<xs:element name="to" type="xs:string"/>
	<xs:element name="from" type="xs:string"/>
	<xs:element name="heading" type="xs:string"/>
	<xs:element name="body" type="xs:string"/>
      </xs:sequence>
    </xs:complexType>
</xs:element>

</xs:schema>
```

```xml
<?xml version="1.0"?>
<note
xmlns="http://www.w3school.com.cn"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.w3school.com.cn note.xsd"
>
<!-- xmlns:別名 -->
<xsi:to>George</xsi:to><!-- 使用別名 -->
<from>John</from><!-- 無指定用默認的約束文件 -->
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note>
```

## 🍀解析方式
DOM方式：將XML文檔加載到內存形成樹型結構，并得到一個document對象，可進行增刪改的操作。
[dom4j](https://dom4j.github.io/)，開源XML解析包

```java
public class Test_06_ParseXML {
	public static void main(String[] args) throws DocumentException {
		String url = "src/JavaWeb_01_XML/Test_01_FirstTryXML.xml";
		//解析Test_01_FirstTryXML.xml
		SAXReader reader = new SAXReader();
        Document document = reader.read(url);
        
        Element root = document.getRootElement();
        Iterator<Element> ite = root.elementIterator();
    	  //獲取並打印根元素的子元素
        while(ite.hasNext()) {
        	Element el = ite.next();
        	System.out.println(el.getName());
        	
        	//獲取並打印子元素屬性
        	Iterator<Attribute> ita = el.attributeIterator();
        	while(ita.hasNext()) {
        		Attribute ab = ita.next();
        		System.out.println(ab.getName() + ":" + ab.getValue());
        	}
        	
        	//獲取並打印名為good的子元素名
        	if(el.getName().equals("good")) {
        		Element name = el.element("name");
        		System.out.println(name.getText());
        	}
        }
	}
}
```