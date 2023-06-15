[多线程](https://www.liaoxuefeng.com/wiki/1252599548343744/1255943750561472)
[TOC]
# Java多线程 
方法区和堆区只有一个，每个线程对应一个栈区，主线程的栈底是main方法，其它线程的栈底是run方法。
### Java实现多线程的三种方式：
第一种：通过继承Thread，然后重写run方法。
第二种：创建Thread实例时，传入一个Runnable接口的实现类，或者匿名内部类。
第三种：Runnable接口的缺点是无法捕获异常，以及无法获得返回值。jdk5提供了Callable接口和Future接口，可以获得返回值以及捕获异常。jdk8中引入了CompletableFuture来弥补Future的不足。之后再详细介绍。

### 守护线程
线程分为两种，一种是用户线程，用于处理用户事务，只要有一个用户线程没有结束，JVM就不会退出。另一种是守护线程，它是为用户线程服务的线程，它的存亡无关紧要，垃圾回收线程是最典型的守护线程，如果用户线程全部结束，此时不管是否还有守护线程，JVM都会退出，因此不要在守护线程中使用需要关闭的资源。
如何自定义一个守护线程？在线程start之前，调用`setDiament(true)`即可。

### 线程优先级
java中线程的优先级分为10级，最低是1，最高是10，默认为5。但是线程优先级非常依赖于操作系统，比如Solaris OS中，线程有$2^{31}$个等级，但是windows中只有7个。因此java中的10个优先级等级，会映射到操作系统的优先级等级当中，具体怎么映射，还需要看具体的操作系统。比如java中的1级和2级优先级会对应windows最低级的优先级。

### Thread详解
##### yield
提示调度器当前线程愿意让出处理器，调度器可以忽略这个提示。也就是说，即使执行了yield，当前线程也可能继续执行。

##### join
join的用法很简单，在线程t1中调用t2.join()，代表t1线程会等待t2线程执行结束。
==为什么可以实现？下面这个例子中，明明是thread调用的join，那么在这个join里运行的wait，就应该把thread线程所拥有的对象锁（也就是thread）释放掉，这时候thread不是应该暂停执行吗？== wait是将当前线程阻塞，当前线程是thread1而不是thread。
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("main start");
        Thread thread = new Thread(() -> {
            Utils.printString("thread", 1000);
        });
        Thread thread1 = new Thread(() -> {
            Utils.printString("thread1-1", 100);
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Utils.printString("thread1-2", 100);
        });
        thread.start();
        thread1.start();
    }
}
```
currentThread
onSpinWait
start
interrupt  interrupted  isInterrupted
isAlive 
setPriority getPriority setName getName setDaemon isDaemon
getThreadGroup
activeCount
enumerate
dumpStack
getContextClassLoader setContextClassLoader
holdsLock
getStackTrace getAllStackTraces
State getState  
setDefaultUncaughtExceptionHandler getDefaultUncaughtExceptionHandler 
setUncaughtExceptionHandler getUncaughtExceptionHandler

##### 终止线程
* 使用标志位终止线程
使用被volatile修饰的flag终止线程
``` java
public class Main {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        Thread thread = new Thread(myThread);
        thread.start();
        ...
        myThread.flag = false;
        ...
    }
}
class MyThread implements Runnable{
    public volatile boolean flag = true;
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            if (true) {
                ...
            }
        }
    }
}
```
### synchronized关键字
括号中应该放置一个对象。
假设有t1到t5五个线程，我希望t4、t5不受影响，t1到t3实现线程安全。
则括号中应该放置t1、t2、t3共享的对象。每个对象都有对象锁，synchronized块实现线程安全是通过抢占<mark>对象锁</mark>实现的，当某个线程在同步块中执行时，相当于占用了这个对象锁，因此其它线程想要执行同步块，则必须等到这个线程退出同步块。
synchronized也可以修饰实例方法，同步锁是this。
也可以修饰静态方法，同步锁是类对象。

### volatile关键字
volatile关键字可以实现：可见性以及拒绝指令重排序。
每次使用由volatile修饰的对象时，都会从主内存中读取最新的值。每次修改volatile修饰的对象时，都会将修改后的值同步回主内存中。这两条规则保证了可见性。
通过内存屏障来实现拒绝指令重排序，在volatile所修饰对象之前的代码，在指令重排序之后不能排到该对象的后面。

### 定时器
定时器用来每隔一定时间执行一段代码，可以用java.util.Timer实现定时器。很多框架也支持定时任务。
```java
public class Main {
    public static void main(String[] args) throws ParseException {
        Timer timer = new Timer();
        
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date firstTime = sdf.parse("2021-11-08 00:00:00");
        timer.schedule(new LogTimerTask(), firstTime, 1000 * 10);
    }
}
class LogTimerTask extends TimerTask {//也可以用匿名内部类的形式
    @Override
    public void run() {
        ...//执行你的业务逻辑
    }
}
```
### CompletableFuture