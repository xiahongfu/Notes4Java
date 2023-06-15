[TOC]
# Java关键字：
abstract、assert、boolean、break、byte、case、catch、char、class、continue、default、do、double、else、enum、extends、final、finally、float、for、implements、import、int、interface、instanceof、long、native、new、package、private、protected、public、return、short、static、strictfp、super、switch、synchronized、this、throw、throws、transient、try、void、volatile、while

#### 1. assert：断言，一般用在测试类中。

#### 2. enum：是一个特殊的类，一般用来表示一组常量。
```java
//很多项目中都有一个专门的类来声明项目中用到的所有常量，所以enmu用的不多。
enum Color { //也可以声明在Test类中
    RED, GREEN, BLUE; 
} 
public class Test{
    public static void main(String[] args){
        Color c1 = Color.RED;
        System.out.println(c1);
    }
}
```

#### 3. instanceof：Java中的二元运算符，左边是对象右边是类。
这个关键字可以理解成：a is an instance of B？只要返回真，a就可以强制转换成B。
Java编程规范要求，如果要使用向下转型，最好使用instanceof关键字判断能否进行转换。这对避免ClassCastException很有用。
更多细节：[Java instanceof用法详解](https://blog.csdn.net/kuangay/article/details/81563992)
```java
//注1：null和任何类都是false
String s = "";
System.out.println(s instanceof String); //返回true
```
#### 4. finally：和try catch一起用。无论程序是否正确执行，finally块都会被执行到。
 - finally块中有return时，将会覆盖try和catch块中的return。
 - 如果try或catch中有return，程序先将返回值存在某个地方，再执行finally，因此在finally中改变返回值没有用。
 - 在try中加System.exit(0)或者在try之外的地方出现错误，都会使得程序直接结束，不会执行finally
#### 5. strictfp：FP-strict，精确浮点。用来声明类，方法，接口。在声明的域内浮点数运算完全按照IEEE-754标准进行运算。防止出现不同硬件平台运算结果不一致的问题。用的很少。
#### 6. super：用super();方法在子类构造方法中调用父类构造方法。如果父类的method()方法被子类重写，用super.method();在子类使用父类的方法。
#### 7. synchronized：
#### 8. throw：用来在方法体中抛出异常，需要和throws连用。
#### 9. throws：用来申明该方法可能抛出的所有异常，使用该方法时需要用trycatch捕获。
```java
/*
throws相当于将出现的异常抛给上级。在上级中可以对异常进行捕获，
也可以继续往更上级抛出异常，在合适的地方进行处理
*/
class Test {
	void test01() throws IOException, InterruptedException {
		//抛出异常
		throw new IOException("this is an IOException");
	}
}
//不处理异常，继续向上级抛出。
class Main1 {
	public static void main(String[] args) throws IOException, InterruptedException {
		Test test = new Test();
		test.test01();
	}
}
//使用trycatch处理异常。
class Main2 {
	public static void main(String[] args) {
		Test test = new Test();
		try {
			test.test01();
		} catch (IOExcpetion e) {
			e.printStackTrace();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```
#### 10. transient：一个和序列化有关的关键字。更多细节：[Java transient关键字](https://www.runoob.com/w3cnote/java-transient-keywords.html)
- ArrayList源码中的array就使用transient修饰了，原因是ArrayList自己提供了更高效合适的序列化方法。可以减小序列化开销。参考：[Why does ArrayList use transient storage?](https://stackoverflow.com/questions/9848129/why-does-arraylist-use-transient-storage#:~:text=ArrayList%20implements%20Serializable%20,%20so%20it,s%20writeObject%20and%20readObject%20methods.)
#### 11. native： 允许Java 代码调用其它编程语言编写的代码
一个相关的概念是JNI：Java Native Interface
#### 11. volatile ：总结一下就是：不保证原子性，一定程度上保证有序性，保证可见性。
- 可见性：一个线程修改了变量，其他线程能够立即看得到修改后的值。
- 有序性：程序执行的顺序按照代码的先后顺序执行
- 对变量的写操作不依赖变量当前值、也不依赖其他变量的值时可以使用。
- 可用于多线程中的状态标记量、线程安全的单例模式的双重检查锁中。
更多细节：[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)
#### 12. final关键字：
- final修饰的**基本数据类型**变量一旦初始化就不能被修改。
- final修饰的**引用数据类型**变量一旦初始化就不能改变其指向的地址。
- final修饰的**类**不能被继承
- final修饰的**方法**可以被继承，但不能被子类修改
更多细节：[浅析Java中的final关键字](https://www.cnblogs.com/dolphin0520/p/3736238.html)
```java
final Object obj = new Object();
Object obj2 = new Object();
obj = new Object; //会报错。
obj = obj2;  //也会报错。
```