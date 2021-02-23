# &#127800; JVM基础版本 &#127800;

![JVM结构图](https://imgkr2.cn-bj.ufileos.com/754fafb2-e892-400f-9fb4-797554fd4d89.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=%252B0T2NacGaYGKVIKTFRkWT82tu14%253D&Expires=1614133179)

## &#127800; 1. JVM的位置 &#127800;

![](https://static01.imgkr.com/temp/1d7f00c1e5ef4bf19e7e7bef27689193.png)

## &#127800; 2. JVM的体系结构 &#127800;

![](https://imgkr2.cn-bj.ufileos.com/8d568f1e-7f58-46ba-b359-886a725f39e1.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=U7lU9ywnt8P%252BKalU07UMiNZdIy0%253D&Expires=1613877199)

> 本地方法接口：JNI（Java Native Interface）

## &#127800; 3. 类加载器 &#127800;
> 作用：加载Class文件。

![](https://static01.imgkr.com/temp/92ee650862be4ddbbd2a43d1acc3eb59.png)
<details>
<summary>&#127808; 类 与 对象的区别 &#127808;</summary>
  
```java
package JVM;

public class Difference_Class_New_Test {

    // 类(class)是模板, 对象(new ***)是具体的
    public static void main(String[] args){

        // new 的对象, JVM 都会给它分配一个新的内存空间
        Car car1 = new Car();
        Car car2 = new Car();
        Car car3 = new Car();

        System.out.println(car1.hashCode());
        System.out.println(car2.hashCode());
        System.out.println(car3.hashCode());

        // 创建对象的引用地址不一样，但是都是一个 Class(Car) 出来的
        Class<? extends Car> aClass1 = car1.getClass();
        Class<? extends Car> aClass2 = car2.getClass();
        Class<? extends Car> aClass3 = car3.getClass();

        System.out.println(aClass1.hashCode());
        System.out.println(aClass2.hashCode());
        System.out.println(aClass3.hashCode());
    }
}

```
</details>


1. 启动类（根）加载器

<details>
<summary>&#127808; BootstrapClassLoader &#127808;</summary>
  
c++ 编写，加载 java 核心库 java.*,构造 `ExtClassLoader` 和 `AppClassLoader `。由于**引导类加载器涉及到虚拟机本地实现细节**，开发者无法直接获取到启动类加载器的引用，所以**不允许直接通过引用进行操作**

</details>

2. 扩展类加载器

<details>
<summary>&#127808; ExtClassLoader  &#127808;</summary>
  
java 编写，加载扩展库，如 `classpath` 中的 `jre ，javax.*` 或者
`java.ext.dir` 指定位置中的类，开发者可以直接使用标准扩展类加载器。

</details>

3. 应用程序加载器

<details>
<summary>&#127808; AppClassLoader  &#127808;</summary>
  
java编写，加载程序所在的目录，如 `user.dir` 所在的位置的 class

</details>

4. 用户自定义类加载器

<details>
<summary>&#127808; ClassLoader_Test &#127808;</summary>
  
```java
package JVM;

public class ClassLoader_Test {
    // 类(class)是模板, 对象(new ***)是具体的
    public static void main(String[] args) {

        Car car1 = new Car();
        Car car2 = new Car();
        Car car3 = new Car();

        System.out.println(car1.hashCode());
        System.out.println(car2.hashCode());
        System.out.println(car3.hashCode());

        // 创建对象的引用地址不一样，但是都是一个 Class(Car) 出来的
        Class<? extends Car> aClass1 = car1.getClass();
        ClassLoader classLoader = aClass1.getClassLoader();
        System.out.println(classLoader);                            // AppClassLoader_应用程序加载器
        System.out.println(classLoader.getParent());                // ExtClassLoader_扩展类加载器
        System.out.println(classLoader.getParent().getParent());    // BootStarpClassLoader_启动类（根）加载器        java程序获取不到，因为 获取扩展类加载器的父类加载器 --> 根加载器(用C++编写的，无法直接获取)
    }
}

```
</details>

## &#127800; 4. 双亲委派机制 &#127800;

![委派机制的流程图](http://lc-dDwI9S44.cn-n1.lcfile.com/a331a646859ff9464b37.png/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6.png)

### &#127800; 4.1 双亲委派机制的作用

1. 保证数据安全、防止重复加载同一个 `.class`: 
  - 通过委托去向上面问一问，加载过了，就不用再加载一遍。保证数据安全。
2. 保证核心 `.class` 不能被篡改: 
  - 通过委托方式，不会去篡改核心`.class`，即使篡改也不会去加载，即使加载也不会是同一个 `.class` 对象了。不同的加载器加载同一个 `.class` 也不是同一个 `Class` 对象。这样保证了 `Class` 执行安全。

### &#127800; 4.2 如何打破双亲委派？
1. 自定义类加载器，重写 `loadClass` 方法
2. 使用线程上下文类加载器

<details>
<summary>&#127808; 源码分析(了解即可) &#127808;</summary>
  
```java
protected Class<?> loadClass(String name, boolean resolve)
            throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先检查这个classsh是否已经加载过了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // c==null表示没有加载，如果有父类的加载器则让父类加载器加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        //如果父类的加载器为空 则说明递归到bootStrapClassloader了
                        //bootStrapClassloader比较特殊无法通过get获取
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {}
                if (c == null) {
                    //如果bootstrapClassLoader 仍然没有加载过，则递归回来，尝试自己去加载class
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
</details>

[java双亲委派机制及作用](https://www.jianshu.com/p/1e4011617650 "参考作者 秦时的明月夜")

## &#127800; 5. 沙箱安全机制 &#127800;
### &#127800; 5.1 什么是沙箱
- 沙箱是一个**限制程序运行的环境**。
- 沙箱机制就是**将 Java 代码限定在虚拟机(JVM)特定的运行范围中**，并且**严格限制代码对本地系统资源访问**，通过这样的措施来保证对代码的有效隔离，**防止对本地系统造成破坏**。
- 沙箱主要**限制系统资源访问**，那系统资源包括什么？
  - **CPU、内存、文件系统、网络**。不同级别的沙箱对这些资源访问的限制也可以不一样。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

### &#127800; 5.1 组成沙箱的基本组件
- **字节码校验器**（bytecode verifier）：确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。
- **类装载器**（class loader）：其中类装载器在3个方面对Java沙箱起作用
  -  它防止恶意代码去干涉善意的代码；
  -  它守护了被信任的类库边界；
  -  它将代码归入保护域，确定了代码可以进行哪些操作

虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成，每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

类装载器采用的机制是**双亲委派模式**。

<details>
<summary>&#127808; 了解即可 &#127808;</summary>
  

1. 从最内层JVM自带类加载器开始加载，外层恶意同名类得不到加载从而无法使用；
2. 由于严格通过包来区分了访问域，外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。

- 存取控制器（access controller）：存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定，可以由用户指定。
- 安全管理器（security manager）：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。
- 安全软件包（security package）：java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括：
  - 安全提供者
  - 消息摘要
  - 数字签名
  - 加密
  - 鉴别

</details>

## &#127800; 6. Native &#127800;

<details>
<summary>&#127808; Native &#127808;</summary>

```java
package JVM;

public class Native_Test {
    public static void main(String[] args){
        new Thread(() -> {},"Thread1").start();
    }

    /**
     *     start() 源码
     *
     *     public synchronized void start() {
     *
     *         if (threadStatus != 0)
     *             throw new IllegalThreadStateException();
     *
     *         group.add(this);
     *
     *         boolean started = false;
     *         try {
     *             start0(); // 调用了start0()方法
     *             started = true;
     *         } finally {
     *             try {
     *                 if (!started) {
     *                     group.threadStartFailed(this);
     *                 }
     *             } catch (Throwable ignore) {
     *
     *             }
     *         }
     *     }
     */

    // Java在内存区域中专门开辟了一块标记区域——本地方法栈，用来登记native方法，凡是带了native关键字的，会进入到本地方法栈中，调用本地方法接口（JNI），在最终执行的时候，加载本地方法库中的方法通过JNI
    // 凡是带了native关键字的，就说明Java的作用范围达不到了，会去调用底层C语言的库
    private native void start0(); //start0()方法的定义，这个方法会调用底层C
}

```
</details>

![](https://imgkr2.cn-bj.ufileos.com/23250579-f025-4786-8dc7-e5441fa8f9dd.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=vG5NhOj5F0fgscGeP%252F6JKZWNUg4%253D&Expires=1614139111)

- JNI的作用：扩展Java的使用，融合不同的编程语言为Java所用，不过最初是想融合C，C++的，因为Java诞生的时候，C，C++横行，想要立足的话就要有能调用C的程序。现在很少用了。
- 本地方法栈：具体做法是，在Native Method Stack中登记native方法，在执行引擎执行的时候加载Native Libraies【本地库】

## &#127800; 7. PC寄存器 &#127800;

- **程序计数器:** Program Counter Register
  - 每个线程都有一个程序计数器， 是**线程私有的**
  - 就是一个指针，指向方法区中的方法字节码 (用来存储指向像一条指令的地址，也即将要执行的指令代码) 再执行引擎读取下一条指令, 是一个非常小的内存空间，几乎可以忽略不计。
## &#127800; 8. 方法区 &#127800;

- **方法区:** Method Area
- 方法区是**被所有线程共享**，所有字段和方法字节码，以及一些特殊方法，如构造函数，接口代码也在此定义，
  - 即所有**定义的方法的信息**都保存在该区域，此区域属于共享区间；
### 重点！！！
- **静态变量、常量、类信息(构造方法、接口定义)、运行时的常量池存**在**方法区**中，
  - 但是**实例变量**存在**堆内存**中，和**方法区**无关。即 **static、final、Class、常量池**

![MethodsArea](https://static01.imgkr.com/temp/626c43a97fe749ae90df5b3211a233b8.png)

<details>
<summary>&#127808; MethodsArea_Test &#127808;</summary>

```java
package JVM;

public class MethodsArea_Test {
    private int a;
    private String name = "Test";
    public static void main(String[] args){
        MethodsArea_Test test = new MethodsArea_Test();
        test.a = 1;
        test.name = "chen";
    }
}
```
</details>

## &#127800; 9. 栈 &#127800;
- 程序 = 数据结构 + 算法 【持续学习】
- 大多数人程序 = 框架 + 业务逻辑【饭碗】

栈：栈内存，主管程序的运行，生命周期和线程同步；
线程结束，栈内存也就释放了，对于栈来说**不存在垃圾回收问题**，一旦线程结束，栈就Over了

### &#127800; 9.1 栈里面存放什么
- 栈：8大基本类型 + 对象的引用 + 实例的方法
### &#127800; 9.2 栈运行原理

![](https://static01.imgkr.com/temp/8550323662e2465aac361740427bb314.png)
### &#127800; 9.3 栈、堆、方法区的交互关系

![](https://imgkr2.cn-bj.ufileos.com/de7c8134-eb83-4915-a3dc-a5e7d213fb90.png?UCloudPublicKey=TOKEN_8d8b72be-579a-4e83-bfd0-5f6ce1546f13&Signature=al9h2Ay%252FtBNrKInXoGpE7z3RqPc%253D&Expires=1614150994)

## &#127800; 10. 三种JVM &#127800;
- Sun公司 HotSpot Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
- BEA JRockit
- IBM J9VM

## &#127800; 11.堆 &#127800;
- Heap，一个JVM只有一个堆内存，堆内存的大小是可以调节的。
### &#127800; 11.1 堆里面存放什么
- 类加载器读取了类文件后，一般会把什么东西放在堆中？ 
  - 类，方法，常量，变量，保存我们所有引用类型的真实对象
- 堆内存中还要细分为三个区域：
  - 新生区（伊甸园区）young/new
  - 养老区 old
  - 永久区 perm
- 经研究，99%的对象都是临时对象
  - 所以GC垃圾回收，主要在 **伊甸园区** 和 **养老区**

![堆](http://lc-dDwI9S44.cn-n1.lcfile.com/98430cf33d914dd8c46b.png/GC%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.png)


### &#127800; 11.2 出现OOM(OutOfMemoryError) —— 堆溢出

- 假设内存满了，OOM，堆内存不够。则会出现OOM(OutOfMemoryError)

<details>
<summary>&#127808; OutOfHeapSpace_Test &#127808;</summary>
  
```java
package JVM;

import java.util.Random;

public class OutOfHeapSpace_Test {
    public static void main(String[] args){
        String s = "OutOfHeapSpace_Test";
        while (true){
            s += s + new Random().nextInt(888888) + "_" + new Random().nextInt(999999);
            System.out.println(s);
        }
    }
}

```
```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.expandCapacity(AbstractStringBuilder.java:137)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:121)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:647)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at JVM.OutOfHeapSpace_Test.main(OutOfHeapSpace_Test.java:9)
```
</details>
### &#127800; 11.6 VM options参数

## &#127800; 12. 新生区 &#127800;
- 新生区：类诞生和成长的地方，甚至死亡；
  - 伊甸园区，所有对象都是在伊甸园区new出来的
  - 幸存者区（0、1）

![新生区](http://lc-dDwI9S44.cn-n1.lcfile.com/919a2e00f10f5747d1e0.png/%E6%96%B0%E7%94%9F%E5%8C%BA.png)
## &#127800; 13. 老年区 &#127800;

- 新生区没干掉，没杀死的来到了养老区

## &#127800; 14. 永久区 &#127800;

- 这个区域是常驻内存的。用来存放JDK自身携带的 `Class对象，Interface元数据` ，存储的是 `Java` 运行时的一些环境或类信息，这个区域不存在垃圾回收！
  - 当关闭VM虚拟机就会释放这个区域的内存。
- 一个启动类加载了**大量的第三方 jar 包；`Tomcat` 部署了太多的应用**；**大量动态生成的反射类等** 不断的被加载，直到内存满，就会出现OOM。

  - jdk1.6 之前：永久代，常量池是在方法区中；
  - jdk1.7 ：永久代，但是慢慢退化了，去永久代，常量池在堆中
  - jdk1.8 之后：**无永久代**，常量池在元空间
> 但是，元空间：逻辑上存在，物理上不存在

![永久区](http://lc-dDwI9S44.cn-n1.lcfile.com/56dd079e1fffe2bc73d2.png/%E6%B0%B8%E4%B9%85%E5%8C%BA.png)

<details>
<summary>&#127808; Max_Total_Memory &#127808;</summary>
  
```java
package JVM;

public class Max_Total_Memory {
    public static void main(String[] args) {
        // 返回jvm试图使用的最大内存
        long max = Runtime.getRuntime().maxMemory();

        // 返回jvm的初始化内存
        long total = Runtime.getRuntime().totalMemory();

        System.out.println("max = "+max+" 字节 = "+(max/(1024*1024))+" MB");
        System.out.println("total = "+total+" 字节 = "+(total/(1024*1024))+" MB");

        //默认情况下，试图分配的最大内存是电脑内存的1/4，而初始化的内存是1/64
        // -Xms1024m -Xmx1024m -XX:+PrintGCDetails
    }
}
```
```java
max = 2839543808 字节 = 2708 MB
total = 192937984 字节 = 184 MB
```
### 当修改了VM选项后：-Xms1024m -Xmx1024m -XX:+PrintGCDetails
输出结果：
```java
max = 1029177344 字节 = 981 MB
total = 1029177344 字节 = 981 MB
Heap
 PSYoungGen      total 305664K, used 20971K [0x00000000eab00000, 0x0000000100000000, 0x0000000100000000)
  eden space 262144K, 8% used [0x00000000eab00000,0x00000000ebf7afb8,0x00000000fab00000)
  from space 43520K, 0% used [0x00000000fd580000,0x00000000fd580000,0x0000000100000000)
  to   space 43520K, 0% used [0x00000000fab00000,0x00000000fab00000,0x00000000fd580000)
 ParOldGen       total 699392K, used 0K [0x00000000c0000000, 0x00000000eab00000, 0x00000000eab00000)
  object space 699392K, 0% used [0x00000000c0000000,0x00000000c0000000,0x00000000eab00000)
 Metaspace       used 3284K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K
```
</details>

- 新生区：305,664k；养老区：699,392k
- 加在一起：1005,056k，除以1024后 = 981.5MB
- 已经沾满 jvm 试图分配的最大内存，所以说元空间逻辑上存在，物理上不存在。

## &#127800; 15. 堆内存调优 &#127800;
### &#127800; 15.1 出现OOM如何排除
1. 尝试扩大堆内存去查看内存结果
  - `-Xms1024m -Xmx1024m -XX:+PrintGCDetails`
2. 若不行，分析内存，看一下是哪个地方出现了问题（专业工具）
  - 能够看到代码第几行出错：内存快照分析工具，MAT（eclipse），Jprofiler

#### MAT，Jprofiler作用：

- 分析 `Dump` 内存文件，快速定位内存泄漏
- 获得堆中的数据
- 获得大的对象
- ......

### &#127800; 15.2 缩小内存( -> 4M )查看 GC 运行过程
- 代码: `-Xms4m -Xmx4m -XX:+PrintGCDetails`
- **轻重 GC 切换**
```
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1024K)] [ParOldGen: 2339K->1408K(2560K)] 2339K->1408K(3584K), [Metaspace: 3289K->3289K(1056768K)], 0.0076056 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 19K->32K(1024K)] 2452K->2464K(3584K), 0.0002780 secs] 
```

<details>
<summary>&#127808; OutOfHeapSpace_Test &#127808;</summary>
  
```java
import java.util.Random;

// -Xms4m -Xmx4m -XX:+PrintGCDetails
public class OutOfHeapSpace_Test {
    public static void main(String[] args){
        String s = "OutOfHeapSpace_Test";
        while (true){
            s += s + new Random().nextInt(888888) + "_" + new Random().nextInt(999999);
//            System.out.println(s);
        }
    }
}
```
```
[GC (Allocation Failure) [PSYoungGen: 509K->504K(1024K)] 509K->504K(3584K), 0.0010511 secs] [Times: user=0.05 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 1016K->488K(1024K)] 1016K->568K(3584K), 0.0011476 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1000K->488K(1024K)] 1080K->684K(3584K), 0.0009246 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 984K->503K(1024K)] 1180K->796K(3584K), 0.0008305 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 896K->503K(1024K)] 1188K->852K(3584K), 0.0004972 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 896K->472K(1024K)] 1244K->1075K(3584K), 0.0007324 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 734K->351K(1024K)] 1338K->1323K(3584K), 0.0006820 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 616K->159K(1024K)] 2100K->1715K(3584K), 0.0004187 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 424K->287K(1024K)] 1980K->1843K(3584K), 0.0004350 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 553K->287K(1024K)] 2620K->2363K(3584K), 0.0003272 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 287K->0K(1024K)] 2363K->2339K(3584K), 0.0004441 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1024K)] [ParOldGen: 2339K->1408K(2560K)] 2339K->1408K(3584K), [Metaspace: 3289K->3289K(1056768K)], 0.0076056 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 19K->32K(1024K)] 2452K->2464K(3584K), 0.0002780 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 32K->0K(1024K)] [ParOldGen: 2432K->1919K(2560K)] 2464K->1919K(3584K), [Metaspace: 3289K->3289K(1056768K)], 0.0064819 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 511K->0K(1024K)] [ParOldGen: 2431K->1663K(2560K)] 2943K->1663K(3584K), [Metaspace: 3289K->3289K(1056768K)], 0.0031002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1024K)] 1663K->1663K(3584K), 0.0003708 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1024K)] [ParOldGen: 1663K->1643K(2560K)] 1663K->1643K(3584K), [Metaspace: 3289K->3289K(1056768K)], 0.0085453 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 1024K, used 39K [0x00000000ffe80000, 0x0000000100000000, 0x0000000100000000)
  eden space 512K, 7% used [0x00000000ffe80000,0x00000000ffe89f30,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 2560K, used 1643K [0x00000000ffc00000, 0x00000000ffe80000, 0x00000000ffe80000)
  object space 2560K, 64% used [0x00000000ffc00000,0x00000000ffd9af40,0x00000000ffe80000)
 Metaspace       used 3321K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 363K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.expandCapacity(AbstractStringBuilder.java:137)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:121)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:421)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at OutOfHeapSpace_Test.main(OutOfHeapSpace_Test.java:7)
```
</details>
### &#127800; VM options参数

- `-Xms` 设置初始化内存分配大小，默认1/64
- `-Xmx` 设置最大分配内存，默认1/4
- `-XX:+PrintGCDetails` 打印GC垃圾回收信息
- `-XX:+HeapDumpOnOutOfMemoryError` 生成oomDump文件

如: 
- `-Xms1m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError`
- `-Xms1024m -Xmx1024m -XX:+PrintGCDetails`

## &#127800; 16 GC &#127800;
### &#127800; 16.1 GC的作用区域

![GC的作用区域](http://lc-dDwI9S44.cn-n1.lcfile.com/dc56ef4e03b7815f5d25.png/GC%E7%9A%84%E4%BD%9C%E7%94%A8%E5%8C%BA%E5%9F%9F.png)

- JVM在进行GC时，并不是对这三个区域统一回收，大部分回收都是新生代
  - 新生代
  - 幸存区（form to）【会交换的，不是一成不变的】
  - 老年区
- GC两种类型：轻GC（普通的GC），重GC（fullGC）
  
### &#127800; 16.2 GC相关题目

- JVM的内存模型和分区~详细到每个区放什么？
- 堆里面的分区有哪些？Eden，from，to，old，说说他们的特点~
- GC的算法有哪些？标记清除法，标记整理/压缩法，复制算法，引用计数法，怎么用的？
- 轻GC和重GC分别在什么时候发生？

### &#127800; 16.3 GC算法
#### 16.3.1 引用计数法(用的少)
- 哪个对象的引用数为0，就会回收哪个对象

![引用计数法](http://lc-dDwI9S44.cn-n1.lcfile.com/cc61f3ae1a3d3cd83d06.png/%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0%E6%B3%95.png)

#### 16.3.2 复制算法

- 一般新生代（伊甸园区、幸存区）会使用复制算法，**生成新的 to 区**
  - 好处：没有内存的碎片
  - 坏处：浪费了内存空间，多了一半空间永远是空的 (to)
- **复制算法最佳使用场景：对象存活度较低的时候，也就是新生区**

![复制算法](http://lc-dDwI9S44.cn-n1.lcfile.com/49176d146c1cd907e6d1.png/%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95.png)


![动态复制算法](http://lc-dDwI9S44.cn-n1.lcfile.com/87a172c84b8dc6b26bb0.png/%E5%8A%A8%E6%80%81%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95.png)

#### 16.3.3 标记清除

- 缺点：两次扫描，严重浪费时间，会产生内存碎片
- 优点：不需要额外的空间

![标记清除](http://lc-dDwI9S44.cn-n1.lcfile.com/b3e100db10f506ce0401.png/%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4.png)

#### 16.3.4 标记压缩
- 对于标记清除的再压缩,但是又多了一个移动成本

![](http://lc-dDwI9S44.cn-n1.lcfile.com/3f1d2df515887d461136.png/%E6%A0%87%E8%AE%B0%E5%8E%8B%E7%BC%A9.png)

#### 16.3.5 标记清除压缩
- 先标记清除一次，然后再压缩

### &#127800; 16.4 总结

#### 16.4.1 内存效率：
  - 复制算法 > 标记清除算法 > 标记压缩算法（时间复杂度）

#### 16.4.2 内存整齐度：
  - 复制算法 = 标记压缩算法 > 标记清除算法

#### 16.4.3 内存利用率：
  - 标记压缩算法 = 标记清除算法 > 复制算法

#### 16.4.4 思考：难道没有最优算法吗？
  - 没有，没有最好的算法，只有最合适的——>GC：分代收集算法

#### 16.4.4 年轻代、老年代对比
- 年轻代：
  - 存活率低
  - **适合** 复制算法
- 老年代：
  - 区域大，存活率高
  - **适合** 标记清除（内存碎片不是太多） + 标记压缩混合实现



## &#127800; 17 JMM &#127800;
### &#127800; 17.1 什么是JMM？
- java 内存模型 : Java Memory Model

### &#127800; 17.1 作用？

- 作用：**缓存一致性协议**，用于定义数据读写的规则。
- JMM 定义了**线程工作内存**和**主内存**之间的抽象关系，
  - 线程之间的**共享内存**存储在**主内存**(Main Memory)中
  - 每个线程都有一个**私有的本地内存**（Local Memory）

![JMM](http://lc-dDwI9S44.cn-n1.lcfile.com/9a86ddc56caa0a4e0dac.png/JMM.png)

## &#127800; 18 总结 &#127800; 

- 《深入理解JVM》
- 多看面试题、boke

[遇见狂神说](https://www.bilibili.com/video/BV1iJ411d7jS?p=14&spm_id_from=pageDriver "狂神B站")

[开心点](https://gitee.com/dracoFei/study-diary/blob/master/JVM/%E7%AC%AC%E4%B8%80%E7%89%88.md "gitee博客")