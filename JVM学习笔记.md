# JVM

## 虚拟机介绍

### 1、HotSpot虚拟机

hotspot虚拟机包含两个即时编译器 C1：编译耗时短但输出代码优化程度低

​															 C2：编译耗时长但输出优化质量更高

​																	（Graal编译器（java编写）开始逐步替代C2（C++编写），支持预测性优化等）

hotSpot虚拟机需要长时间的预热才能达到最佳性能，这与目前流行的微服务框架相违背（传统单体项目保证24小时全天候运行，项目很少重启，预热时间忽略不计，但是微服务框架下，各个服务可以中断和更新，预热时间长与之相悖）

APPCDS把允许加载后的类型信息缓存起来，JDK10后，APPCDS开始支持用户代码，在逐步解决这个问题。



### JVM 的组成部分

1. 类加载器子系统：将编译好的 class 文件加载到 jvm 中
2. 运行时数据区: 存储 jvm 运行过程中产生的数据
3. 执行引擎: 包括即时编译器和垃圾回收器
4. 本地接口库
5. 本地方法库

![v2-e971d8198261658e39dd6cb0886c3513_r](C:\Users\wy\Desktop\笔记\images\v2-e971d8198261658e39dd6cb0886c3513_r.jpg)





## JAVA内存区域与内存泄漏

### 内存区域

#### 1、程序计算器

#### 2、java虚拟机栈

#### 3、本地方法栈

#### 4、java堆

java heap存放对象实例，也被称为GC堆。java堆可以处于物理上不连续的内存空间中，但是逻辑上是连续的（类似磁盘文件存储）。多数虚拟机出于实现简单，会要求java堆处于连接空间。

#### 5、方法区

也被叫做Non-Heap，用于储存已被

#### 6、运行时常量区

#### 直接内存



### JAVA对象

#### 对象的创建

​	（1）一般都采用**指针碰撞**（已分配内存和未分配内存的分界线），当new一个对象时，移动指针。

​	（2）当需要new一个较大文件时，单个空闲内存不够，采用**空闲列表**，在空闲列表中选择多个空闲块。

具体采用那种方式需要看java堆的GC机制，当GC带有**空间压缩能力**（GC时移动内存块）时选择第一中方式。像CMS这种基于**清除算法的GC**机制，理论上只能采用空闲表（但是实际设计中，CMS设计了一种分配缓冲区，会先在空闲列表中收集较大的内存空间，也能只能采用指针碰撞的方式进行内存分配）。 

​		注意：内存分配存在线程安全问题

​		解决办法：（1）采用**CAS**加上失败重试的方式解决

​							（2）每个线程在创建时预先分配一小块内存，本地线程分配缓存（Thread local allocation buffer **TLAB**），在缓存空间用完了，使用**锁的方式分配新的内存**。（开启TLAB对象在实例化时可以不赋初始值）



#### 对象的内存布局

（1）对象头

​		1）Markword 用于存储hashcode、GC分代年龄、锁状态码、线程持有的锁，偏向线程ID，偏向时间戳等

​		2）类型指针，用于确定该对象时那个类的实例	

（2）实例数据

（3）对齐填充

​			hotspot任何对象的大小都必须是8字节的整数倍，对象头大小为8字节的1-2倍，如果实例数据不是8字节的倍数，需要做填充



#### 对象的定位

​	在java栈的本地变量表中的reference数据来中，一般有2种方式

​		1）reference指向句柄池中的句柄，句柄中包含对象实例数据的指针

​		2）reference直接指向对象实例数据的指针。

​	第一种方式的好处的reference指向的句柄地址是稳定的，对象移动，句柄地址不会变（改变的句柄存储的指针内容）

​	第二种方式的好处的效率高，没有中介。





## 垃圾收集器与内存分配策略

### GC机制

#### 判断对象是否存活

1）引用计数法：一个地方引用计算器+1,引用失效-1,当引用计数=0时，对象就不会在被使用了（经验）

2）可达性分析算法：GCRoots作为根节点，不在引用链上的对象被回收。

#### GC算法

##### 标记清除算法Mark-Sweep

先标记出可以回收的对象，标记完成后统一回收

缺点：1）大量标记和清除工作随着对象数量上升导致效率越来越慢

​			2）会产生大量不碎片空间，导致大对象无法找到合适空间提前触发垃圾收集

##### 标记复制算法

把内存按容量分成2个区域，当半区内存用完时，把还存活的对象复制到另一个半区，并清除当前半区内存。

（目前商用虚拟机的都采用这种算法去回收新时代，因为新生代98%对象都会在第一轮回收）

缺点：1）在对象存活率高的应用场景下，需要进行大量的复制操作。效率低。

​			2）需要一倍的额外空间

##### 标记整理法

让所有存活对象向内存的一端移动，不直接清除可回收对象，而是当移动完成时直接清除掉存活对象边界后的内存空间。（一般用于老年代回收）

缺点：进行移动的时候需要**暂停全部用户进程**。（Stop the world）问题

权衡方法：在老年代中先采用标记清除算法，当内存碎片化达到一定影响后再采用标记整理法。（CMS收集器）

### 

### HotSpot的GC实现

#### 根节点枚举（OopMaps）

​		根节点（GC Roots）枚举也需要暂停整个用户进程，如果需要去全局引用和执行上下文中去找根节点，在庞大的java应用中，无疑是非常慢的，所以HotSpot采用OopMaps来记录引用。一但类完成加载，hotspot就会在Oopmap中把对象在什么偏移量上是什么类型的数据记录下来，并且记录那些位置是引用。

#### 安全点（SafePoint）

​		**HotSpot不可能为每条指令都生成OopMaps（耗费大量内存**），只有在特定位置才会生成OopMaps，被称为安全点。安全点的选择是根据指令序列的复用，列如方法调用、循环跳转、异常跳转等指令序列复用，具有这些功能的指令才会产生安全点。

​		中断存在问题：多线程情况下，怎么让所有线程都跑到最近的安全点停顿下来。

​		解决办法：1）抢断式中断：系统先中断全部线程，发现有线程不在安全点上，先让这个线程恢复，跑到安全点上再中断，最后一起响应GC。（目前没有虚拟机采用）

​							2）主动式中断：设置一个标志位，各个线程轮询这个标志位。标志位为真是线程在最近的安全点中断，主动挂起。轮询标志和安全点的位置重合，在安全点查看标志位

​							（hotspot采用test指令产生轮询效果，在安全点后面插入一条test指令，当需要中断用户线程的时候把内存页设置为不可读，test指令执行就会产生自陷指令，然后挂起）

存在问题：安全点能解决暂停用户线程问题。但是当用户线程不执行的时候（如没有分配的时间片）就没法执行到安全点，不能响应虚拟机的中断请求。

#### 安全区域

​	安全区域是确保在一段代码中，引用关系不会发生变化。因此在这个区域中，任何时候发生GC都是安全的。

当线程要离开安全区域时，它要检查虚拟机是否完成了根节点的枚举，如果完成了继续执行，没有完成中断等待。



#### 记忆集与卡集

​	为了解决跨代引用的时，新生代扫描GC roots时需要扫描整个老年代。创建了一个新的数据结构-记忆集。

记忆集的实现：

​	1）字节精度：每条记录对应一个字长。表示这个字长包含跨代引用。

​	2）对象精度：每条记录对应一个对象，记录这个对象的指针。表示这个对象包含跨代引用。

​	3）卡精度：每条记录对应一个卡表，表示这个卡表区域包含跨代引用。

​				CARD_Table [this address>>9] = 1; 来标记卡表。可以看出一个卡表包含2^9即512个字节。表示这512个字节空间中包含跨代引用。（一般都采用卡集记录，节省维护空间）

##### 写屏障

​	为了解决实现卡集，在何时写入卡表等问题。Hotspot采用写屏障，应用AOP思想。把维护卡表看作是java赋值操作的一个AOP切面。实现在每次赋值操作时，更新卡表。

#### 并发的可达性分析

​	垃圾收集器判断对象是否死亡采用可达性分析。而且可达性分析必须暂停全部用户线程，对这个操作的优化十分有必要。对于GCroot的收集由于使用了OopMaps，暂停的时间是相对固定且短暂的。根据GCroots继续遍历对象图，会随着java堆的庞大，停顿时间越来越长。所以设计收集器进程与用户进程并发执行。

​	采用三色标记对象： 1）白色：还未被收集器扫描过

​										2）灰色：被收集器扫描过，但至少还存在一个引用还没被收集器扫描（扫了一半）

​										3）黑色：全部引用都被收集器扫描

存在问题：对象消失（还被引用的对象被标记为了白色）

产生原因：1）赋值器插入一条或者多条从黑色对象到白色对象的引用

​					2）赋值器删除了全部从灰色对象到该白色对象的引用

解决办法：1）增量更新：

​								在收集器执行过程中，把赋值器**插入**从**黑色对象到白色**对象的引用记录下来，在这个图扫描结						束时，把黑色对象当作根节点再扫描一次。

​					2）原始快照：

​								在收集器执行过程中，把赋值器**删除**从**灰色对象到白色**对象的引用记录下来，在这个图扫描结						束时，把灰色对象当作根节点再扫描一次。

### 收集器发展历程

#### Serial/Serial Old

新生代采用标记复制算法，老年代采用标记整理算法，执行过程暂停整个用户进程

#### PerNew/Pernew Old

Serial的多线程版，新生代或者老年代执行算法采用多线程。

#### Parallel Scavenge

通过动态调整新生代空间（空间越小会导致收集频率变得频繁）来调整收集器的吞吐量=用户运行代码的时间/（用户运行代码的时间+垃圾收集的时间）

#### Parallel Old

Parallel Scavenge的老年代版本，支持多线程并行收集，采用标记整理算法

#### CMS(Concurrent Mark Sweep)

​	CMS采用标记清除算法，可以实现垃圾收集与用户进程并发执行，主要包括4个阶段：

1）初始标记：只标记与GCroots直接关联的对象

2）并发标记：从初始标记标记的对象遍历整个对象图

3）重新标记：解决并发标记发生的对象消失问题，采用增量更新

4）并发清除：清理需要删除的空间，由于不需要移动，可以并发执行

（初始标记和重新标记仍然需要暂停整个用户进程）

#### Garbage First（G1）

​	G1收集器的回收范围不再是整个java、新生代或者老年代。而是先把java堆分成一个个大小相等的独立区域（Region），每个Region可以是新生代或者老年代，还有专门存放大对象的Humongous Region。G1收集器建立可以可预测的停顿时间模型，根据统计学给每个回收区域一个优先级，优先回收有价值的区域（能回收垃圾多的）来保证暂停时间的稳定。

#### 低延迟垃圾收集器

​	4TB以下的堆容量暂停时间不超过10ms

##### 	Shenandoah收集器

Shenandoah堆的内存分布和G1类似，也是基于Region。但shenandoah不采用记忆集去解决跨代引用，而是采用全局连接矩阵。

1）初始标记：与G1一样只标记与GCroots直接关联的对象，需要暂停整个用户进程

2）并发标记：从初始标记标记的对象遍历整个对象图，并发进行

3）最终标记：与G1一样，处理剩余的SATB扫描，并统计回收价值最高的region，将这些Region统计成一组回收集。会有一小段时间的暂停。

4）并发清除：清理那些没有一个存活对象的Region

5）并发回收：把回收集Region里面还存活的对象复制到还未分配的region里面，需要与用户进程同步进行。采取读屏障和转发指针来解决并发会引发的错误。

6)初始引用更新：并发回收阶段复制到新区域的对象，需要把旧的引用修正到新的引用地址。初始引用更新不进行修改，只是建立一个线程集合点，确保并发回收的线程都已经完成。

7) 并发引用更新：更新引用

8）最终引用更新：修正GCroots中的引用问题，需要短暂停用户进程。

9）并发清理：并发清理回收集中的Region。

​	ZGC

​	并发整理阶段，ZGC采用染色指针，直接把值存在在引用指针的高4位地址中，不需要额外的存储空间。



## 虚拟机执行子系统

### 虚拟机类加载机制

类的生命周期：

加载->|验证->准备->解析|->初始化->使用->卸载

​		   |<-------连接------->|

#### 类加载的过程

##### 加载

1）通过类的全限定名来获取定义此类的二进制字节流。

2）将字节流的存储结构转换为方法区运行时的数据结构。

3）在内存中生成一个java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

##### 验证

确保Class文件字节流符合规范要求。

文件格式验证

元数据验证

字节码验证

符合引用验证

##### 准备

将类中定义的静态变量（static）初始化（赋值为0）。

##### 解析

将常量池中的符号引用替换为直接引用。

**符号引用**：在编译时，java类并不知道所引用的类的实际地址，因此只能使用符号（CONSTANT_Class_info）引用来代替。

**直接引用**：直接引用时可以直接指向目标的指针、相对偏移量或者一个间接定位到目标的句柄。

##### 初始化







## 虚拟机性能监控、故障处理工具

### 基础故障处理工具

#### jps：虚拟机进程状态工具

JVM Process Status Tool

```
jps -l
#可以查询jvm中java进程的ID（与PID一致）
```

#### jstat：统计信息监视工具

jstat JVM Statistics Moinitoring tool

```
jstat -gc 10672 250 20
```

![image-20211116155739129](C:\Users\wy\Desktop\笔记\images\image-20211116155739129.png)

监视10672进程java堆的状况，包括Eden区、2个Survivor区、老年代、永久代的容量（百分比）已用空间、垃圾回收时间等，每250ms监控一次，一共监控20次。

#### jmap：java内存映射工具

#### jhat：堆转储快照分析工具

#### jstack：java堆栈跟踪工具

### 调优案例分析

### 1、大内存硬件上的程序部署

应用背景：案例中服务器使用jdk5，Java堆大小为12G，FullGC一次要12s，服务器的位15w PV/日。由于是在线文档类型的网站，文档序列化后对象很大，直接进入了老年代，新生代回收没有回收这些对象，导致12G内存很快用完，不得不进行FullGC，导致用户进程暂停，用户请求无响应。

解决方案：

1）通过一个单独的JAVA实例来管理大量的JAVA堆内存：

​	最好的解决方案是使用Shennadoah（JDK12）或者ZGC（JDK11）这些明确控制暂停时间的垃圾收集器，但技术还不成熟。这里采用Parallel Scavenge/Parallel Old（JDK6）收集器，控制整个系统的吞吐量，但是要控制Full GC的频率，最好控制在10多个小时以上进行一次Full GC（这样可以在夜间进行Full GC）,由于B/S应用对象都随着web请求的结束而死亡，复合朝生夕灭的原则。这里需要升级JDK版本。

2）通过部署多个java虚拟机，建立逻辑集群来利用硬件资源。

​	hotspot虚拟机只分配1.5G内存时，暂停时间不会让用户有明显感觉。所以在服务器上启动多个虚拟机部署应用，每个引用部署在不同端口，在搭建一个负载均衡器，把请求分配到不同的虚拟机中响应。

​	缺点：1）节点竞争磁盘资源，很容易产生I/O异常

​				2）很难有效利用某些资源池，比如（Mysql连接池），有些虚拟机连接池满了，有些还是空的。

​				3）如果应用使用大量缓存（全局HashMap），每个虚拟机的应用都要单独一份缓存，浪费空间。

### 2、集群间同步导致的内存溢出

​		集群之间频繁的写操作，导致各个节点之间频繁的进行数据同步，并且网络状况不好，大量写操作缓存在内存中，导致内存溢出。非集中式的集群缓存可以运行大量的读操作，读操作的消耗比较少，但是不能允许大量的写操作，会带来很大的网络同步开销。

### 3、堆外内存导致溢出错误

​		windows32位系统中内存限制位2GB，分给java堆的内存位1.6G，直接内存（堆外内存）就只剩0.4G。直接内存不会像新生代和老年代一样，发现内存不足了就通知垃圾回收器回收，只能等FullGC出现顺便帮忙清理掉，或者捕获内存溢出异常，在catch中调用system.gc()。

​		在小应用或者32位应用中，不仅要关注堆的内存异常，还要关注直接内存的异常。

### 4、外部命令导致系统缓慢

​	用户请求调用shell脚本（用Runtime.getRuntime().exec()方法调用），如果这个脚本系统消耗很大，频繁请求导致系统变慢。

### 5、服务器虚拟机奔溃

#### 6、





## CAS（compareAndSet）

```java
volatile int number = 0;
//number前面加了volatile关键词修饰，volatile不保证原子性
public void addPlusPlus(){

	number++;
}

AtomicInterger atomicInterger = new AtomicInteger();
public void addMyAtomic(){
	atomicInterger.getAndIncrement(); //保证原子性
}
```

```java
plubic class CASDemo{

	public static void main(String[] args){
		AtomicInterger atomicInterger = new AtomicInteger(5);
		//compareAndSet(expect, update)  expect期望值，update更新值，当期望值和当前值一样的有时候（没有别的线程修改过）把当前值修改为update值
		atomicInterger.compareAndSet(5, 2021);
		atomicInterger.compareAndSet(5, 2016);	//修改失败返回flase，并且不修改，atomicInterger.get()还是2021
	}
}
```

### UnSafe类

​	**Unsafe类的所有方法都是native修饰的，所有Unsafe类中的方法都直接调用操作系统层资源执行相应任务**。使用操作系统原语进行操作，原语由若干条指令组成，用于完成某个特定的功能，原语的执行都是连续的，不允许打断，所以CAS是一条CPU的原子指令，不会造成数据不一致问题。

​	由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于后门，基于该类可用直接操作特定内存的数据。Unsafe类存在于sun.misc包中，其内部的方法可用像c的指针一样直接操作内存，因此Java中的CAS操作的执行依赖于Unsafe类的方法。

```java
public final int getAndIncrement() { 	
    *//this是当前对象, valueOffset是*        
        return unsafe.getAndAddInt(this, valueOffset, 1);    
}
```

```java
//Unsafe类
//获取内存地址为obj+offset的变量值, 并将该变量值加上delta
public final int getAndAddInt(Object obj, long offset, int delta) {
    int v;
    do {
    	//通过对象和偏移量获取变量的值
    	//由于volatile的修饰, 所有线程看到的v都是一样的
        v= this.getIntVolatile(obj, offset);  //直接读取内存中的v值
		
        /*
       this.compareAndSwapInt(obj, offset, v, v + delta)=false 表示获取v值后有其他线程修改了内存中目标值，重新进入循环，获取最新的v值。
        */
    } while(!this.compareAndSwapInt(obj, offset, v, v + delta));

    return v;
}



```

​	以++操作为例，多线程出错的情况是A线程获取v值为1，B线程也获取1，B线程先修改v值为2，然而A线程后修改++，还是2（正常应该是3）出现错误
​	使用自旋锁保证++的原子性，基础是**unsafe.compareAndSwapInt(obj, offset, v, v + delta)是一个原子操作**，不会被线程中断，**通过循环比较当前进程工作内存读取的v值与共享内存中的v值是否相同**，来确认是否修改。保证不会出现两个线程都读取到v值为2，先后进行++操作导致修改出现错误



### CAS的缺点

​	（1）高并发下，大量进程自旋，cpu占用过高，带来很大开销。

​	（2）CAS只能保证单个变量的原子操作。

​	（3）AtomicInteger类引发ABA问题

​				ABA问题：有A,B两个线程，A线程执行一次2s，B线程执行一次10s，2个线程都要操作共享变量V，开始V变量为0，A,B线程同时开始执行，开始时A，B工作内存中的V都为0，B执行比较快，先把V改成了1，又执行一次把V改回了0，这个时候B线程开始修改V的值，发现共享内存中的V和工作内存中的V值相同（但是已经发生了多次改变），还是对V的值进行了修改。所以A线程出现ABA问题没有保证CAS的原子性。

​				ABA解决办法：给原子引用添加时间戳，



### 原子引用

```
class User{
	String userName;
	int age;
}

pulic class AtomicReferenceDemo{
	public static void main(String[] args){
		User z3 = new User("z3", 22);
		User li4 = new User("li4", 25);
		AtomicReference<User> atomicReference = new AtomicReference<>();
		atomicReference.compareAndSet(z3, li4);
		atomicReference.compareAndSet(z3, li4);
	}
}
```

### ABA解决办法

使用AtomicStampedReference给原子引用添加时间戳，

```java
package test;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * @auther wy
 * @create 2021/11/3 18:54
 */
public class ABADemo {
    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicReference.compareAndSet(100, 2021)+"\t"+atomicReference.get());

        },"t2").start();
        new Thread(()->{
            atomicReference.compareAndSet(100,101);
            System.out.println("成功把100修改为"+atomicReference.get());
            atomicReference.compareAndSet(101,100);
            System.out.println("成功把101修改为"+atomicReference.get());

        }, "t1").start();

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();

            try {
                TimeUnit.SECONDS.sleep(4);
            }catch (InterruptedException e){
                e.printStackTrace();
            }

            boolean result = atomicStampedReference.compareAndSet(100, 2021, stamp, stamp+1);
            System.out.println(Thread.currentThread().getName()+"\t第一次时间戳 "+stamp+"\t修改成功否:"+result+"\t当前值为:"+atomicStampedReference.getReference());

        }, "t3").start();

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();
            try {
                TimeUnit.SECONDS.sleep(2);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"\t第一次时间戳:"+stamp);
            atomicStampedReference.compareAndSet(100, 101, stamp, stamp+1);

            stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t第二次时间戳 "+stamp);
            atomicStampedReference.compareAndSet(101, 100, stamp, stamp+1);
            stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t第三次次时间戳 "+stamp);
        }, "t4").start();
    }
}

```







## JVM名称解释

AOT （Ahead of Time Compilation）： 提前编译

APPCDS（Application Class Data Sharing）允许把加载后的类型信息缓存起来

TLAB Thread local allocation buffer 本地线程分配缓存

CAS compare and set

OOM Out of Memory Error 内存越界错误

AOP （Aspect Oriented Programming） 面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。