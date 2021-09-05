## 多线程进阶->JUC并发编程

## 1、什么是JUC

java.util工具包、包、类

## 2、线程和进程

进程：一个程序的集合，比如QQ.exe Music.exe

一个进程往往可以包含多个线程，至少包含一个！

java默认有两个线程：main、GC

线程：开了一个进程Typro写字，自动保存(线程负责的)

**java真的可以开启线程吗？**

本地方法，底层c++，java不能直接操作硬件 private native void start0();

所以不能

> 并发、并行

 并发编程：并发、并行

并发（多线程操作同一个资源）

 cpu单核，模拟出来多条线程，天下武功为快不破，快速交替*

并行（多个人一起行走）

cpu多核，多个线程可以同时执行

并发编程的本质：**充分利用cpu的资源**

> 线程有几个状态

新生
NEW,
 运行
RUNNABLE,
 阻塞
BLOCKED,
 等待，死死地等
WAITING,
超时等待
TIMED_WAITING,
终止
TERMINATED;

> wait/sleep区别

### 1、来自不同的类
​	wait->object
​	sleep->Thread:企业中不会用

### 2、关于锁的释放
​	wait会释放锁，sleep睡觉了，抱着锁睡觉，不会释放
### 3、使用的范围是不同的
 	wait：必须在同步代码块中使用
 	 sleep可以再任何地方睡
### 4、是否需要捕获异常
 	wait:不需要捕获异常
 	 sleep:必须要捕获异常,超时等待的异常



所有的线程都有中断异常：InterruptedException

## Lock锁（重点）

> 传统的synchronized



```java
package com.jie.demo01;
/**
 * 公司中真正的多线程开发，降低耦合性
 * 线程就是一个单独的资源类，没有任何附属的操作
 * 属性和方法
 *
 */
public class SaleTicketDemo01 {
    public static void main(String[] args) {
        //并发,多线程操作同一个资源类,把资源类丢入线程
        Ticket ticket = new Ticket();
        //@FunctionalInterface 函数式接口
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"a").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"b").start();
        new Thread(()->{
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        },"c").start();
    }
}
//资源类oop
class Ticket{

    //属性方法
    private  int number=30;

    //卖票的方法
    //synchronized,本质：排队，队列，锁
    public synchronized void sale(){
        if(number>0){
            System.out.println(Thread.currentThread().getName()+"卖出了"+(number--)+"票，剩余"+number);
        }
    }
}

```



> Lock接口



公平锁： FairSync(),非常公平，可以先来后到 

非公平鎖：NonfairSync()，非常不公平，可以插队(默认)，为了公平，每个线程执行的时间



```java
package com.jie.demo01;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
/**
 *
 * 锁的是对象、class
 *
 * synchronized和lock锁的区别：
 * 1.synchronized 内置的java关键字，Lock是java的一个类
 * 2.synchronized 无法判断获取锁的状态，Lock可以判断是否获取到了锁
 * 3.synchronized 自动释放锁（a执行完会释放，b抛异常也会释放），lock 必须要手动释放锁！如果不释放锁，死锁
 * 4.synchronized 线程1(获得锁，阻塞)、线程2(等待，傻傻的等)；lock锁就不一定等待下去
 * 5.synchronized 可重入锁，不可中断的，非公平的；lock，可重入锁，可以判断锁，非公平(可以自己设置)
 * 6.synchronized 适合锁少量的代码同步问题，lock适合锁大量的同步代码
 *
 * 锁是什么，如何判断锁的是什么？
 */
public class SaleTicketDemo02 {
    public static void main(String[] args) {
        //并发,多线程操作同一个资源类,把资源类丢入线程
        Ticket2 ticket = new Ticket2();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) ticket.sale();
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) ticket.sale();
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) ticket.sale();
        }, "C").start();
    }
}

/**
 * 第一步：new ReentrantLock();
 * 第二步：lock.lock();加锁
 * 第三步：finally -> lock.unlock();解锁
 */
//资源类oop
class Ticket2 {
    //属性方法
    private int number = 30;

    /**
     * public ReentrantLock(boolean fair) {
     * sync = fair ? new FairSync() : new NonfairSync();
     * }
     * <p>
     * 公平锁： FairSync(),非常公平，可以先来后到
     * 非公平鎖：NonfairSync()，非常不公平，可以插队(默认)，为了公平，每个线程执行的时间
     */
    Lock lock = new ReentrantLock();
    public void sale() {

        lock.lock();
        try {
            //业务代码
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了" + (number--) + "票，剩余" + number);
            }

        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            //解锁
            lock.unlock();
        }
    }

}

```



>synchronized和lock锁的区别：

 * 1.synchronized 内置的java关键字，Lock是java的一个类
 * 2.synchronized 无法判断获取锁的状态，Lock可以判断是否获取到了锁
 * 3.synchronized 自动释放锁（a线程执行完会释放，b线程抛异常也会释放），lock 必须要手动释放锁！如果不释放锁，死锁
 * 4.synchronized 线程1(获得锁，阻塞)、线程2(等待，傻傻的等)；lock锁就不一定等待下去
 * 5.synchronized 可重入锁，不可中断的，非公平的；lock，可重入锁，可以判断锁，非公平(可以自己设置)
 * 6.synchronized 适合锁少量的代码同步问题，lock适合锁大量的同步代码

> 锁是什么，如何判断锁的是什么？



## 4、生产者和消费者问题

> 生产者和消费者问题synchronized版



```java
package com.jie.pc;

/**
 * 线程之间的通讯问题：生产者和消费者问题！等待唤醒，通知唤醒
 * 线程交替执行 A  B操作同一个变量  num=0
 * A num+1
 * B num-1
 *
 *
 */
public class A {
    public static void main(String[] args) {

        Data data = new Data();
        new Thread(()->{

            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
    }
}

//判断等待，业务，通知
class Data{
    private  int number=0;
    //+1
    public synchronized  void increment() throws InterruptedException {
        if(number!=0){
            //等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName()+"-->"+number);
        //通知其他线程，我加1完毕了
        this.notifyAll();
    }
    //-1
    public synchronized  void decrement() throws InterruptedException {
        if(number==0){
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"-->"+number);
        //通知其他线程，我减1完毕了
        this.notifyAll();
    }
}
```

> 问题，现在是A和B线程，那么这个时候有ABCD四个线程呢，虚假唤醒

if改为while判断

```java
package com.jie.pc;

/**
 * 线程之间的通讯问题：生产者和消费者问题！等待唤醒，通知唤醒
 * 线程交替执行 A  B操作同一个变量  num=0
 * A num+1
 * B num-1
 *
 *
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{

            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}
//判断等待，业务，通知
class Data{
    private  int number=0;

    //+1
    public synchronized  void increment() throws InterruptedException {
      /*  if(number!=0){
            //等待
            this.wait();
        }*/

        //解决，悬架唤醒：if改为while判断
        while(number!=0){
            //等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName()+"-->"+number);
        //通知其他线程，我加1完毕了
        this.notifyAll();
    }
    //-1
    public synchronized  void decrement() throws InterruptedException {
   /*     if(number==0){
            this.wait();
        } */
        while (number==0){
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"-->"+number);
        //通知其他线程，我减1完毕了
        this.notifyAll();
    }
}
```

> juc的生产者和消费者问题



![image-20210519160655026](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210519160655026.png)



通过lock找到condition

代码实现：

```java
package com.jie.pc;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
/**
 * 线程之间的通讯问题：生产者和消费者问题！等待唤醒，通知唤醒
 * 线程交替执行 A  B操作同一个变量  num=0
 * A num+1
 * B num-1
 *
 *
 */
public class B {
    public static void main(String[] args) {

        Data2 data = new Data2();
        new Thread(()->{

            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}

//判断等待，业务，通知
class Data2{
    private  int number=0;
    Lock lock=new ReentrantLock();
    Condition condition = lock.newCondition();
       // condition.await();//等待
        //condition.signalAll();//唤醒全部
    //+1
    public   void increment() throws InterruptedException {

        lock.lock();
        try {
            //业务代码
            //解决，悬架唤醒：if改为while判断
            while(number!=0){
                //等待
                condition.await();//等待
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"-->"+number);
            //通知其他线程，我加1完毕了
            condition.signalAll();//唤醒全部

        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            //解锁
            lock.unlock();
        }
    }
    //-1
    public   void decrement() throws InterruptedException {
        lock.lock();
        try {
            //业务代码
            while (number==0){
                condition.await();//等待
            }
            number--;
            System.out.println(Thread.currentThread().getName()+"-->"+number);
            //通知其他线程，我减1完毕了
            condition.signalAll();//唤醒全部

        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            //解锁
            lock.unlock();
        }
    }
}
```

任何一个新的技术绝不仅仅是覆盖原来的技术，绝对有补充和优势

> Condition精准的通知和唤醒线程



![image-20210519163414258](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210519163414258.png)



代码测试：

```java
package com.jie.pc;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
/**
 * 杰哥说
 */
public class C {
    public static void main(String[] args) {
        Data3 data3 = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data3.printC();
            }
        }, "C").start();
    }
}
class Data3 {
    private Lock lock = new ReentrantLock();
    //同步监视器，一个只能监视一个
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number=1;//1A,2B,3C
    public void printA() {
        lock.lock();
        try {
            //业务，判断->执行->通知
            while (number!=1){
                //等待
                condition1.await();
            }
            number=2;
            System.out.println(Thread.currentThread().getName()+"==>AAAAAAAAAAAA");
            //唤醒，唤醒指定的人B
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            //业务，判断->执行->通知
            while (number!=2){
                //等待
                condition2.await();
            }
            number=3;
            System.out.println(Thread.currentThread().getName()+"==>BBBBBBBBBBBBB");
            //唤醒，唤醒指定的人C
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

            lock.unlock();
        }
    }
    public void printC() {
        lock.lock();
        try {
            //业务，判断->执行->通知
            while (number!=3){
                //等待
                condition3.await();
            }
            number=1;
            System.out.println(Thread.currentThread().getName()+"==>CCCCCCCCCCCCCC");
            //唤醒，唤醒指定的人A
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

            lock.unlock();
        }
    }
}
```



## 5、八锁现象

如何判断锁是谁？

对象、Class模板

```java
package com.jie.lock8;
import java.util.concurrent.TimeUnit;
/**
 * 8锁就是关于锁的八个问题
 *
 * 1：标准情况下，两个线程会先打印 发短信还是打电话？  1/发短信，2/打电话
 *2：sendSms延迟4秒，两个线程会先打印 发短信还是打电话？   1发/短信，2/打电话
 */
public class Test1 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        //并不是因为A先调用 ，是因为有锁的存在
        new Thread(()->{
            phone.sendSms();
        },"A").start();
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone.call();
        },"B").start();
    }
}
class Phone{
    //synchronized 锁的对象是方法的调用者
    //两个方法用的是同一个锁，谁先拿到谁执行
    public synchronized void sendSms(){
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
}
```





```java
package com.jie.lock8;
import java.util.concurrent.TimeUnit;
/**
 * 3；增加了一个普通方法（hello）后两个线程执行完，先打印发短信还是hello  ??  1/hello 2/发短信
 *
 *4:两个对象，两个同步方法两个线程执行完，先打印发短信，还是打电话？  1先打电话  2后发短信 看时间
 */
public class Test2 {
    public static void main(String[] args) {
        //两个对象，两个调用者，两个不同的对象，两把锁
        Phone2 phone1 = new Phone2();
        Phone2 phone2 = new Phone2();
        //并不是因为A先调用 ，是因为有锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
           // phone1.hello();
            phone2.call();

        },"B").start();
    }
}
class Phone2{
    //synchronized 锁的对象是方法的调用者
    //两个方法用的是同一个锁，谁先拿到谁执行
    public synchronized void sendSms(){
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public synchronized void call(){
        System.out.println("打电话");
    }
    //这里没有锁，不是同步方法，不存在抢，不受锁的影响
    public  void hello(){
        System.out.println("hello");
    }
}
```



```java
package com.jie.lock8;
import java.util.concurrent.TimeUnit;
/**
 * 5:增加两个静态的同步方法，只有一个对象，先打印发短信？还是打电话？发短信
 *
 * 6：两个对象，增加两个静态的同步方法，先打印发短信？还是打电话？发短信
 */
public class Test3 {
    public static void main(String[] args) {
        //两个对象的类模板只有一个,static,锁的是class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();
        //并不是因为A先调用 ，是因为有锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            // phone1.hello();
            phone2.call();
        },"B").start();
    }
}
//Phone3唯一的Class，
class Phone3{
    //synchronized 锁的对象是方法的调用者
    //static 静态方法
    //类一加载就有了，锁的是Class，
    public static synchronized void sendSms(){
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    public static synchronized void call(){
        System.out.println("打电话");
    }
}
```



```java
package com.jie.lock8;
import java.util.concurrent.TimeUnit;
/**
 *7；一个静态的同步方法，一个普通的同步方法，一个对象，先打印发短信？还是打电话？ 先打印打电话，两把锁
 *8；一个静态的同步方法，一个普通的同步方法，两个对象，先打印发短信？还是打电话？ 先打印打电话
 */
public class Test4 {
    public static void main(String[] args) {
        //两个对象的类模板只有一个
        Phone4 phone1= new Phone4();
        Phone4 phone2= new Phone4();
        //并不是因为A先调用 ，是因为有锁的存在
        new Thread(()->{
            phone1.sendSms();
        },"A").start();
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            // phone1.hello();
            phone2.call();
        },"B").start();
    }
}

//Phone3唯一的Class，
class Phone4{
    //synchronized 锁的对象是方法的调用者
    //static 静态方法
    //类一加载就有了，锁的是Class类模板，
    //静态同步方法
    public static synchronized void sendSms(){
        //捕获异常
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }
    //普通同步方法，锁的是调用者
    public  synchronized void call(){
        System.out.println("打电话");
    }
}
```



> 小结



new  this 具体的一个手机

static Class 唯一的一个模板

## 6、集合类不安全

> List不安全

```java
package com.jie.unsafe;
import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.stream.Collectors;
//java.util.ConcurrentModificationException并发修改异常
public class ListTest {
    public static void main(String[] args) {
        //并发下ArrayList不安全，synchronized
        /**
         * 解决方案：
         *1、 List<String> list= new Vector<>();Vector线程安全,Vector比ArrayList后出来，所以不建议Vector
         *怎么让ArrayList变得安全
         * 2、List<String> list= Collections.synchronizedList(new ArrayList<>())
         *3、List<String> list= new CopyOnWriteArrayList<>();
         */
      //List<String> list= new ArrayList<>();
        //List<String> list= new Vector<>();
       // List<String> list= Collections.synchronizedList(new ArrayList<>()) ;
        // CopyOnWrite写入时复制  COW 计算机程序设计领域的一种优化策略
        //多个线程调用的时候,list,读取的时候，固定的，写入(覆盖)
        //再写入的时候避免覆盖，造成数据问题
        //CopyOnWriteArrayList比Vector牛逼在哪里？
        //Vector用的是synchronized，效率低；而CopyOnWriteArrayList用的lock锁，效率高
        //读写分离
        List<String> list= new CopyOnWriteArrayList<>();
        for (int i = 1; i <= 10; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(1,5));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

> Set不安全

```java
package com.jie.unsafe;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.CopyOnWriteArraySet;
/**
 * 同理可证：java.util.ConcurrentModificationException
 * 解决方案
 * 1:Set<String> set = Collections.synchronizedSet(new HashSet<>());
 * 2:Set<String> set = new CopyOnWriteArraySet<>();
 */
public class SetTest {
    public static void main(String[] args) {
       // Set<String> set = new HashSet<>();
       // Set<String> set = Collections.synchronizedSet(new HashSet<>());
        Set<String> set = new CopyOnWriteArraySet<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(1,5));
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
}
```

> HashSet底层是什么？

就是HashMap

```java
public HashSet() {
    map = new HashMap<>();
}
```

//add set本质是 map key是无法重复的

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

```java
private static final Object PRESENT = new Object();//不变的值
```

> HashMap不安全



```java
package com.jie.unsafe;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
//java.util.ConcurrentModificationException
public class MapTest {
    public static void main(String[] args) {
        //map是这样用的吗？不是，工作中不用这
        // 默认等价于什么？new HashMap<>(16,0.75);
       // Map<String, String> map = new HashMap<>();
      //  Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
        Map<String, String> map = new ConcurrentHashMap<>();
        //加载因子，初始化容量
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(1,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}
```



## 7、Callable



- ```
  @FunctionalInterface
  public interface Callable<V>
  ```

  返回结果并可能引发异常的任务。实现者定义一个没有参数的单一方法，称为`call` 。

  `Callable`接口类似于[`Runnable`](../../../java/lang/Runnable.html)  ，因为它们都是为其实例可能由另一个线程执行的类设计的。 然而，A  `Runnable`不返回结果，也不能抛出被检查的异常。 

  该[`Executors`](../../../java/util/concurrent/Executors.html)类包含的实用方法，从其他普通形式转换为`Callable`类。

1.可以有返回值；

2.可以抛出异常；

3.方法不同。run()/call();

> 代码测试



```java
package com.jie.caellable;
import lombok.SneakyThrows;
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;
public class CallableTest {
    @SneakyThrows
    public static void main(String[] args) {
       // new Thread(new Runnable()).start();
        //new Thread(new FutureTask<V>()).start();
        //new Thread(new FutureTask<V>(Callable)).start();

        MyThead myThead = new MyThead();
        FutureTask stringFutureTask = new FutureTask<>(myThead);//适配类
        new Thread(stringFutureTask,"A").start();
        String o = (String) stringFutureTask.get();//获取Callable的返回结果,get方法可能会产生阻塞，把他放到最后
        //或者使用一部通信来处理
        System.out.println(o);
    }
}
class MyThead implements Callable<String>{
    @Override
    public String call()  {
        System.out.println("call()");
        //耗时的操作
        return "123456";
    }
}
```





```java
package com.jie.caellable;
import lombok.SneakyThrows;
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;
public class CallableTest {
    @SneakyThrows
    public static void main(String[] args) {
       // new Thread(new Runnable()).start();
        //new Thread(new FutureTask<V>()).start();
        //new Thread(new FutureTask<V>(Callable)).start();
        MyThead myThead = new MyThead();
        FutureTask stringFutureTask = new FutureTask<>(myThead);//适配类
        new Thread(stringFutureTask,"A").start();
        new Thread(stringFutureTask,"B").start();//结果会被缓存,效率高
        String o = (String) stringFutureTask.get();//获取Callable的返回结果,get方法可能会产生阻塞，把他放到最后
        //或者使用一部通信来处理
        System.out.println(o);
    }
}
class MyThead implements Callable<String>{
    @Override
    public String call()  {
        System.out.println("call()");//会打印几个call()
        //耗时的操作
        return "123456";
    }
}
```

**细节：**

1.有缓存；

2.结果可能需要等待，会阻塞

## 8、常用的辅助类(必会)

### 8.1、CountDownLatch 减法计数器

- ```
  public class CountDownLatch
  extends Object
  ```

  允许一个或多个线程等待直到在其他线程中执行的一组操作完成的同步辅助。

  A `CountDownLatch`用给定的*计数*初始化。 [`await`](../../../java/util/concurrent/CountDownLatch.html#await--)方法阻塞，直到由于[`countDown()`](../../../java/util/concurrent/CountDownLatch.html#countDown--)方法的[调用](../../../java/util/concurrent/CountDownLatch.html#countDown--)而导致当前计数达到零，之后所有等待线程被释放，并且任何后续的`await`  [调用立即](../../../java/util/concurrent/CountDownLatch.html#await--)返回。  这是一个一次性的现象 - 计数无法重置。 如果您需要重置计数的版本，请考虑使用[`CyclicBarrier`](../../../java/util/concurrent/CyclicBarrier.html)  。 

  A `CountDownLatch`是一种通用的同步工具，可用于多种用途。  一个`CountDownLatch`为一个计数的CountDownLatch用作一个简单的开/关锁存器，或者门：所有线程调用[`await`](../../../java/util/concurrent/CountDownLatch.html#await--)在门口等待，直到被调用[`countDown()`](../../../java/util/concurrent/CountDownLatch.html#countDown--)的线程打开。  一个`CountDownLatch`初始化*N*可以用来做一个线程等待，直到*N个*线程完成某项操作，或某些动作已经完成N次。 

  `CountDownLatch`一个有用的属性是，它不要求调用`countDown`线程等待计数到达零之前继续，它只是阻止任何线程通过[`await`](../../../java/util/concurrent/CountDownLatch.html#await--)  ，直到所有线程可以通过。 

  **示例用法：**这是一组类，其中一组工作线程使用两个倒计时锁存器： 

  - 第一个是启动信号，防止任何工作人员进入，直到驾驶员准备好继续前进; 
  - 第二个是完成信号，允许司机等到所有的工作人员完成。 

  ```java
     class Driver { // ... void main() throws InterruptedException { CountDownLatch startSignal = new CountDownLatch(1); CountDownLatch doneSignal = new CountDownLatch(N); for (int i = 0; i < N; ++i) // create and start threads new Thread(new Worker(startSignal, doneSignal)).start(); doSomethingElse(); // don't let run yet startSignal.countDown(); // let all threads proceed doSomethingElse(); doneSignal.await(); // wait for all to finish } } class Worker implements Runnable { private final CountDownLatch startSignal; private final CountDownLatch doneSignal; Worker(CountDownLatch startSignal, CountDownLatch doneSignal) { this.startSignal = startSignal; this.doneSignal = doneSignal; } public void run() { try { startSignal.await(); doWork(); doneSignal.countDown(); } catch (InterruptedException ex) {} // return; } void doWork() { ... } } 
  ```

  另一个典型的用法是将问题划分为N个部分，用一个Runnable来描述每个部分，该Runnable执行该部分并在锁存器上倒计时，并将所有Runnables排队到执行器。  当所有子部分完成时，协调线程将能够通过等待。 （当线程必须以这种方式反复倒数时，请[改用`CyclicBarrier`](../../../java/util/concurrent/CyclicBarrier.html)  ）） 

  ```java
     class Driver2 { // ... void main() throws InterruptedException { CountDownLatch doneSignal = new CountDownLatch(N); Executor e = ... for (int i = 0; i < N; ++i) // create and start threads e.execute(new WorkerRunnable(doneSignal, i)); doneSignal.await(); // wait for all to finish } } class WorkerRunnable implements Runnable { private final CountDownLatch doneSignal; private final int i; WorkerRunnable(CountDownLatch doneSignal, int i) { this.doneSignal = doneSignal; this.i = i; } public void run() { try { doWork(i); doneSignal.countDown(); } catch (InterruptedException ex) {} // return; } void doWork() { ... } } 
  ```

  内存一致性效果：直到计数调用之前达到零，在一个线程操作`countDown()` [*happen-before*](package-summary.html#MemoryVisibility)以下由相应的成功返回行动`await()`在另一个线程。 

代码

减法计数器



```java
package com.jie.add;

import java.util.concurrent.CountDownLatch;
//计数器
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {

        //总数是6,必须要执行任务的时候，再使用
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"go out");
                countDownLatch.countDown();//总数-1
            },String.valueOf(i)).start();
        }
        countDownLatch.await();//等待计数器归零，然后再往下执行
        System.out.println("close doors");
    }
}
```



原理：

==countDownLatch.countDown();//总数-1==

==countDownLatch.await();//等待计数器归零，然后再往下执行==



每次线程调用.countDown()数量-1，假设计数器为0，countDownLatch.await()就会被唤醒，继续执行!

### 8.2、CyclicBarrier 加法计数器



- 允许一组线程全部等待彼此达到共同屏障点的同步辅助。循环阻塞在涉及固定大小的线程方的程序中很有用，这些线程必须偶尔等待彼此。屏障被称为*循环*  ，因为它可以在等待的线程被释放之后重新使用。

  A `CyclicBarrier`支持一个可选的[`Runnable`](../../../java/lang/Runnable.html)命令，每个屏障点运行一次，在派对中的最后一个线程到达之后，但在任何线程释放之前。  在任何一方继续进行之前，此*屏障操作*对更新共享状态很有用。 

  **示例用法：**以下是在并行分解设计中使用障碍的示例： 

  ```
     class Solver { final int N; final float[][] data; final CyclicBarrier barrier; class Worker implements Runnable { int myRow; Worker(int row) { myRow = row; } public void run() { while (!done()) { processRow(myRow); try { barrier.await(); } catch (InterruptedException ex) { return; } catch (BrokenBarrierException ex) { return; } } } } public Solver(float[][] matrix) { data = matrix; N = matrix.length; Runnable barrierAction = new Runnable() { public void run() { mergeRows(...); }}; barrier = new CyclicBarrier(N, barrierAction); List<Thread> threads = new ArrayList<Thread>(N); for (int i = 0; i < N; i++) { Thread thread = new Thread(new Worker(i)); threads.add(thread); thread.start(); } // wait until done for (Thread thread : threads) thread.join(); } } 
  ```

  这里，每个工作线程处理矩阵的一行，然后等待屏障，直到所有行都被处理。当处理所有行时，执行提供的[`Runnable`](../../../java/lang/Runnable.html)屏障操作并合并行。如果合并确定已经找到解决方案，那么`done()`将返回`true`  ，并且每个工作人员将终止。

  如果屏障操作不依赖于执行方暂停的各方，那么该方可以在释放任何线程时执行该操作。 为了方便这一点，每次调用[`await()`](../../../java/util/concurrent/CyclicBarrier.html#await--)返回该线程在屏障上的到达索引。  然后，您可以选择哪个线程应该执行屏障操作，例如： 

  ```
     if (barrier.await() == 0) { // log the completion of this iteration } 
  ```

  `CyclicBarrier`对失败的同步尝试使用all-or-none断裂模型：如果线程由于中断，故障或超时而过早离开障碍点，那么在该障碍点等待的所有其他线程也将通过[`BrokenBarrierException`](../../../java/util/concurrent/BrokenBarrierException.html)  （或[`InterruptedException`）异常离开](../../../java/lang/InterruptedException.html)如果他们也在同一时间被打断）。 

  内存一致性效果：线程中调用的行动之前， `await()` [*happen-before*](package-summary.html#MemoryVisibility)行动是屏障操作的一部分，进而*发生，之前*的动作之后，从相应的成功返回`await()`其他线程。

```java
package com.jie.add;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
//加法计数器
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙成功");
        });
        for (int i = 1; i <=7; i++) {
            final int temp=i;
            //lambda能操作到i吗，不能
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"手机"+temp+"个龙珠");
                try {
                    cyclicBarrier.await();//等待计数器加一
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```



### 8.3、Semaphore

Semaphore:信号量

- 一个计数信号量。在概念上，信号量维持一组许可证。如果有必要，每个[`acquire()`都会](../../../java/util/concurrent/Semaphore.html#acquire--)阻塞，直到许可证可用，然后才能使用它。每个[`release()`](../../../java/util/concurrent/Semaphore.html#release--)添加许可证，潜在地释放阻塞获取方。但是，没有使用实际的许可证对象;`Semaphore`只保留可用数量的计数，并相应地执行。

  信号量通常用于限制线程数，而不是访问某些（物理或逻辑）资源。  例如，这是一个使用信号量来控制对一个项目池的访问的类： 

  ```
     class Pool { private static final int MAX_AVAILABLE = 100; private final Semaphore available = new Semaphore(MAX_AVAILABLE, true); public Object getItem() throws InterruptedException { available.acquire(); return getNextAvailableItem(); } public void putItem(Object x) { if (markAsUnused(x)) available.release(); } // Not a particularly efficient data structure; just for demo protected Object[] items = ... whatever kinds of items being managed protected boolean[] used = new boolean[MAX_AVAILABLE]; protected synchronized Object getNextAvailableItem() { for (int i = 0; i < MAX_AVAILABLE; ++i) { if (!used[i]) { used[i] = true; return items[i]; } } return null; // not reached } protected synchronized boolean markAsUnused(Object item) { for (int i = 0; i < MAX_AVAILABLE; ++i) { if (item == items[i]) { if (used[i]) { used[i] = false; return true; } else return false; } } return false; } } 
  ```

  在获得项目之前，每个线程必须从信号量获取许可证，以确保某个项目可用。  当线程完成该项目后，它将返回到池中，并将许可证返回到信号量，允许另一个线程获取该项目。 请注意，当[调用`acquire()`](../../../java/util/concurrent/Semaphore.html#acquire--)时，不会保持同步锁定，因为这将阻止某个项目返回到池中。  信号量封装了限制对池的访问所需的同步，与保持池本身一致性所需的任何同步分开。 

  信号量被初始化为一个，并且被使用，使得它只有至多一个允许可用，可以用作互斥锁。  这通常被称为*二进制信号量* ，因为它只有两个状态：一个许可证可用，或零个许可证可用。  当以这种方式使用时，二进制信号量具有属性（与许多[`Lock`](../../../java/util/concurrent/locks/Lock.html)实现不同），“锁”可以由除所有者之外的线程释放（因为信号量没有所有权概念）。  这在某些专门的上下文中是有用的，例如死锁恢复。 

  此类的构造函数可选择接受*公平*参数。  当设置为false时，此类不会保证线程获取许可的顺序。 特别是，  *闯入*是允许的，也就是说，一个线程调用[`acquire()`](../../../java/util/concurrent/Semaphore.html#acquire--)可以提前已经等待线程分配的许可证-在等待线程队列的头部逻辑新的线程将自己。  当公平设置为真时，信号量保证调用[`acquire`](../../../java/util/concurrent/Semaphore.html#acquire--)方法的线程被选择以按照它们调用这些方法的顺序获得许可（先进先出;  FIFO）。 请注意，FIFO排序必须适用于这些方法中的特定内部执行点。  因此，一个线程可以在另一个线程之前调用`acquire`  ，但是在另一个线程之后到达排序点，并且类似地从方法返回。 另请注意， [未定义的`tryAcquire`](../../../java/util/concurrent/Semaphore.html#tryAcquire--)方法不符合公平性设置，但将采取任何可用的许可证。 

  通常，用于控制资源访问的信号量应该被公平地初始化，以确保线程没有被访问资源。  当使用信号量进行其他类型的同步控制时，非正常排序的吞吐量优势往往超过公平性。 

  本课程还提供了方便的方法， [一次`acquire`](../../../java/util/concurrent/Semaphore.html#acquire-int-)和[`release`](../../../java/util/concurrent/Semaphore.html#release-int-)多个许可证。  当没有公平地使用这些方法时，请注意增加无限期延期的风险。 

  内存一致性效应：在另一个线程中[成功](package-summary.html#MemoryVisibility)执行“获取”方法（如`acquire()`之前，调用“释放”方法之前的线程中的操作，例如`release()`  [*happen-before*](package-summary.html#MemoryVisibility)  。 

```java
package com.jie.add;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;
public class SemaphoreDemo {
    public static void main(String[] args) {

        //线程数量，停车位。限流
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                //acquire() 得到
                //release() 释放
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();//释放
                }
            },String.valueOf(i)).start();
        }
    }
}
```

原理：

`acquire()  获得,假设如果已经满了，等待，等待被释放为止

`release() 释放`，会将当前的信号量释放+1,然后唤醒等待的线程。

作用：

多个共享的资源互斥的使用；并发限流，控制最大的线程数。



## 9、读写锁

ReadWriteLock

读的时候可以被多个线程读，写的时候，只能由一个线程去写

![image-20210525150113599](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210525150113599.png)



```java
package com.jie.rw;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;
/**
 * 独占锁(写锁) 一次只能被一个线程占有
 * 共享锁(读锁) 多个线程可以同时占有
 *
 *
 * ReadWriteLock
 * 读-读 可以共存
 * 读-写 不能共存
 * 写-写 不能共存
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        //没有枷锁的缓存，写的时候会有插队
       // MyCache myCache = new MyCache();
        //加锁的缓存
        MyCacheLock myCache = new MyCacheLock();
        //多个线程写入
        for (int i = 1; i <=5; i++) {
            final int temp=i;
            new Thread(()->{

                myCache.put(temp+"",temp+"");
            },String.valueOf(i)).start();
        }
        //读取
        for (int i = 1; i <=5; i++) {
            final int temp=i;
            new Thread(()->{
                myCache.get(temp+"");
            },String.valueOf(i)).start();
        }
    }
}
//没有加锁的缓存
//自定义缓存
class MyCache{
    private volatile Map<String,Object> map=new HashMap<>();
    //存，写
    public void put(String key,Object value){
        System.out.println(Thread.currentThread().getName()+"写入"+key);
        map.put(key,value);
        System.out.println(Thread.currentThread().getName()+"写入ok");

    }
    //取，读
    public void get(String key){
        System.out.println(Thread.currentThread().getName()+"读取"+key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName()+"读取ok");
    }
}
//加锁的缓存
class MyCacheLock{
    private volatile Map<String,Object> map=new HashMap<>();
    //读写锁，更加细粒度的控制
     private ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
    //存，写入得时候，只希望同时只有一个线程写；
    public void put(String key,Object value){
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"写入"+key);
            map.put(key,value);
            System.out.println(Thread.currentThread().getName()+"写入ok");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
    //取，读,所有人都可以读
    public void get(String key){
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"读取"+key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName()+"读取ok");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
```

## 10、阻塞队列



![image-20210525155136004](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210525155136004.png)

阻塞队列：



![image-20210525155556515](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210525155556515.png)

![image-20210525161151267](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210525161151267.png)

BlockingQueue不是新东西

什么情况下我们使用阻塞队列：多线程并发处理，线程池！



 ![image-20210525161345262](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210525161345262.png)

**学会使用队列**

添加、移除

**四组API**

1、抛出异常

2、不会抛出异常

3、阻塞 等待

4、超时等待

| 方式         | 抛出异常  | 有返回值，不会抛出异常 | 阻塞 等待 | 超时等待  |
| ------------ | --------- | ---------------------- | --------- | --------- |
| 添加         | add       | offer()                | put()     | offer(,,) |
| 移除         | remove    | poll()                 | take()    | poll(,)   |
| 检测队首元素 | element() | peek()                 |           |           |



```java
package com.jie.bq;
import java.util.concurrent.ArrayBlockingQueue;
public class Test {
    public static void main(String[] args) {
        test1();
    }
    /**
     * 抛出异常
     */
    public static void test1(){
        //队列的大小
        ArrayBlockingQueue<Object> blockQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockQueue.add("a"));
        System.out.println(blockQueue.add("b"));
        System.out.println(blockQueue.add("c"));
        //添加第四个时，抛异常，java.lang.IllegalStateException: Queue full
       // System.out.println(blockQueue.add("d"));
        System.out.println(blockQueue.remove());
        System.out.println(blockQueue.remove());
        System.out.println(blockQueue.remove());
        //队列里面已经没有元素，往出取： java.util.NoSuchElementException
        //System.out.println(blockQueue.remove());
    }
}
```

```
/**
 * 有返回值，没有抛出异常
 */
public static void test2(){
    //队列的大小
    ArrayBlockingQueue<Object> blockQueue = new ArrayBlockingQueue<>(3);
    System.out.println(blockQueue.offer("a"));
    System.out.println(blockQueue.offer("a"));
    System.out.println(blockQueue.offer("a"));
    System.out.println(blockQueue.offer("a"));//false,不抛出异常
    System.out.println("++++++++++++++++++++++++++++++=============================");
    System.out.println(blockQueue.poll());
    System.out.println(blockQueue.poll());
    System.out.println(blockQueue.poll());
    System.out.println(blockQueue.poll());//返回null,不抛出异常

}
```



```java
/**
 * 等待，阻塞(一直等待)
 */
public static void test3() throws InterruptedException {
    //队列的大小
    ArrayBlockingQueue<Object> blockQueue = new ArrayBlockingQueue<>(3);

    blockQueue.put("a");
    blockQueue.put("b");
    blockQueue.put("c");
    //blockQueue.put("d");//队列没位置了，一直阻塞

    System.out.println(blockQueue.take());
    System.out.println(blockQueue.take());
    System.out.println(blockQueue.take());
   // System.out.println(blockQueue.take());//没有这个元素
    
}
```

```java
/**
 * 等待，阻塞(等待超时)
 */
public static void test4() throws InterruptedException {
    //队列的大小
    ArrayBlockingQueue<Object> blockQueue = new ArrayBlockingQueue<>(3);
    blockQueue.offer("a");
    blockQueue.offer("a");
    blockQueue.offer("a");
    blockQueue.offer("a",2, TimeUnit.SECONDS);//等待超过两秒就退出
    System.out.println("=================================================");
    System.out.println(blockQueue.poll());
    System.out.println(blockQueue.poll());
    System.out.println(blockQueue.poll());
    blockQueue.poll(2,TimeUnit.SECONDS);//等待超过两秒就退出
}
```



> SynchronousQueue 同步队列

没有容量

进去一个元素，必须等待取出来之后，才能往里面放一个元素

put、take

```java
package com.jie.bq;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;
/**
 * 同步队列
 * 和其他的BlockingQueue不一样，SynchronousQueue不存储元素
 * put了一个元素，必须从里面先take,否则不能在put进去值
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingDeque = new SynchronousQueue<>();//同步队列
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread()+"put 1");
                blockingDeque.put("1");
                System.out.println(Thread.currentThread()+"put 2");
                blockingDeque.put("2");
                System.out.println(Thread.currentThread()+"put 3");
                blockingDeque.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {

            }
        },"T1").start();   
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"取"+blockingDeque.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"取"+blockingDeque.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"取"+blockingDeque.take());

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}
```

## 11、 线程池(重点)

线程池：三大方法、7大参数、4重拒绝策略

> 池化技术

程序运行，本质：占用系统的资源！优化资源的使用！=》池化技术

线程池、连接池、内存池、对象池///......创建，销毁，十分浪费资源

池化技术：事先准备一些资源，有人使用，就来我这里拿，用完之后还给我



**==线程池好处==**

1、降低资源的而消耗

2、提高响应的速度

3、方便管理

==线程复用、可以控制最大的并发数、管理线程==

> 线程池：3大方法

![image-20210526101700709](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526101700709.png)

```
package com.jie.pool;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
//Executors  工具类，3大方法
public class Demo01 {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();//单个线程
        //ExecutorService threadPool = Executors.newFixedThreadPool(5);//创建一个固定的线程池大小
        //ExecutorService threadPool = Executors.newCachedThreadPool();//可伸缩的《遇强则强，遇弱则弱

        try {
            for (int i = 0; i < 100; i++) {
                //使用线程池之后，使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //线程池用完，程序结束，关闭线程
            threadPool.shutdown();
        }


    }
}
```

> 7大参数

源码分析

```jave
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    
        public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    //
    public ThreadPoolExecutor(int corePoolSize,//核心线程大小
                              int maximumPoolSize,//最大核心线程池大小
                              long keepAliveTime,//超时了没有人调用就会释放
                              TimeUnit unit,//超时单位
                              BlockingQueue<Runnable> workQueue,//阻塞队列
                              ThreadFactory threadFactory,//线程工厂，创建线程的，一般不用动
                              RejectedExecutionHandler handler //拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```



![image-20210526112240053](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526112240053.png)





> 手动创建一个线程





```java
package com.jie.pool;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
//Executors  工具类，3大方法
public class Demo01 {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newSingleThreadExecutor();//单个线程
       // ExecutorService threadPool = Executors.newFixedThreadPool(5);//创建一个固定的线程池大小
        //ExecutorService threadPool = Executors.newCachedThreadPool();//可伸缩的《遇强则强，遇弱则弱
        try {
            for (int i = 0; i < 100; i++) {
                //使用线程池之后，使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //线程池用完，程序结束，关闭线程
            threadPool.shutdown();
        }
    }
}
```





> 四种拒绝策略

```java
package com.jie.pool;
import java.util.concurrent.*;
//Executors  工具类，3大方法
/**
 *四种拒绝策略：
 *  new ThreadPoolExecutor.AbortPolicy());//银行满了，还有人进来，不处理这个人的，抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy());//哪来的去哪里
 * new ThreadPoolExecutor.DiscardPolicy());//队列满了，丢掉任务，不会抛出异常
 * new ThreadPoolExecutor.DiscardOldestPolicy());//队列满了，尝试和最早的竞争，也不会抛出异常
 *
 */
public class Demo02 {
    public static void main(String[] args) {

        //自定义线程池！工作ThreadPoolExecutor
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy());//队列满了，尝试和最早的竞争，也不会抛出异常
        try {
            //最大承载Deque+max
            //超过抛 java.util.concurrent.RejectedExecutionException
            for (int i = 1; i <=9; i++) {
                //使用线程池之后，使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"  ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //线程池用完，程序结束，关闭线程
            threadPool.shutdown();
        }
    }
}
```



//Executors  工具类，3大方法
/**
 *四种拒绝策略：
 *  new ThreadPoolExecutor.AbortPolicy());//银行满了，还有人进来，不处理这个人的，抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy());//哪来的去哪里
 * new ThreadPoolExecutor.DiscardPolicy());//队列满了，丢掉任务，不会抛出异常
 * new ThreadPoolExecutor.DiscardOldestPolicy());//队列满了，尝试和最早的竞争，也不会抛出异常
 *
  */

![image-20210526120008864](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526120008864.png)

> 小结和拓展

池的最大大小如何去设置

了解，IO密集型，CPU密集型，（调优）

```
package com.jie.pool;
import java.util.concurrent.*;
public class Demo02 {
    public static void main(String[] args) {
        //自定义线程池！工作ThreadPoolExecutor
        //最大线程该如何定义
        //1.CPU 密集型   cpu多核： 几核，就是几，可以保证cpu的效率最高
        //2. IO 密集型 >判断你程序中十分耗IO的程序
        //程序中  15个大型任务  io十分占用资源
        //获取cpu的核数
        System.out.println(Runtime.getRuntime().availableProcessors());
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy());//队列满了，尝试和最早的竞争，也不会抛出异常
        try {
            //最大承载Deque+max
            //超过抛 java.util.concurrent.RejectedExecutionException
            for (int i = 1; i <=9; i++) {
                //使用线程池之后，使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"  ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //线程池用完，程序结束，关闭线程
            threadPool.shutdown();
        }
    }
}

```

## 12、四大函数式接口（必须掌握）

新时代的程序员：lambda表达式、链式编程、函数式接口、Stream流式计算

> 函数式接口：只有一个方法的接口

```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
//超级多的@FunctionalInterface
//简化编程模型，在新版本的框架底层大量应用
//foreach(消费类的函数式接口)
```



![image-20210526123201426](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526123201426.png)

代码测试：

> function 函数式接口

```java
package com.jie.function;
import java.util.function.Function;
public class Demo01 {
    public static void main(String[] args) {
     /*   Function function = new Function<String,String>(){
            @Override
            public String apply(String s) {
                return s;
            }
        };*/
       // Function function =(str)->{ return str;};
        Function function =str->{ return str;};
        System.out.println(function.apply("s"));
    }
}
```



>断定型接口

```
package com.jie.function;
import java.util.function.Predicate;

/**
 * 断定型接口，有一个输入参数，返回值只能是布尔值
 */
public class Demo02 {
    public static void main(String[] args) {

        //可以用于判断
        //判断一个字符串是否为空
/*        Predicate<String> predicate = new Predicate<String>() {
            @Override
            public boolean test(String str) {
                return str.isEmpty();
            }
        };*/
        Predicate<String> predicate =(str)->{
            return str.isEmpty();
        };
        System.out.println(predicate.test(""));
    }
}
```





> Consumer 消费型接口



```java
package com.jie.function;
import java.util.function.Consumer;
/**
 * Consumer 消费型接口，只有输入参数 没有返回值
 */
public class Demo03 {
    public static void main(String[] args) {
  /*      Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };*/
        Consumer<String> consumer =(str)->{
            System.out.println(str);
        };
        consumer.accept("str");
    }
}
```







> Supplier 供给型接口





```java
package com.jie.function;
import java.util.function.Supplier;
/**
 * 供给型接口，没有参数，只有返回值
 */
public class Demo04 {
    public static void main(String[] args) {
/*
        Supplier<Integer> supplier = new Supplier<Integer>() {
            @Override
            public Integer get() {
                System.out.println("get()");
                return 1024;
            }
        };*/
        Supplier<Integer> supplier =()->{
            return 1024;
        };
        System.out.println(supplier.get());
    }
}
```



## 13、Stream 流式计算

> 什么是Stream流式计算

大数据：存储+计算

集合、mysql本质就是存储东西的！

计算都应该交给流来计算；

![image-20210526160734360](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526160734360.png)



```java
package com.jie.stream;
import java.util.Arrays;
import java.util.List;
/**
 * 题目要求：一分钟内完成此题
 * 现在有5个用户！筛选：
 * 1、id必须是偶数
 * 2、年龄个必须大于23岁
 * 3、用户名转为大写字母
 * 4、用户名字母倒着排序
 * 5、只输出一个用户！
 */
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "B", 22);
        User u3 = new User(3, "C", 23);
        User u4 = new User(4, "D", 24);
        User u5 = new User(6, "E", 25);

        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
        //lambda表达式、链式编程、函数式接口、Stream流式计算
        list.stream()
                .filter(u -> {return u.getId()%2==0;})
                .filter(u->{return u.getAge()>23;})
                .map(u->{return u.getName().toUpperCase();})
                .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```

## 14、ForkJoin

> 什么是ForkJoin

(分支合并)ForkJoin在JDK1.7,并行执行任务！提高效率。大数据量！

大数据：Map Reduce(把大任务拆成小任务)



![image-20210526172702667](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526172702667.png)4



> ForkJoin 特点：工作窃取



这个里面维护的是双端队列

![image-20210526182233254](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210526182233254.png)





>ForkJoin 

![image-20210603120044563](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210603120044563.png)



![(C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210603120715367.png)





![image-20210603120856701](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210603120856701.png)



```java
package com.jie.forkjoin;
import java.util.concurrent.RecursiveTask;
/**
 * 求和计算的任务
 * <p>
 * 如何使用forkjoin
 * 1.forkJoinPool 通过执行
 * 2.计算任务 forkJoinPool.execute(ForkJoinTask task)
 */
public class ForkJoinDemo extends RecursiveTask<Long> {
    private Long start;
    private Long end;
    private Long temp = 1000_0000L;
    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute() {
        if ((end - start) < temp) {
            Long sum = 0L;

            for (Long i = start; i < end; i++) {
                sum += i;
            }
           // System.out.println(sum);
            return sum;
        } else {

            Long middle=(start+end)/2;
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();//拆分任务，把任务压入线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle+1, end);
            task2.fork();//拆分任务，把任务压入线程队列
            return task1.join()+task2.join();
        }
    }

}
```

```java
package com.jie.forkjoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();//7304
        test2();//3695
        test3();
    }

    //普通程序员(不可以调优)
    public static void test1() {
        long start = System.currentTimeMillis();
        Long sum = 0L;

        for (Long i = 0L; i < 10_0000_0000L; i++) {
            sum += i;
        }
        System.out.println(sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + "时间" + (end - start));
    }

    //会使用ForkJoin(可以调优)
    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);
        //同步提交
       // forkJoinPool.execute(task);//没有返回结果
        //异步提交
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);
        Long sum = submit.get();
        System.out.println(sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + "时间" + (end - start));
    }

    public static void test3() {
        long start = System.currentTimeMillis();

        //LongStream并行流，parallel并行，规约
        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        System.out.println(sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + "时间" + (end - start));
    }

}
```



## 15、异步回调

> Future设计的初衷:对将来的某个事件的结果进行建模

![image-20210608152107092](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210608152107092.png)



```java
package com.jie.future;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
/**
 * 异步调用：CompletableFuture
 * 异步执行
 * 成功回调
 * 失败回调
 */

public class Demo01 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //没有返回值的 runAsync 异步回调
 /*       CompletableFuture<Void> completableFuture=CompletableFuture.runAsync(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"runAsync");
        });

        System.out.println("1111111111111");//因为是异步，所以111111先打印，不会阻塞
        completableFuture.get();//获取阻塞执行结果*/

        //有返回值的 supplyAsync 异步回调

        //ajax，成功和失败的回调函数

        CompletableFuture<Integer> completableFuture=CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"supplyAsync");
           // int i= 10/0;
            return 1024;

        });
        System.out.println(completableFuture.whenComplete((t, u) -> {
            System.out.println("t=>" + t);//正常的返回结果 1024
            System.out.println("u=>" + u);//错误的信息,java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
        }).exceptionally((e) -> {

            e.printStackTrace();
            System.out.println(e.getMessage());
            return 233;//可以获取到错误的返回结果
        }).get());
    }
}
```

## 16、JMM

> 请你谈谈你对Volatile的理解

Volatile是java的虚拟机进提供***轻量级的同步机制*** 

1.保证可见性

2.不保证原子性

3.禁止指令重排

> 什么是JMM

JMM：Java内存模型，不存在的东西，概念！约定

关于JMM的一些同步约定：

1、线程解锁前：必须把共享变量==立刻==刷回主存

![image-20210608165346881](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210608165346881.png)

2、线程枷锁前：必须读取主存中的最新值到工作内存中！

3、枷锁和解锁是同一把锁

线程 **工作内存**、**主内存**

**8种操作:**

![image-20210608170109699](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210608170109699.png)

![image-20210608170411296](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210608170411296.png)

**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）**

- lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
- unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
- use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
- assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
- store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
- write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

　**JMM对这八种指令的使用，制定了如下规则：**

- 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
- 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
- 不允许一个线程将没有assign的数据从工作内存同步回主内存
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
- 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
- 对一个变量进行unlock操作之前，必须把此变量同步回主内存

问题：程序不知道主内存的值已经被修改过了

![image-20210608172245051](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210608172245051.png)



## 17、Volatile

> 1、保证可见性

```java
package com.jie.volatiletest;

import java.util.concurrent.TimeUnit;

public class JMMDemo {

    //不加volatile 程序就会死循环
    //加volatile 可以保证可见性
   // private static int num=0;
   private volatile static int num=0;
    public static void main(String[] args) {

        new Thread(()->{//线程1对主内存的变化是不知道的
            while (num==0){
            }
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        num=1;
        System.out.println(num);



    }
}
```

> 2.不保证原子性

原子性：不可分割

线程A在执行任务的时候，不能被打扰的 ，也不能被分割。要么同时成功，要么同时失败。

```java
package com.jie.volatiletest;
//volatile不保证原子性
public class VDemo2 {

    private volatile static int num=0;

    public static void add(){
        num++;
    }
    public static void main(String[] args) {
        //理论上num结果为2万
        for (int i = 0; i < 20; i++) {
           new Thread(()->{
               for (int j = 0; j < 1000; j++) {
                   add();
               }
           }).start();

        }
        while (Thread.activeCount()>2){//main  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+"    "+num);

    }
}
```

==如果不加lock和synchonized,怎么样保证原子性==

javap -c a.class  //反编译为字节码文件![image-20210609214756024](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210609214756024.png)

使用原子类解决原子性问题

![image-20210609215038036](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210609215038036.png)

```java
package com.jie.volatiletest;

import java.util.concurrent.atomic.AtomicInteger;

//volatile不保证原子性
public class VDemo2 {

    //private volatile static int num=0;
    //原子类的Integer
    private volatile static AtomicInteger num=new AtomicInteger();

    public static void add(){
        num.getAndIncrement();//AtomicInteger+1,CAS
    }
    public static void main(String[] args) {
        //理论上num结果为2万
        for (int i = 0; i < 20; i++) {
           new Thread(()->{
               for (int j = 0; j < 1000; j++) {
                   add();
               }
           }).start();

        }
        while (Thread.activeCount()>2){//main  gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName()+"    "+num);

    }
}
```

这些类的底层直接和操作系统挂钩！在内存中修改值 Unsafe是一个很特殊的存在

> 指令重排

什么是指令重排:**你写的程序，计算机并不是按照你写的那样去执行的。**

源代码->编译器优化的重排->指令并行也可能会重排->内存系统也会重排->执行

处理器在进行指令重排的时候，考虑：数据之间的依赖性！

```
int x=1;//1
int y=2;//2
x=x+5;//3
y=x*x; //4

我们所期望的：1234，但是可能执行的时候会变成2134 1324
可不可能是 4123！

```

可能造成影响的结果：abxy这四个值默认都是0；

| 线程A | 线程B |
| :---- | ----- |
| x=a   | y=b   |
| b=1   | a=2   |

正常结果：x=0;y=0;



| 线程A | 线程B |
| :---- | ----- |
| b=1   | y=b   |
| x=a   | a=2   |

指令重排的诡异结果：x=2 ,y=1



> 非计算机专业

**volatile可以避免指令重排：**

内存屏障。cpu指令。作用：

1、保证特定的操作的执行顺序！

2、可以保证某些变量的内存可见性(利用这些特性volatile实现了可见性)

![image-20210609223520779](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210609223520779.png)

**volatile是可以保持可见性。不能保证原子性,由于内存屏障，可以保证避免指令重排的现象产生！**

## 18、彻底玩转单例模式

饿汉式、DCL懒汉式，深究！

> 饿汉式

```java
package com.jie.single;
//饿汉式单例
public class Hungry {

    //耗内存资源，可能会浪费空间
    private byte[]  data1=new byte[1024*1024];
    private byte[]  data2=new byte[1024*1024];
    private byte[]  data3=new byte[1024*1024];
    private byte[]  data4=new byte[1024*1024];

    //构造器私有
    private Hungry(){

    }
    //初始化的时候创建
    private final static Hungry HUNGRY=new Hungry();
    public static Hungry getInstance(){
        return HUNGRY;
    }
}
```





> DCL懒汉式

```java
package com.jie.single;
//饿汉式单例
public class LazyMan {

    private LazyMan(){
        System.out.println(Thread.currentThread().getName()+"ok");
    }
   // private  static  LazyMan lazyMan;
    private volatile static  LazyMan lazyMan;
    //volatile保证原子性，防止指令重排
    public static LazyMan getInstance(){
       /* if(lazyMan==null){
            lazyMan=new LazyMan();
        }*/
        //双重检测锁模式的懒汉式单例DCL懒汉式
        if(lazyMan==null){
            synchronized (LazyMan.class){
                if(lazyMan==null){
                    lazyMan=new LazyMan();//不是原子性操作
                }
            }
        }
        return lazyMan;
    }
    /**
     * 1、分配内存空间
     * 2、执行构造方法、初始化对象
     * 3、把这个对象只想这个空间
     * 123
     * 132 A
     *     B //此时lazyMan还没有完成构造
     */
    //单线程下确实单例ok

    //多线程并发,会有问题
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {

            new Thread(()->{
                LazyMan.getInstance();
            }).start();
        }
    }
}
```

> 静态n内部类

```java
package com.jie.single;

//静态内部类
public class Holder {
    private Holder(){

    }

    public static Holder getInstance(){
        return InnerClass.HOLDER;
    }
    public static  class InnerClass{
        private static final  Holder HOLDER=new Holder();
    }

}
```



> 单例不安全，有反射





> 枚举

```Java
package com.jie.single;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

//enum 是什么？本身也是一个class类
public enum EnumSingle {

    INSTANCE;

    public EnumSingle getInstance(){
        return INSTANCE;
    }



}
class Test{
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        EnumSingle instance1=EnumSingle.INSTANCE;
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class,int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle instance2 = declaredConstructor.newInstance();
        System.out.println(instance1.hashCode());
        System.out.println(instance2);
    }
}
```







枚举类型的最终反编译源码：

// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   EnumSingle.java

package com.jie.single;


public final class EnumSingle extends Enum
{

```java
public static EnumSingle[] values()
{
    return (EnumSingle[])$VALUES.clone();
}

public static EnumSingle valueOf(String name)
{
    return (EnumSingle)Enum.valueOf(com/jie/single/EnumSingle, name);
}

private EnumSingle(String s, int i)
{
    super(s, i);
}

public EnumSingle getInstance()
{
    return INSTANCE;
}

public static final EnumSingle INSTANCE;
private static final EnumSingle $VALUES[];

static 
{
    INSTANCE = new EnumSingle("INSTANCE", 0);
    $VALUES = (new EnumSingle[] {
        INSTANCE
    });
}
```

## 19、深入理解CAS



> 什么是CAS

```java
package com.jie.cas;

import java.util.concurrent.atomic.AtomicInteger;
//CAS  compareAndSet比较并交换
public class CASDemo {
    public static void main(String[] args) {


        AtomicInteger atomicInteger = new AtomicInteger(2020);

        //如果期望的值达到了，那么就更新，否则不更新，CAS是cpu的并发原语！
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
        atomicInteger.getAndIncrement();//++
        System.out.println(atomicInteger.compareAndSet(2020, 2021));

        System.out.println(atomicInteger.get());
    }
}
```



> Unsafe

![image-20210618155507327](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210618155507327.png)



![image-20210618155735600](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210618155735600.png)



![image-20210618155854944](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210618155854944.png)



CAS：比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就一直循环，底层是自旋锁。自带原子性

==缺点==：

1、循环会耗时

2、一次性只能保证一个共享变量的原子性

3、ABA问题







> CAS ABA问题(狸猫换太子)，

![image-20210618161335607](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210618161335607.png)

```java
package com.jie.cas;

import java.util.concurrent.atomic.AtomicInteger;
//CAS  compareAndSet比较并交换
public class CASDemo {
    public static void main(String[] args) {


        AtomicInteger atomicInteger = new AtomicInteger(2020);

        //如果期望的值达到了，那么就更新，否则不更新，CAS是cpu的并发原语！
        System.out.println(atomicInteger.compareAndSet(2020, 2021));
        System.out.println(atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(2021, 2020));
        System.out.println(atomicInteger.get());
        //atomicInteger.getAndIncrement();//++
        System.out.println(atomicInteger.compareAndSet(2020, 666));

        System.out.println(atomicInteger.get());
    }
}
```





## 20、原子引用



> 解决ABA问题,引入原子引用！对应的思想：乐观锁！

带版本号的原子操作

```java
package com.jie.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

//CAS  compareAndSet比较并交换
public class CASDemo2 {

    //AtomicStampedReference 注意如果泛型是一个包装类，注意对象的引用问题
    //正常业务操作，这里面比较得都是一个个对象
   static AtomicStampedReference<Integer> integerAtomicStampedReference = new AtomicStampedReference<>(1, 1);

    public static void main(String[] args) {


        new Thread(()->{

           int stemp= integerAtomicStampedReference.getStamp();//获取版本号
            System.out.println("a1=>"+stemp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //如果期望的值达到了，那么就更新，否则不更新，CAS是cpu的并发原语！
            System.out.println(integerAtomicStampedReference.compareAndSet(1, 2,integerAtomicStampedReference.getStamp(),integerAtomicStampedReference.getStamp()+1));
            System.out.println("a2=>"+integerAtomicStampedReference.getStamp());
            System.out.println(integerAtomicStampedReference.compareAndSet(2, 1,integerAtomicStampedReference.getStamp(),integerAtomicStampedReference.getStamp()+1));
            System.out.println("a3=>"+integerAtomicStampedReference.getStamp());
        },"a").start();

        //原理跟乐观锁相同
        new Thread(()->{

            int stemp= integerAtomicStampedReference.getStamp();//获取版本号
            System.out.println("b1=>"+stemp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //如果期望的值达到了，那么就更新，否则不更新，CAS是cpu的并发原语！
            System.out.println(integerAtomicStampedReference.compareAndSet(1, 6,integerAtomicStampedReference.getStamp(),integerAtomicStampedReference.getStamp()+1));
            System.out.println("b2=>"+integerAtomicStampedReference.getStamp());

        },"b").start();

    }
}
```

**注意**

Interger使用了对象缓存机制，默认范围是-128~127，推荐使用静态工厂方法valueOf获取对象实例，因为valueOf使用缓存，而new 一定会创建新的对象分配新的内存空间

![image-20210619112840810](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210619112840810.png)

## 21、各种锁的理解



### 1、公平锁、非公平锁

公平锁：非常公平，不能够插队，必须先来后到!

非公平锁：非常不公平，可以插队（默认都是非公平）

```
public ReentrantLock() {
    sync = new NonfairSync();
}
```

```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### 2、可重入锁

可重入锁（递归锁）

![image-20210620145241683](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210620145241683.png)

> synchronized版

```java
package com.jie.lock;

public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new  Thread(()->{
            phone.sms();
        },"A").start();

        new  Thread(()->{
            phone.sms();
        },"B").start();

    }
}

class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName()+"sms");
        call();//这里也有把锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName()+"call");
    }
}
```

> Lock版

```java
package com.jie.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new  Thread(()->{
            phone.sms();
        },"A").start();

        new  Thread(()->{
            phone.sms();
        },"B").start();

    }
}

class Phone2{

    Lock lock=new ReentrantLock();

    public  void sms(){
        lock.lock();//细节问题：lock.lock() lock.unlock();
        //锁必须配对，否则会死在里面
       //lock.lock();//没有配对
        try {
            System.out.println(Thread.currentThread().getName()+"sms");
             call();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            //lock.unlock();
        }

    }
    public  void call(){
        lock.lock();

        try {
            System.out.println(Thread.currentThread().getName()+"call");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
}
```



### 3、自旋锁

spinlock







```
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```



自定义自旋锁

```java
package com.jie.lock;

import java.util.concurrent.atomic.AtomicReference;

/**
 * 自旋锁
 */
public class SpinLockdemo {

    //int 0
    //Thread 对象null
    AtomicReference<Thread> atomicReference= new AtomicReference<>();

    //加锁
    public void  myLock(){

        Thread thread = Thread.currentThread();

        System.out.println(Thread.currentThread().getName()+"==>mylock");
        while(!atomicReference.compareAndSet(null,thread)){

        }
    }

    //解锁
    public void  myUnLock(){

        Thread thread = Thread.currentThread();

        System.out.println(Thread.currentThread().getName()+"==>myUnLock");
      atomicReference.compareAndSet(thread,null);


    }



}
```



测试

```java
package com.jie.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class TestSpinLock {
    public static void main(String[] args) throws InterruptedException {

        /*ReentrantLock reentrantLock=new ReentrantLock();
        reentrantLock.lock();
        reentrantLock.unlock();*/

        SpinLockdemo spinLockdemo = new SpinLockdemo();

        new Thread(()->{
            spinLockdemo.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinLockdemo.myUnLock();
            }
        },"T1"

        ).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            spinLockdemo.myLock();
            try {

                TimeUnit.SECONDS.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                spinLockdemo.myUnLock();
            }
        },"T2"

        ).start();



    }
}
```



### 4、死锁

> 死锁是什么

![image-20210622211947362](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622211947362.png)

```
package com.jie.lock;
import java.util.concurrent.TimeUnit;

/**
 * 死锁
 */
public class DeadLockDemo {
    public static void main(String[] args) {

        String lockA="lockA";
        String lockB="lockB";
        new Thread(new MyThread(lockA,lockB),"T1").start();
        new Thread(new MyThread(lockB,lockA),"T2").start();


    }
}
class MyThread implements Runnable{

    private String lockA;
    private String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {

        synchronized (lockA){
            System.out.println(Thread.currentThread().getName()+"lock"+lockA+"=>get"+lockB);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (lockB){
                System.out.println(Thread.currentThread().getName()+"lock"+lockB+"=>get"+lockA);
            }
        }

    }
}
```



> 解决问题

1、jdk bin  使用jps -l 定位进程号

![image-20210622213925140](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622213925140.png)

2、使用jstack 进程号 找到死锁问题

![image-20210622214253988](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622214253988.png)

面试、工作中!排查问题

1、日志

2、查看堆栈信息