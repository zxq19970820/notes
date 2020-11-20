# 请问s1`==`s3是true还是false，s1`==`s4是false还是true?s1`==`s5呢？

```java
   String s1 = "abc";
   String s2 = "a";
   String s3 = s2 + "bc";
   String s4 = "a" + "bc";
   String s5 = s3.intern();
```

s1`==`s3返回false,

s1`==`s4返回true,

s1`==`s5返回true.

“abc"这个字符串常量值会直接方法字符串常量池中,s1是对其的引用.由于s2是个变量,编译器在编译期间无法确定该变量后续会不会改,因此无法直接将s3的值在编译器计算出来,因此s3是堆中"abc"的引用.因此s1!=s3.

对于s4而言,其赋值号右边是常量表达式,因此可以在编译阶段直接被优化为"abc”,由于"abc"已经在字符串常量池中存在,因此s4是对其的引用,此时也就意味s1和s4引用了常量池中的同一个"abc".所以s1`==`s4.

String中的`intern()`会首先从字符串常量池中检索是否已经存在字面值为"abc"的对象,如果不存在则先将其添加到字符串常量池中,否则直接返回已存在字符串常量的引用.此处由于"abc"已经存在字符串常量池中了,因此s5和s1引用的是同一个字符串常量.

# 以下代码中,s5`==`s2返回值是什么?

```java
String s1="ab";
String s2="a"+"b";
String s3="a";
String s4="b";
String s5=s3+s4;
```

返回false.在编译过程中,编译器会将s2直接优化为"ab",将其放置在常量池当中;而s5则是被创建在堆区,相当于s5=new String(“ab”);





# ==intern()==

**方法返回字符串对象的规范化表示形式**。

它遵循以下规则：对于任意两个字符串 s 和 t，当且仅当 s.equals(t) 为 true 时，s.intern() == t.intern() 才为 true。

### 语法

```
public String intern()
```

### 参数

- 无

### 返回值

一个==字符串==，内容与此字符串相同，但一定取自具有唯一字符串的池。

### 实例

```
public class Test {
    public static void main(String args[]) {
        String Str1 = new String("www.runoob.com");
        String Str2 = new String("WWW.RUNOOB.COM");

        System.out.print("规范表示:" );
        System.out.println(Str1.intern());

        System.out.print("规范表示:" );
        System.out.println(Str2.intern());
    }
}
```

以上程序执行结果为：

```
规范表示:www.runoob.com
规范表示:WWW.RUNOOB.COM
```



尽管在输出中调用intern方法并没有什么效果，但是实际上后台这个方法会做一系列的动作和操作。在调用”ab”.intern()方法的时候会返回”ab”，但是这个方法会首先检查字符串池中是否有”ab”这个字符串，如果存在则返回这个字符串的引用，否则就将这个字符串添加到字符串池中，然会返回这个字符串的引用。

可以看下面一个范例：

```
String str1 = "a";
String str2 = "b";
String str3 = "ab";
String str4 = str1 + str2;
String str5 = new String("ab");
 
System.out.println(str5.equals(str3));
System.out.println(str5 == str3);
System.out.println(str5.intern() == str3);
System.out.println(str5.intern() == str4);
```

得到的结果：

```
true
false
true
false
```

为什么会得到这样的一个结果呢？我们一步一步的分析。

-  第一、str5.equals(str3)这个结果为true，不用太多的解释，因为字符串的值的内容相同。
-  第二、str5 == str3对比的是引用的地址是否相同，由于str5采用new String方式定义的，所以地址引用一定不相等。所以结果为false。
-  第三、当str5调用intern的时候，会检查字符串池中是否含有该字符串。由于之前定义的str3已经进入字符串池中，所以会得到相同的引用。
-  第四，当str4 = str1 + str2后，str4的值也为”ab”，但是为什么这个结果会是false呢？先看下面代码：

```
String a = new String("ab");
String b = new String("ab");
String c = "ab";
String d = "a" + "b";
String e = "b";
String f = "a" + e;

System.out.println(b.intern() == a);
System.out.println(b.intern() == c);
System.out.println(b.intern() == d);
System.out.println(b.intern() == f);
System.out.println(b.intern() == a.intern());
```

运行结果：

```
false
true
true
false
true
```

由运行结果可以看出来，b.intern() == a和b.intern() == c可知，==采用new 创建的字符串对象不进入字符串池==，并且通过b.intern() == d和b.intern() == f可知，字符串相加的时候，都是静态字符串的结果会添加到字符串池，==如果其中含有变量（如f中的e）则不会进入字符串池中==。但是字符串一旦进入字符串池中，就会先查找池中有无此对象。如果有此对象，则让对象引用指向此对象。如果无此对象，则先创建此对象，再让对象引用指向此对象。

当研究到这个地方的时候，突然想起来经常遇到的一个比较经典的Java问题，就是对比equal和的区别，当时记得老师只是说“==”判断的是“地址”，但是并没说清楚什么时候会有地址相等的情况。现在看来，在定义变量的时候赋值，如果赋值的是静态的字符串，就会执行进入字符串池的操作，如果池中含有该字符串，则返回引用。

执行下面的代码：

```
String a = "abc";
String b = "abc";
String c = "a" + "b" + "c";
String d = "a" + "bc";
String e = "ab" + "c";
        
System.out.println(a == b);
System.out.println(a == c);
System.out.println(a == d);
System.out.println(a == e);
System.out.println(c == d);
System.out.println(c == e);
```

运行的结果：

```
true
true
true
true
true
true
```



# 浅谈String.intern()方法

### 1.String类型“==”比较样例代码如下：

```java
public class StringTest {

	public static void main(String[] args) {

		String str1 = "todo";
        String str2 = "todo";
        String str3 = "to";
        String str4 = "do";
        String str5 = str3 + str4;
        String str6 = new String(str1);


        System.out.println("------普通String测试结果------");
        System.out.print("str1 == str2 ? ");
        System.out.println( str1 == str2);
        System.out.print("str1 == str5 ? ");
        System.out.println(str1 == str5);
        System.out.print("str1 == str6 ? ");
        System.out.print(str1 == str6);
        System.out.println();


        System.out.println("---------intern测试结果---------");
        System.out.print("str1.intern() == str2.intern() ? ");
        System.out.println(str1.intern() == str2.intern());
        System.out.print("str1.intern() == str5.intern() ? ");
        System.out.println(str1.intern() == str5.intern());
        System.out.print("str1.intern() == str6.intern() ? ");
        System.out.println(str1.intern() == str6.intern());
        System.out.print("str1 == str6.intern() ? ");
        System.out.println(str1 == str6.intern());
	}
}
```

   代码运行结果如下所示：

```yaml
------普通String测试结果------
str1 == str2 ? true
str1 == str5 ? false
str1 == str6 ? false
---------intern测试结果---------
str1.intern() == str2.intern() ? true
str1.intern() == str5.intern() ? true
str1.intern() == str6.intern() ? true
str1 == str6.intern() ? true
```

   普通String代码结果分析：Java语言会使用常量池保存那些在编译期就已确定的已编译的class文件中的一份数据。主要有类、接口、方法中的常量，以及一些以文本形式出现的符号引用，如类和接口的全限定名、字段的名称和描述符、方法和名称和描述符等。因此在编译完Intern类后，生成的class文件中会在常量池中保存“todo”、“to”和“do”三个String常量。变量str1和str2均保存的是常量池中“todo”的引用，所以str1==str2成立；在执行 str5 = str3 + str4这句时，JVM会先创建一个StringBuilder对象，通过StringBuilder.append()方法将str3与str4的值拼接，然后通过StringBuilder.toString()返回一个堆中的String对象的引用，赋值给str5，因此str1和str5指向的不是同一个String对象，str1 == str5不成立；String str6 = new String(str1)一句显式创建了一个新的String对象，因此str1 == str6不成立便是显而易见的事了。

### 2.String.intern()使用原理

   String.intern()是一个Native方法，底层调用C++的 StringTable::intern方法实现。当通过语句str.intern()调用intern()方法后，JVM 就会在当前类的常量池中查找是否存在与str等值的String，若存在则直接返回常量池中相应Strnig的引用；若不存在，则会在常量池中创建一个等值的String，然后返回这个String在常量池中的引用。因此，只要是等值的String对象，使用intern()方法返回的都是常量池中同一个String引用，所以，这些等值的String对象通过intern()后使用==是可以匹配的。由此就可以理解上面代码中------intern------部分的结果了。因为str1、str5和str6是三个等值的String，所以通过intern()方法，他们均会指向常量池中的同一个String引用，因此str1.intern() == str5.intern() == str6.intern()均为true。

### 3.String.intern() in JDK6

   Jdk6中常量池位于PermGen（永久代）中，PermGen是一块主要用于存放已加载的类信息和字符串池的大小固定的区域。执行intern()方法时，若常量池中不存在等值的字符串，JVM就会在常量池中创建一个等值的字符串，然后返回该字符串的引用。除此以外，JVM 会自动在常量池中保存一份之前已使用过的字符串集合。Jdk6中使用intern()方法的主要问题就在于常量池被保存在PermGen中：首先，PermGen是一块大小固定的区域，一般不同的平台PermGen的默认大小也不相同，大致在32M到96M之间。所以不能对不受控制的运行时字符串（如用户输入信息等）使用intern()方法，否则很有可能会引发PermGen内存溢出；其次String对象保存在Java堆区，Java堆区与PermGen是物理隔离的，因此如果对多个不等值的字符串对象执行intern操作，则会导致内存中存在许多重复的字符串，会造成性能损失。

### 4.String.intern() in JDK7

   Jdk7将常量池从PermGen区移到了Java堆区，执行intern操作时，如果常量池已经存在该字符串，则直接返回字符串引用，否则复制该字符串对象的引用到常量池中并返回。堆区的大小一般不受限，所以将常量池从PremGen区移到堆区使得常量池的使用不再受限于固定大小。除此之外，位于堆区的常量池中的对象可以被垃圾回收。当常量池中的字符串不再存在指向它的引用时，JVM就会回收该字符串。可以使用 -XX:StringTableSize 虚拟机参数设置字符串池的map大小。字符串池内部实现为一个HashMap，所以当能够确定程序中需要intern的字符串数目时，可以将该map的size设置为所需数目*2（减少hash冲突），这样就可以使得String.intern()每次都只需要常量时间和相当小的内存就能够将一个String存入字符串池中。

### 5.intern()适用场景

   Jdk6中常量池位于PermGen区，大小受限，所以不建议适用intern()方法，当需要字符串池时，需要自己使用HashMap实现。Jdk7、8中，常量池由PermGen区移到了堆区，还可以通过-XX:StringTableSize参数设置StringTable的大小，常量池的使用不再受限，由此可以重新考虑使用intern()方法。intern(）方法优点：执行速度非常快，直接使用==进行比较要比使用equals(）方法快很多；内存占用少。虽然intern()方法的优点看上去很诱人，但若不是在恰当的场合中使用该方法的话，便非但不能获得如此好处，反而还可能会有性能损失。下面程序对比了使用intern()方法和未使用intern()方法存储100万个String时的性能，从输出结果可以看出，若是单纯使用intern()方法进行数据存储的话，程序运行时间要远高于未使用intern()方法时：

```java
public class InternTest {

    public static void main(String[] args) {
        print("noIntern: " + noIntern());
        print("intern: " + intern());
    }

    private static long noIntern(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {

            int j = i % 100;
            String str = String.valueOf(j);
        }
        return System.currentTimeMillis() - start;
    }

    private static long intern(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            int j = i % 100;
            String str = String.valueOf(j).intern();
        }
        return System.currentTimeMillis() - start;
    }
}
```

   程序运行结果：

```groovy
noIntern: 48    // 未使用intern方法时，存储100万个String所需时间
intern: 99      // 使用intern方法时，存储100万个String所需时间
```

   由于intern()操作每次都需要与常量池中的数据进行比较以查看常量池中是否存在等值数据，同时JVM需要确保常量池中的数据的唯一性，这就涉及到加锁机制，这些操作都是有需要占用CPU时间的，所以如果进行intern操作的是大量不会被重复利用的String的话，则有点得不偿失。由此可见，String.intern()主要 **适用于只有有限值，并且这些有限值会被重复利用的场景**，如数据库表中的列名、人的姓氏、编码类型等。



### 6.总结：

   String.intern()方法是一种手动将字符串加入常量池中的方法，

原理如下：如果在常量池中存在与调用intern()方法的字符串等值的字符串，就直接返回常量池中相应字符串的引用，否则在常量池中复制一份该字符串，并将其引用返回（Jdk7中会直接在常量池中保存当前字符串的引用）；Jdk6 中常量池位于PremGen区，大小受限，不建议使用String.intern()方法，不过Jdk7 将常量池移到了Java堆区，大小可控，可以重新考虑使用String.intern()方法，但是由对比测试可知，使用该方法的耗时不容忽视，所以需要慎重考虑该方法的使用；String.intern()方法主要适用于程序中需要保存有限个会被反复使用的值的场景，这样可以减少内存消耗，同时在进行比较操作时减少时耗，提高程序性能。





# 多线程

![image-20201004212810269](java%E5%9F%BA%E7%A1%80/image-20201004212810269.png)

## 说说进程,线程之间的区别?

简而言之,进程是程序运行和资源分配的基本单位,一个程序至少有一个进程,一个进程至少有一个线程.进程在执行过程中拥有独立的内存单元,而多个线程共享内存资源,减少切换次数,从而效率更高.线程是进程的一个实体,是cpu调度和分派的基本单位,是比程序更小的能独立运行的基本单位.同一进程中的多个线程之间可以并发执行.在Linux中,进程也称为Task.

## 守护线程是什么?它和非守护线程有什么区别

程序运行完毕,jvm会等待非守护线程完成后关闭,但是jvm不会等待守护线程.守护线程最典型的例子就是GC线程.

## 什么是多线程上下文切换

多线程的上下文切换是指CPU控制权由一个已经正在运行的线程切换到另外一个就绪并等待获取CPU执行权的线程的过程.

## 创建两种线程的方式?他们有什么区别?

通过实现java.lang.Runnable或者通过扩展java.lang.Thread类.相比扩展Thread,实现Runnable接口可能更优.原因有二:

1. Java不支持多继承.因此扩展Thread类就代表这个子类不能扩展其他类.而实现Runnable接口的类还可能扩展另一个类.
2. 类可能只要求可执行即可,因此继承整个Thread类的开销过大.

## Callable和Runnable的区别是什么?

两者都能用来编写多线程,但实现Callable接口的任务线程能返回执行结果,而实现Runnable接口的任务线程不能返回结果.Callable通常需要和Future/FutureTask结合使用,用于获取异步计算结果.

## Thread类中的start()和run()方法有什么区别?

在`start(`)方法中最终要的是调用了Native方法`start0()`用来启动新创建的线程线程启动后会自动调用`run()`方法.如果我们直接调用其run()方法就和我们调用其他方法一样,不会在新的线程中执行.

\##怎么检测一个线程是否持有对象锁

Thread类提供了一个Native方法`holdsLock(Object obj)`方法用于检测是否持有某个对象锁:当且仅当对象obj的锁被某线程持有的时候才会返回true.

```java
 public static native boolean holdsLock(Object obj);
```

## 线程阻塞有哪些原因?

阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。

| 方法                  | 说明                                                         |
| :-------------------- | :----------------------------------------------------------- |
| ==sleep==()           | sleep() 允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止 |
| suspend() 和 resume() | 两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。 |
| yield()               | yield() 使当前线程放弃当前已经分得的CPU 时间，但不使当前线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程 |
| wait() 和 notify()    | 两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许 指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用. |



## wait(),notify()和suspend(),resume()之间的区别

### *区别的核心在于:

### *1、suspend()在引起当前所在线程阻塞后，不会释放线程占用的锁（如果占用了的话）*

###  2、wait() 引起当前所在线程阻塞后，会释放占用的锁，并且必须 wait对象的锁必须被当前线程持有 3、wait与notify 需要 synchronized 锁来包裹.

初看起来它们与 suspend() 和 resume() 方法对没有什么分别，但是事实上它们是截然不同的。区别的核心在于，前面叙述的所有方法，阻塞时都不会释放占用的锁（如果占用了的话），而这一对方法则相反。上述的核心区别导致了一系列的细节上的区别。

首先，前面叙述的所有方法都隶属于 Thread 类，但是这一对却直接隶属于 Object 类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致从调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

其次，前面叙述的所有方法都可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。

wait() 和 notify() 方法的上述特性决定了它们经常和synchronized关键字一起使用，将它们和操作系统进程间通信机制作一个比较就会发现它们的相似性：synchronized方法或块提供了类似于操作系统原语的功能，它们的执行不会受到多线程机制的干扰，而这一对方法则相当于 block 和wakeup 原语（这一对方法均声明为 synchronized）。它们的结合使得我们可以实现操作系统上一系列精妙的进程间通信的算法（如信号量算法），并用于解决各种复杂的线程间通信问题。

关于 wait() 和 notify() 方法最后再说明两点：
第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。

第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。

谈到阻塞，就不能不谈一谈死锁，略一分析就能发现，suspend() 方法和不指定超时期限的 wait() 方法的调用都可能产生死锁。遗憾的是，Java 并不在语言级别上支持死锁的避免，我们在编程中必须小心地避免死锁。

以上我们对 Java 中实现线程阻塞的各种方法作了一番分析，我们重点分析了 wait() 和 notify() 方法，因为它们的功能最强大，使用也最灵活，但是这也导致了它们的效率较低，较容易出错。实际使用中我们应该灵活使用各种方法，以便更好地达到我们的目的。

## 产生死锁的条件

1.互斥条件：一个资源每次只能被一个进程使用。
2.请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3.不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
4.循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

## 为什么wait()方法和notify()/notifyAll()方法要在同步块中被调用

这是JDK强制的，wait()方法和notify()/notifyAll()方法在调用前都必须先获得对象的锁

## wait()方法和notify()/notifyAll()方法在放弃对象监视器时有什么区别

wait()方法和notify()/notifyAll()方法在放弃对象监视器的时候的区别在于：wait()方法立即释放对象监视器，notify()/notifyAll()方法则会等待线程剩余代码执行完毕才会放弃对象监视器。

## wait()与sleep()的区别

关于这两者已经在上面进行详细的说明,这里就做个概括好了:

- sleep()来自Thread类，和wait()来自Object类.调用sleep()方法的过程中，线程不会释放对象锁。而 调用 wait 方法线程会释放对象锁
- sleep()睡眠后不出让系统资源，wait让其他线程可以占用CPU
- sleep(milliseconds)需要指定一个睡眠时间，时间一到会自动唤醒.而wait()需要配合notify()或者notifyAll()使用

## 为什么wait,nofity和nofityAll这些方法不放在Thread类当中

一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。

\##怎么唤醒一个阻塞的线程
如果线程是因为调用了wait()、sleep()或者join()方法而导致的阻塞，可以中断线程，并且通过抛出InterruptedException来唤醒它；如果线程遇到了IO阻塞，无能为力，因为IO是操作系统实现的，Java代码并没有办法直接接触到操作系统。

\##什么是多线程的上下文切换
多线程的上下文切换是指CPU控制权由一个已经正在运行的线程切换到另外一个就绪并等待获取CPU执行权的线程的过程。

## synchronized和ReentrantLock的区别

synchronized是和if、else、for、while一样的关键字，ReentrantLock是类，这是二者的本质区别。既然ReentrantLock是类，那么它就提供了比synchronized更多更灵活的特性，可以被继承、可以有方法、可以有各种各样的类变量，ReentrantLock比synchronized的扩展性体现在几点上：
（1）ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁
（2）ReentrantLock可以获取各种锁的信息
（3）ReentrantLock可以灵活地实现多路通知
另外，二者的锁机制其实也是不一样的:ReentrantLock底层调用的是Unsafe的park方法加锁，synchronized操作的应该是对象头中mark word.

## FutureTask是什么

这个其实前面有提到过，FutureTask表示一个异步运算的任务。FutureTask里面可以传入一个Callable的具体实现类，可以对这个异步运算的任务的结果进行等待获取、判断是否已经完成、取消任务等操作。当然，由于FutureTask也是Runnable接口的实现类，所以FutureTask也可以放入线程池中。

## 一个线程如果出现了运行时异常怎么办?

如果这个异常没有被捕获的话，这个线程就停止执行了。另外重要的一点是：如果这个线程持有某个某个对象的监视器，那么这个对象监视器会被立即释放

## Java虚拟机对synchronized的优化

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级.

1. 偏向锁: 偏向锁是JDK 1.6之后加入的新锁，它是一种针对加锁操作的优化手段，经过研究发现，在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁(会涉及到一些CAS操作,耗时)的代价而引入偏向锁。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁
2. 轻量级锁:倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(1.6之后加入的)，此时Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁
3. 自旋锁: 轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

除此之外,锁消除也是一项非常重要的优化手段.Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间.

## 线程中断相关方法

当一个线程处于被阻塞状态或者试图执行一个阻塞操作时，使用`Thread.interrupt()`方式中断该线程，此时将会抛出一个InterruptedException的异常，同时中断状态将会被复位(由中断状态改为非中断状态).在Java中提供了以下三个与中断相关的方法:

```java
//中断线程（实例方法）



public void Thread.interrupt();



 



//判断线程是否被中断（实例方法）



public boolean Thread.isInterrupted();



 



//判断是否被中断并清除当前中断状态



public static boolean Thread.interrupted();



 
```

## 如何在两个线程间共享数据

通过在线程之间共享对象就可以了，然后通过wait/notify/notifyAll、await/signal/signalAll进行唤起和等待，比方说阻塞队列BlockingQueue就是为线程之间共享数据而设计的

## 如何正确的使用wait()?使用if还是while?

wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：

```java
 synchronized (obj) {



    while (condition does not hold)



      obj.wait(); // (Releases lock, and reacquires on wakeup)



      ... // Perform action appropriate to condition



 }
```

## 什么是线程局部变量ThreadLocal

线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。

## ThreadLoal的作用是什么?

简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal.ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了.

## 生产者消费者模型的作用是什么?

（1）通过平衡生产者的生产能力和消费者的消费能力来提升整个系统的运行效率，这是生产者消费者模型最重要的作用
（2）解耦，这是生产者消费者模型附带的作用，解耦意味着生产者和消费者之间的联系少，联系越少越可以独自发展而不需要收到相互的制约

## 写一个生产者-消费者队列

可以通过阻塞队列实现,也可以通过wait-notify来实现.

### 使用阻塞队列来实现

```java
//消费者
public class Producer implements Runnable{
    private final BlockingQueue<Integer> queue;
    public Producer(BlockingQueue q){
        this.queue=q;
    }
    
    @Override
    public void run() {
        try {
            while (true){
                Thread.sleep(1000);//模拟耗时
                queue.put(produce());
            }
        }catch (InterruptedException e){
        }
    }

    private int produce() {

        int n=new Random().nextInt(10000);
        System.out.println("Thread:" + Thread.currentThread().getId() + " produce:" + n);
        return n;
    }
}

//消费者
public class Consumer implements Runnable {

    private final BlockingQueue<Integer> queue;

    public Consumer(BlockingQueue q){
        this.queue=q;
    }
    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(2000);//模拟耗时
                consume(queue.take());
            }catch (InterruptedException e){
            }
        }
    }

    private void consume(Integer n) {
        System.out.println("Thread:" + Thread.currentThread().getId() + " consume:" + n);
    }
}

//测试
public class Main {

    public static void main(String[] args) {
        BlockingQueue<Integer> queue=new ArrayBlockingQueue<Integer>(100);

        Producer p=new Producer(queue);
        
        Consumer c1=new Consumer(queue);
        Consumer c2=new Consumer(queue);

        new Thread(p).start();
        new Thread(c1).start();
        new Thread(c2).start();
    }
}
```

### 使用wait-notify来实现

该种方式应该最经典,这里就不做说明了

\##如果你提交任务时,线程池队列已满,这时会发生什么

如果使用的LinkedBlockingQueue,也就是无界队列的话,继续添加任务到阻塞队列中等待执行.因为LinkedBlockingQueue可以近乎认为是一个无穷大的队列,可以无限存放任务;如果你使用的是有界队列比方说ArrayBlockingQueue的话,任务首先会被添加到ArrayBlockingQueue中,ArrayBlockingQueue满了,则会使用拒绝策略RejectedExecutionHandler处理满了的任务,默认是AbortPolicy.



# 死锁面试题（什么是死锁，产生死锁的原因及必要条件）

![img](java%E5%9F%BA%E7%A1%80/original.png)

[AddoilDan](https://me.csdn.net/hd12370) 2018-09-22 17:52:59 ![img](java%E5%9F%BA%E7%A1%80/articleReadEyes.png) 101065 ![img](java%E5%9F%BA%E7%A1%80/tobarCollect.png) 收藏 407

分类专栏： [Java](https://blog.csdn.net/hd12370/category_7807994.html) 文章标签： [死锁](https://so.csdn.net/so/search/s.do?q=死锁&t=blog&o=vip&s=&l=&f=&viparticle=)

版权

# 什么是死锁？

所谓死锁，是指多个进程在运行过程中因争夺资源而造成的一种僵局，当进程处于这种僵持状态时，若无外力作用，它们都将无法再向前推进。 因此我们举个例子来描述，如果此时有一个线程A，按照先锁a再获得锁b的的顺序获得锁，而在此同时又有另外一个线程B，按照先锁b再锁a的顺序获得锁。如下图所示：

![img](java%E5%9F%BA%E7%A1%80/20180922173936964)

# 产生死锁的原因？

可归结为如下两点：

a. 竞争资源

- 系统中的资源可以分为两类：

1. 可剥夺资源，是指某进程在获得这类资源后，该资源可以再被其他进程或系统剥夺，CPU和主存均属于可剥夺性资源；
2. 另一类资源是不可剥夺资源，当系统把这类资源分配给某进程后，再不能强行收回，只能在进程用完后自行释放，如磁带机、打印机等。

- 产生死锁中的竞争资源之一指的是竞争不可剥夺资源（例如：系统中只有一台打印机，可供进程P1使用，假定P1已占用了打印机，若P2继续要求打印机打印将阻塞）
- 产生死锁中的竞争资源另外一种资源指的是竞争临时资源（临时资源包括硬件中断、信号、消息、缓冲区内的消息等），通常消息通信顺序进行不当，则会产生死锁

b. 进程间推进顺序非法

- 若P1保持了资源R1,P2保持了资源R2，系统处于不安全状态，因为这两个进程再向前推进，便可能发生死锁
- 例如，当P1运行到P1：Request（R2）时，将因R2已被P2占用而阻塞；当P2运行到P2：Request（R1）时，也将因R1已被P1占用而阻塞，于是发生进程死锁

# 死锁产生的4个必要条件？

产生死锁的必要条件：

1. 互斥条件：进程要求对所分配的资源进行排它性控制，即在一段时间内某资源仅为一进程所占用。
2. 请求和保持条件：当进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：进程已获得的资源在未使用完之前，不能剥夺，只能在使用完时由自己释放。
4. 环路等待条件：在发生死锁时，必然存在一个进程--资源的环形链。

# 解决死锁的基本方法

## 预防死锁：

- 资源一次性分配：一次性分配所有资源，这样就不会再有请求了：（破坏请求条件）
- 只要有一个资源得不到分配，也不给这个进程分配其他的资源：（破坏请保持条件）
- 可剥夺资源：即当某进程获得了部分资源，但得不到其它资源，则释放已占有的资源（破坏不可剥夺条件）
- 资源有序分配法：系统给每类资源赋予一个编号，每一个进程按编号递增的顺序请求资源，释放则相反（破坏环路等待条件）

1、以确定的顺序获得锁

如果必须获取多个锁，那么在设计的时候需要充分考虑不同线程之前获得锁的顺序。按照上面的例子，两个线程获得锁的时序图如下：

![img](java%E5%9F%BA%E7%A1%80/20180922174807514)

 如果此时把获得锁的时序改成：

![img](java%E5%9F%BA%E7%A1%80/20180922174829303)

 那么死锁就永远不会发生。 针对两个特定的锁，开发者可以尝试按照锁对象的hashCode值大小的顺序，分别获得两个锁，这样锁总是会以特定的顺序获得锁，那么死锁也不会发生。问题变得更加复杂一些，如果此时有多个线程，都在竞争不同的锁，简单按照锁对象的hashCode进行排序（单纯按照hashCode顺序排序会出现“环路等待”），可能就无法满足要求了，这个时候开发者可以使用银行家算法，所有的锁都按照特定的顺序获取，同样可以防止死锁的发生，该算法在这里就不再赘述了，有兴趣的可以自行了解一下。

2、超时放弃

当使用synchronized关键词提供的内置锁时，只要线程没有获得锁，那么就会永远等待下去，然而Lock接口提供了boolean tryLock(long time, TimeUnit unit) throws InterruptedException方法，该方法可以按照固定时长等待锁，因此线程可以在获取锁超时以后，主动释放之前已经获得的所有的锁。通过这种方式，也可以很有效地避免死锁。 还是按照之前的例子，时序图如下：

![img](java%E5%9F%BA%E7%A1%80/20180922174924551)

## 避免死锁:

- 预防死锁的几种策略，会严重地损害系统性能。因此在避免死锁时，要施加较弱的限制，从而获得 较满意的系统性能。由于在避免死锁的策略中，允许进程动态地申请资源。因而，系统在进行资源分配之前预先计算资源分配的安全性。若此次分配不会导致系统进入不安全的状态，则将资源分配给进程；否则，进程等待。其中最具有代表性的避免死锁算法是银行家算法。
- 银行家算法：首先需要定义状态和安全状态的概念。系统的状态是当前给进程分配的资源情况。因此，状态包含两个向量Resource（系统中每种资源的总量）和Available（未分配给进程的每种资源的总量）及两个矩阵Claim（表示进程对资源的需求）和Allocation（表示当前分配给进程的资源）。安全状态是指至少有一个资源分配序列不会导致死锁。当进程请求一组资源时，假设同意该请求，从而改变了系统的状态，然后确定其结果是否还处于安全状态。如果是，同意这个请求；如果不是，阻塞该进程知道同意该请求后系统状态仍然是安全的。

## 检测死锁

1. 首先为每个进程和每个资源指定一个唯一的号码；
2. 然后建立资源分配表和进程等待表。

## 解除死锁:

当发现有进程死锁后，便应立即把它从死锁状态中解脱出来，常采用的方法有：

- 剥夺资源：从其它进程剥夺足够数量的资源给死锁进程，以解除死锁状态；
- 撤消进程：可以直接撤消死锁进程或撤消代价最小的进程，直至有足够的资源可用，死锁状态.消除为止；所谓代价是指优先级、运行代价、进程的重要性和价值等。

# 死锁检测

**1、Jstack命令**

jstack是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息。 Jstack工具可以用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

**2、JConsole工具**

Jconsole是JDK自带的监控工具，在JDK/bin目录下可以找到。它用于连接正在运行的本地或者远程的JVM，对运行在Java应用程序的资源消耗和性能进行监控，并画出大量的图表，提供强大的可视化界面。而且本身占用的服务器内存很小，甚至可以说几乎不消耗。 --------------------- 本文来自 江溢jonny 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/jonnyhsu_0913/article/details/79633656?utm_source=copy

参考博文：https://blog.csdn.net/jonnyhsu_0913/article/details/79633656

[         https://blog.csdn.net/sinat_21043777/article/details/53457233](https://blog.csdn.net/sinat_21043777/article/details/53457233)

[         https://blog.csdn.net/wsq119/article/details/82218911](https://blog.csdn.net/wsq119/article/details/82218911)







# 序列化

## 一、序列化和反序列化的概念



![这里写图片描述](java%E5%9F%BA%E7%A1%80/20180408163613978)

　　**把==对象==转换为==字节序列==的过程称为对象的==序列化==**。
　　**把==字节序列==恢复为==对象==的过程称为对象的==反序列化==**。

![这里写图片描述](java%E5%9F%BA%E7%A1%80/20180408163634701)

　　对象的序列化主要有两种==用途==：
　　1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；
　　2） 在网络上传送对象的字节序列。

​		==反序列化==的最重要的作用：

根据字节流中保存的对象状态及描述信息，通过反序列化==重建对象==。



　　在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是Web服务器中的Session对象，当有 10万用户并发访问，就有可能出现10万个Session对象，内存可能吃不消，于是Web容器就会把一些seesion先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

　　当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象。



序列化最重要的作用：在传递和保存对象时.保证对象的完整性和可传递性。对象转换为有序字节流,以便在网络上传输或者保存在本地文件中。

​    

  ==总结==：核心作用就是对象状态的保存和重建。（整个过程核心点就是字节流中所保存的对象状态及描述信息）



## 二、哪些属性不序列化

​       　　　==static== 属性不会被序列化，反序列化时恢复为当前类变量的值。

　　　　　　因为 static 代表“一个类一个”，而不是“一个对象一个”，因此它不是对象的状态。

　　　　如果某个实例变量不能或不应该被序列化，就把它标记为 transient（瞬时）的。

　　　　　　当某个属性的类型没有实现 Serializable 接口，而又不能修改该类，那么该实例变量不能被实例化。

　　　　　　那么只能把该属性标记为==transient== 。或者动态数据只可以在执行时求出而不能或不必存储。

　　　　　　被标记为 transient 的数学，在反序列化时，被恢复为默认值。

## 三、**JDK类库中的序列化API**

　　java.io.Object==Output==Stream代表对象输出流，它的==writeObject(Object obj)==方法可对参数指定的obj对象进行==序列化==，把得到的字节序列写到一个目标输出流中。
　　java.io.Object==Input==Stream代表对象输入流，它的==readObject()==方法从一个源输入流中读取字节序列，再把它们==反序列化==为一个对象，并将其返回。
　　只有实现了Serializable和Externalizable接口的类的对象才能被序列化。Externalizable接口继承自 Serializable接口，实现Externalizable接口的类完全由自身来控制序列化的行为，而仅实现Serializable接口的类可以 采用默认的序列化方式 。
　　对象序列化包括如下步骤：
　　1） 创建一个对象输出流，它可以包装一个其他类型的目标输出流，如文件输出流；
　　2） 通过对象输出流的writeObject()方法写对象。

　　对象反序列化的步骤如下：
　　1） 创建一个对象输入流，它可以包装一个其他类型的源输入流，如文件输入流；
　　2） 通过对象输入流的readObject()方法读取对象。



## Java有很多基础类已经实现了serializable接口，

比如String,Vector等。但是也有一些没有实现serializable接口的；







# JSON

![img](java%E5%9F%BA%E7%A1%80/041348087856490.png)

![img](java%E5%9F%BA%E7%A1%80/041348098488149.png)



~~~java

~~~







## **maven依赖包：**

~~~java
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.47</version>
</dependency>
~~~



## **一、FastJson**是用于**java**后台处理**json**格式数据的一个工具包，包括“序列化”和“反序列化”两部分，它具备如下特征**：**

>   （1）速度最快，测试表明，fastjson具有极快的性能，超越任其他的java json parser。
>   （2）功能强大，完全支持java bean、集合、Map、日期、Enum，支持范型，支持自省。
>   （3）无依赖，能够直接运行在Java SE 5.0以上版本

## **二、FastJson**对于**json**格式字符串的解析主要用到了一下三个类：

>   （1）JSON：fastJson的解析器，用于JSON格式字符串与JSON对象及javaBean之间的转换。
>   （2）JSONObject：fastJson提供的json对象。
>   （3）JSONArray：fastJson提供json数组对象。

 

## **三、测试所需的实体类**

```java
public class Data implements Serializable {

    private static final long serialVersionUID = -6957361951748382519L;
    private String id;
    private String suborderNo;
    private String organUnitType;
    private String action;
    private String parent;
    private String organUnitFullName;
    private Long ordinal;

    get、set方法省略。。。
}

```

 

~~~java
public class Error implements Serializable {

    private static final long serialVersionUID = -432908543160176349L;
    private String code;
    private String message;
    private String success;
    private List<Data> data = new ArrayList<>();

  get、set方法省略。。。
}
~~~





## **四、JSON**格式字符串、**JSON**对象及**JavaBean**之间的相互转换

###   **4.1)**

###  JAVA**对象转**JSON字符串

```java
    //java对象转json字符串

    public static void beanTojson() {
        Data data = new Data();
        data.setAction("add");
        data.setId("1");
        data.setOrdinal(8L);
        data.setOrganUnitFullName("testJSON");
        data.setParent("0");
        data.setSuborderNo("58961");
        String s = JSON.toJSONString(data);
        System.out.println("toJsonString()方法：s=" + s);

        //输出结果{"action":"add","id":"1","ordinal":8,"organUnitFullName":"testJSON","parent":"0","suborderNo":"58961"}



    }
```

###   **4.2)**

###  A. JSON**字符串转**JSON对象

```java
    //json字符串转json对象

    public static void jsonToJsonBean() {
        String s ="{\"action\":\"add\",\"id\":\"1\",\"ordinal\":8,\"organUnitFullName\":\"testJSON\",\"parent\":\"0\",\"suborderNo\":\"58961\"}";


        JSONObject jsonObject = JSON.parseObject(s);
        String action = jsonObject.getString("action");
        String id = jsonObject.getString("id");
        
        System.out.println("action ="+action);//add
        System.out.println("id ="+id);//1
        System.out.println("jsonObject ="+jsonObject);


        //action =add
        //id =1
        //jsonObject ={"parent":"0","organUnitFullName":"testJSON","action":"add","id":"1","suborderNo":"58961","ordinal":8}
    }
```

###     **B.** 复杂JSON格式字符串与**JSONObject**之间的转换

```java
 public static void jsonToBean() {


        String str ="{\"meta\":{\"code\":\"0\",\"message\":\"同步成功!\"},\"data\":{\"orderno\":\"U_2018062790915774\",\"suborderno\":\"SUB_2018062797348039\",\"type\":\"organunit\",\"result\":{\"organunit\":{\"totalCount\":2,\"successCount\":0,\"failCount\":2,\"errors\":[{\"code\":\"UUM70004\",\"message\":\"组织单元名称不能为空\",\"data\":{\"id\":\"254\",\"suborderNo\":\"SUB_2018062797348039\",\"organUnitType\":\"部门\",\"action\":\"add\",\"parent\":\"10000\",\"ordinal\":0,\"organUnitFullName\":\"组织单元全称\"},\"success\":false},{\"code\":\"UUM70004\",\"message\":\"组织单元名称不能为空\",\"data\":{\"id\":\"255\",\"suborderNo\":\"SUB_2018062797348039\",\"organUnitType\":\"部门\",\"action\":\"add\",\"parent\":\"10000\",\"ordinal\":0,\"organUnitFullName\":\"组织单元全称\"},\"success\":false}]},\"role\":{\"totalCount\":0,\"successCount\":0,\"failCount\":0,\"errors\":[]},\"user\":{\"totalCount\":0,\"successCount\":0,\"failCount\":0,\"errors\":[]}}}}";



        JSONObject jsonObject = JSON.parseObject(str);
        JSONObject data = jsonObject.getJSONObject("data");
        JSONObject result = data.getJSONObject("result");
        String organunit1 = result.getString("organunit");
        System.out.println(organunit1);
        JSONObject organunit = result.getJSONObject("organunit");
        JSONArray errors2 = organunit.getJSONArray("errors");
        List<Error> error = JSON.parseObject(errors2.toJSONString(), new TypeReference<List<Error>>() {
        });
    }
```

###   **4.3) **

### A. JSON**字符串转**JAVA简单对象

```java
    //json字符串转java简单对象
    public static void jsonStrToJavaBean() {

        String s ="{\"action\":\"add\",\"id\":\"1\",\"ordinal\":8,\"organUnitFullName\":\"testJSON\",\"parent\":\"0\",\"suborderNo\":\"58961\"}";

        Data data = JSON.parseObject(s, Data.class);
        System.out.println("data对象"+data.toString());
        System.out.println("action="+data.getAction()+"---id="+data.getId());

        //data对象Data{id='1', suborderNo='58961', organUnitType='null', action='add', parent='0', organUnitFullName='testJSON', ordinal=8}


        //action=add---id=1
 
        
        
        /**
         * 另一种方式转对象
         */
        Data dd = JSON.parseObject(s, new TypeReference<Data>() {});
        System.out.println("另一种方式获取data对象"+dd.toString());
        System.out.println("另一种方式获取="+dd.getAction()+"---id="+dd.getId());
        //另一种方式获取data对象Data{id='1', suborderNo='58961', organUnitType='null', action='add', parent='0', organUnitFullName='testJSON', ordinal=8}
        //另一种方式获取=add---id=1
    }
```

###     **B. JSON**字符串 数组类型与**JAVA**对象的转换

测试json字符串

```java
//json字符串--数组型与JSONArray对象之间的转换

    public static void jsonStrToJSONArray() {

        String str = {"errors":[{"code":"UUM70004","message":"组织单元名称不能为空","data":{"id":"254","suborderNo":"SUB_2018062797348039","organUnitType":"部门","action":"add","parent":"10000","ordinal":0,"organUnitFullName":"组织单元全称"},"success":false},{"code":"UUM70004","message":"组织单元名称不能为空","data":{"id":"255","suborderNo":"SUB_2018062797348039","organUnitType":"部门","action":"add","parent":"10000","ordinal":0,"organUnitFullName":"组织单元全称"},"success":false}]};

        JSONObject jsonObject = JSON.parseObject(str);
        JSONArray error = jsonObject.getJSONArray("errors");
        List<Error> errors = JSON.parseObject(error.toJSONString(), new TypeReference<List<Error>>() {
        });

        for (Error e: errors) {
            //Error的属性
            System.out.println("Error属性="+e.getSuccess());
            System.out.println("Error属性="+e.getCode());
            System.out.println("Error属性="+e.getMessage());
            //Error集合属性

            List<Data> datas = e.getData();
            for (Data d: datas) {
                System.out.println("data对象属性="+d.getId());
                System.out.println("data对象属性="+d.getAction());
                System.out.println("data对象属性="+d.getSuborderNo());
            }
        }

        //Error属性=false
        //Error属性=UUM70004
        //Error属性=组织单元名称不能为空
        
        //data对象属性=254
        //data对象属性=add
        //data对象属性=SUB_2018062797348039
        
        //Error属性=false
        //Error属性=UUM70004
        //Error属性=组织单元名称不能为空
        
        //data对象属性=255
        //data对象属性=add
        //data对象属性=SUB_2018062797348039



    }
```

###     **C. JSON**字符串 第二种方法-->数组类型与**JAVA**对象的转换

```java
   //第二种方法：json字符串--数组型与JSONArray对象之间的转换
    @Test
    public void jsonStrToJSONArray2() {

        String str = "{\"errors\":[{\"code\":\"UUM70004\",\"message\":\"组织单元名称不能为空\",\"data\":{\"id\":\"254\",\"suborderNo\":\"SUB_2018062797348039\",\"organUnitType\":\"部门\",\"action\":\"add\",\"parent\":\"10000\",\"ordinal\":0,\"organUnitFullName\":\"组织单元全称\"},\"success\":false},{\"code\":\"UUM70004\",\"message\":\"组织单元名称不能为空\",\"data\":{\"id\":\"255\",\"suborderNo\":\"SUB_2018062797348039\",\"organUnitType\":\"部门\",\"action\":\"add\",\"parent\":\"10000\",\"ordinal\":0,\"organUnitFullName\":\"组织单元全称\"},\"success\":false}]}";


        //获取jsonobject对象
        JSONObject jsonObject = JSON.parseObject(str);
        
        //把对象转换成nArray数组
        JSONArray error = jsonObject.getJSONArray("errors");

        //error==>[{"code":"UUM70004","message":"组织单元名称不能为空","data":{"id":"254","suborderNo":"SUB_2018062797348039","organUnitType":"部门","action":"add","parent":"10000","ordinal":0,"organUnitFullName":"组织单元全称"},"success":false},{"code":"UUM70004","message":"组织单元名称不能为空","data":{"id":"255","suborderNo":"SUB_2018062797348039","organUnitType":"部门","action":"add","parent":"10000","ordinal":0,"organUnitFullName":"组织单元全称"},"success":false}]

        //将数组转换成字符串
        String jsonString = JSONObject.toJSONString(error);//将array数组转换成字符串
        
        //将字符串转成list集合
        List<Error>  errors = JSONObject.parseArray(jsonString, Error.class);//把字符串转换成集合



        for (Error e: errors) {
            
            //Error的属性
            System.out.println("另一种数组转换Error属性="+e.getSuccess());
            System.out.println("另一种数组转换Error属性="+e.getCode());
            System.out.println("另一种数组转换Error属性="+e.getMessage());

            //Error集合属性
            List<Data> datas = e.getData();
            for (Data d: datas) {
                System.out.println("另一种数组转换data对象属性="+d.getId());
                System.out.println("另一种数组转换data对象属性="+d.getAction());
                System.out.println("另一种数组转换data对象属性="+d.getSuborderNo());
            }
        }
        

        //另一种数组转换Error属性=false
        //另一种数组转换Error属性=UUM70004
        //另一种数组转换Error属性=组织单元名称不能为空
        
        //另一种数组转换data对象属性=254
        //另一种数组转换data对象属性=add
        //另一种数组转换data对象属性=SUB_2018062797348039
        
        //另一种数组转换Error属性=false
        //另一种数组转换Error属性=UUM70004
        //另一种数组转换Error属性=组织单元名称不能为空

        //另一种数组转换data对象属性=255
        //另一种数组转换data对象属性=add
        //另一种数组转换data对象属性=SUB_2018062797348039


    }
```

###  **4.4) **

### **JAVA**对象转**JSON**对象

```java
    //javabean转json对象
    public static void jsonBenToJsonObject() {

       
        Data data = new Data();
        
        data.setAction("add");
        data.setId("1");
        data.setOrdinal(8L);
        data.setOrganUnitFullName("testJSON");
        data.setParent("0");
        data.setSuborderNo("58961");


        JSONObject jsonObj = (JSONObject) JSON.toJSON(data);
        JSON json = (JSON) JSON.toJSON(data);

        System.out.println("jsonObj"+jsonObj);
        System.out.println("json对象"+json);

 //jsonObj{"parent":"0","organUnitFullName":"testJSON","action":"add","id":"1","suborderNo":"58961","ordinal":8}



        //json对象{"parent":"0","organUnitFullName":"testJSON","action":"add","id":"1","suborderNo":"58961","ordinal":8}



    }
```

### **五、后记**

> （1）对于JSON对象与JSON格式字符串的转换可以直接用 toJSONString()这个方法。
> （2）javaBean与JSON格式字符串之间的转换要用到：JSON.toJSONString(obj);
> （3）javaBean与json对象间的转换使用：JSON.toJSON(obj)，然后使用强制类型转换，JSONObject或者JSONArray。























































































































































































































































