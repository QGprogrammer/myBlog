#  实战：OutOfMemoryError异常

***Win10  + Idea +  JDK 8 + JProfiler***



#### Java堆内存溢出异常

-Xms20m: 堆内存最小20m

-Xmx20m：堆内存最大20m

-XX:+HeapDumpOnOutOfMemoryError  内存溢出时存快照

```java
/**
 * VM Args：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 */
public class Test {
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();
        while(true) {
            list.add(new OOMObject());
        }
    }

    static class OOMObject {}
}
```

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid11476.hprof ...
Heap dump file created [29420974 bytes in 0.065 secs]
```



#### 虚拟机栈和本地方法栈溢出

-Xss128k：栈内存最大128k

```java
/**
 * VM Args：-Xss128k
 */
public class Test {
    private int stackLength = 1;
    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        Test test = new Test();
        try {
            test.stackLeak();
        } catch (Throwable e) {
            System.out.println(test.stackLength);
            throw e;
        }
    }

}
```

```java
994
Exception in thread "main" java.lang.StackOverflowError

```

HotSpot***不支持***动态扩展栈容量大小，无论是由于栈帧太大还是虚拟机栈容量太小，当新的栈帧内存无法分配的时候，HotSpot虚拟机抛出的都是StackOverflowError异常。。

如果测试时不限于单线程，通过不断建立线程的方式，在HotSpot上也是可以产生内存溢出异常的：

***OutOfMemoryError***。



#### 方法区和运行时常量池溢出

***JDK 6***中，常量池使用***永久代***存储在方法区中，故而通过设置***-XX:PermSize***和***-XX：MaxPerSize***可以限制永久代的大小。

```java
/**
 * -XX:PermSize=6M -XX:MaxPermSize=6M
 */
public class Test {

    public static void main(String[] args) {
    Set<String> set = new HashSet<String>();
    short i = 0;
    while(true) {
        set.add(String.valueOf(i++).intern());
    }
    }

}
```

在JDK 8中，上述代码运行时，不会受运行参数影响。

```java
public class Test {

    public static void main(String[] args) {
    String str1 = new StringBuilder("计算机").append("软件").toString();
    String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str1.intern() == str1);  //true
        System.out.println(str1.intern() == "计算机软件");  //true
        System.out.println(str1 == "计算机软件");  //true
        System.out.println(str2.intern() == str2); //false
        System.out.println(str2.intern() == "java");  //true
    }

```

***JDK 6中：intern()方法会把首次遇到的字符串实例复制到永久代的字符串常量池中存储，返回的也是永久代里面这个字符串实例的引用。***

***JDK 6以后：intern()方法实现不再拷贝字符串的实例到永久代了，字符串常量池已经移到Java堆中，那只需要在常量池里记录一下首次出现的实例引用即可。***

第一个输出：将StringBuilder创建的对象放到常量池中，由于常量池中没有这个字符，所以将其放堆空间的常量池中，当前对象引用指向常量池中的字符串。

第二个输出：都是在常量池中做比较。

第三个输出：如果没有第一个铺垫，此时返回false，一个常量池引用，一个是普通对象引用。

第四个输出：”java“早就在常量池中了，此时intern（）返回的引用不会等同于当前对象的引用。

第五个输出：都是常量池中取字符串当然相同。