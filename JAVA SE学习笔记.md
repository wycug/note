

# JAVA SE学习笔记

enum枚举

Java 枚举是一个特殊的类，一般表示一组常量，比如一年的 4 个季节，一个年的 12 个月份，一个星期的 7 天，方向有东南西北等。Java 枚举类使用 enum 关键字来定义，各个常量使用逗号 **,** 来分割。

```java

enum Color { 
	RED, GREEN, BLUE; 
} 
```

等价于：

```java
class Color{
     public static final Color RED = new Color();
     public static final Color BLUE = new Color();
     public static final Color GREEN = new Color();
}
```

一般使用方法。

```java

public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    // 普通方法  
    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    // get set 方法  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  
}  
```

## 一、基础

### 1、跳出多重循环

1）for循环最外面添加标签，break 标签跳出循环。

```
flag: for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 5; j++) {
                System.out.println("outloopByBreakLikeGoto==j==" + j);
                if (j == 2) break flag;
            }
            System.out.println("outloopByBreakLikeGoto==i====>>>" + i);
        }
```

2）设置一个boolean类型的标签flag，初始为false。当需要跳出循环时候设置为true。在循环条件中添加&&flag。

3）把多重循环单独写到一个函数里，要退出循环时候直接return。

### 2、 请你讲讲&和&&的区别

&是按位与，&&是逻辑与。&&运算符是短路与运算，当符合左边为false时会直接放回false，不会继续判断符合右边。判断用户名不为null且不为空字符串时username != null &&!username.equals("")，如果用&会返回空指针异常。

### 3、int和Integer的区别

Java是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入了基本数据类型，但是为了能够将这些基本数据类型当成对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class），int的包装类就是Integer。Integer的源码可以看到，Interger构造函数就是给内部变量int value赋值。

```java
private final int value;

/**
 * Constructs a newly allocated {@code Integer} object that
 * represents the specified {@code int} value.
 *
 * @param   value   the value to be represented by the
 *                  {@code Integer} object.
 */
public Integer(int value) {
    this.value = value;
}
```

### 4、我们在web应用开发过程中经常遇到输出某种编码的字符，如iso8859-1等，请你讲讲如何输出一个某种编码的字符串？

读取iso8859-1字符串的比特串，用String的构造函数，输入想要的编码格式作为参数。

```java
Public String translate (String str) {
 String tempStr = “”;
 try {
 tempStr = new String(str.getBytes(“ISO-8859-1″), “GBK”);
 tempStr = tempStr.trim();
 }
 catch (Exception e) {
 System.err.println(e.getMessage());
 }
 return tempStr;
 }
```

### 5、请你说明StringBuilder 和StringBuffer的区别

他们都继承与AbstractStringBuilder抽象类，不过StringBuffer为了线程安全在很大方法上都加了锁。

```java
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

### 6、String和StringBuffer的区别

String 是不可修改的，当做字符串拼接时，使用方法一，jvm是先创建一个新的String str把两字符串拼接起来，旧的str被GC回收。频繁拼接导致str需要反复创建销毁，所以String拼接很慢。

```
//方法一
String str = "hello";
str = str + "world";
```

StringBuffer是直接在对象增加内容，不需要JVM重新创建对象。