# JAVA基础知识

## 1、抽象与接口

### 1、相同与不同

#### **相同点**

（1）都不能被实例化 （2）接口的实现类或抽象类的子类都只有实现了接口或抽象类中的方法后才能实例化。

#### **不同点**

（1）接口只有定义，不能有方法的实现（java 1.8中可以定义default方法体，而抽象类可以有定义与实现，方法可在抽象类中实现）

（2）一个类可以实现多个接口，但一个类只能继承一个抽象类。所以，使用接口可以间接地实现多重继承。

（3）接口强调特定功能的实现，而抽象类强调所属关系

#### jdk9以后

接口和抽象类的很多实现方法都没有太大的区别了，但是回归到设计理念本身。

**接口**的设计目的，是对**类的行为进行约束**（更准确的说是一种“有”约束，因为接口不能规定类不可以有什么行为），也就是提供一种机制，可以**强制要求不同的类具有相同的行为**。它只约束了行为的有无，但不对如何实现行为进行限制。~~对“接口为何是约束”的理解，我觉得配合泛型食用效果更佳。~~

而**抽象类**的设计目的，**是代码复用**。**当不同的类具有某些相同的行为**(记为行为集合A)，且其中一部分行为的实现方式一致时（A的非真子集，记为B），**可以让这些类都派生于一个抽象类**。在这个抽象类中实现了B，避免让所有的子类来实现B，这就达到了代码复用的目的。而A减B的部分，留给各个子类自己实现。正是因为A-B在这里没有实现，所以抽象类不允许实例化出来（否则当调用到A-B时，无法执行）。



### 2、接口

jdk1.8接口新增加了default，static， jdk9新增private方法

```csharp
public interface DefaultInterface {
    // default方法
    default void defaultFunction(){
        System.out.println("this is a default function");
    }
    // static方法
    static void staticFunction(){
        System.out.println("this is a static funcion");
    }
}

```

```java
public class DefaultInterfaceImpl implements DefaultInterface{
    public static void main(String[] args){
        // 调用default方法
        new DefaultInterfaceImpl().defaultFunction();
        // 调用static方法
        DefaultInterface.staticFunction();
    }
}
```

1、实现类可以重写default方法，不能重写static方法

2、如果一个类实现了两个接口，而这两个接口拥有相同方法签名（相同的方法名、参数）、返回类型的default方法时，实现类就必须重写该default方法，否则编译器会因为不知道应该调用哪一个接口中的default方法而报错。重写接口中default方法后，编译器会执行重写后的方法。不过好的编程习惯是明智的选择方法名，避免和其它接口产生冲突。

3、如果一个类同时继承了一个类和实现了一个或多个接口，而父类中拥有和接口中default方法相同的签名、返回值的方法时，当该类未重写该方法，直接调用时，将会调用父类中的方法。

#### 官方jdk8示例

```java
public class Tester {
   public static void main(String []args) {
      LogOracle log = new LogOracle();
      log.logInfo("");
      log.logWarn("");
      log.logError("");
      log.logFatal("");
      
      LogMySql log1 = new LogMySql();
      log1.logInfo("");
      log1.logWarn("");
      log1.logError("");
      log1.logFatal("");
   }
}
final class LogOracle implements Logging { 
}
final class LogMySql implements Logging { 
}
interface Logging {
   String ORACLE = "Oracle_Database";
   String MYSQL = "MySql_Database";

   default void logInfo(String message) {
      getConnection();
      System.out.println("Log Message : " + "INFO");
      closeConnection();
   }
   default void logWarn(String message) {
      getConnection();
      System.out.println("Log Message : " + "WARN");
      closeConnection();
   }
   default void logError(String message) {
      getConnection();
      System.out.println("Log Message : " + "ERROR");
      closeConnection();
   }
   default void logFatal(String message) {
      getConnection();
      System.out.println("Log Message : " + "FATAL");
      closeConnection();
   }
   static void getConnection() {
      System.out.println("Open Database connection");
   }
   static void closeConnection() {
      System.out.println("Close Database connection");
   }
}
```

output：

```
Open Database connection
Log Message : INFO
Close Database connection
Open Database connection
Log Message : WARN
Close Database connection
Open Database connection
Log Message : ERROR
Close Database connection
Open Database connection
Log Message : FATAL
Close Database connection
```

#### 官方jdk9示例

```java
public class Tester {
   public static void main(String []args) {
      LogOracle log = new LogOracle();
      log.logInfo("");
      log.logWarn("");
      log.logError("");
      log.logFatal("");
      
      LogMySql log1 = new LogMySql();
      log1.logInfo("");
      log1.logWarn("");
      log1.logError("");
      log1.logFatal("");
   }
}
final class LogOracle implements Logging { 
}
final class LogMySql implements Logging { 
}
interface Logging {
   String ORACLE = "Oracle_Database";
   String MYSQL = "MySql_Database";

   private void log(String message, String prefix) {
      getConnection();
      System.out.println("Log Message : " + prefix);
      closeConnection();
   }
   default void logInfo(String message) {
      log(message, "INFO");
   }
   default void logWarn(String message) {
      log(message, "WARN");
   }
   default void logError(String message) {
      log(message, "ERROR");
   }
   default void logFatal(String message) {
      log(message, "FATAL");
   }
   private static void getConnection() {
      System.out.println("Open Database connection");
   }
   private static void closeConnection() {
      System.out.println("Close Database connection");
   }
}
```

### 3、抽象

#### 抽象类与抽象方法

```java
/*public abstract 返回值类型 方法名(参数);
抽象类定义的格式：
public abstract class 类名 {
}
看如下代码：*/
//员工 
public abstract class Employee{
	public abstract void work();//抽象函数。需要abstract修饰，并分号;结束
}

//manager
public class Teacher extends Employee {
	public void work() {
		System.out.println("正在赋予权限");
	}
}

//customer
public class Assistant extends Employee {
	public void work() {
		System.out.println("正在使用该系统");
	}
}

//开发人员
public class Manager extends Employee {
	public void work() {
		System.out.println("正在维护此系统");
	}
}

```

## 附录

### 1、常用注解说明

#### @FunctionalInterface

```java
#@FunctionalInterface（函数式接口）是指仅仅只包含一个抽象方法的接口。为了方便编译器检查。
@FunctionalInterface
public interface TestInterface{
	public void add();
}
```

```java
@FunctionalInterface
public interface Function<T, R> {
   R apply(T t);//相当于参数的传递者

   default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
  }

  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
 }

 static <T> Function<T, T> identity() {
    return t -> t;
 }


```

```java
import java.util.function.Function;

public class FunctionTest {
   public static void main(String[] args) {
      Function<Integer, Integer> f1 = i -> i*4;   // lambda  
       /*    
        public interface Function<T, R> {
            R apply(T t);//相当于参数的传递者
        }
        Function<Integer, Integer> f1 = i -> i*4; 
        等价于
        Function<Integer, Integer> f1{
            Integer apply(Intege t){
                return t * 4;
            }
        }
    */
      System.out.println(f1.apply(3));

      Function<Integer, Integer> f2 = i -> i+4;   // lambda      
      System.out.println(f2.apply(3));
     
      Function<String, Integer> f3 = s -> s.length();   // lambda      
      System.out.println(f3.apply("Adithya"));
     
      System.out.println(f2.compose(f1).apply(3));//先执行f1:3*4,然后执行12+4，例：3*4+4=16  说明：apply相当于参数传递者，第一次参数传递进来时3，因为需要先执行f1:i*4,此时参数值已变成12，然后再执行f2：i+4,所以最后的输出结果是：12+4=16
      System.out.println(f2.andThen(f1).apply(3));//先执行3+4，然后执行7*4，例：(3+4)*4=28  说明：与上面相反
     
      System.out.println(Function.identity().apply(10));//identity相当于静态方法，传入什么值就输出什么值
      System.out.println(Function.identity().apply("Adithya"));

   }
}

/*output
12
7
7
16
28
10
Adithya
*/
```

