# Java的一些特性：
Java字符串常量池：
通过new String创建的对象不进入常量池。
通过加法创建的对象，如果都是常数，则结果进入常量池。如果加法中有变量，则结果不进入常量池。（和编译优化有关）
intern方法：如果池中没有这个字符串，则在池中创建，并返回池中的引用。如果池中有这个字符串，直接返回池中的引用[Java intern() 方法](https://www.runoob.com/java/java-string-intern.html)



#### 继承：
 - 子类无法继承父类的static方法和变量。
 - 子类可以继承父类的private成员变量，但是看不到。但是可以用父类提供的public方法改变这个成员变量。
 - 父类被final修饰的类可以被继承，不能被重写，至于能不能被看到取决于private。

```java
//使用这种方法创建的对象son，只能使用Father拥有的变量和方法。
Father son = new Son();
```
#### 多态：
- 多态通过继承、接口来实现。
- 当使用多态方式调用方法时，在编译阶段检查父类中是否有该方法，如果没有，则编译错误；如果有，则编译通过，在运行阶段调用子类的同名方法。
#### 重载：
- 重载的方法，名称和返回值一定要相同，传入的参数一定要不同。

#### 重写：
- 子类对父类==可以访问的方法==进行重写
- 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的派生类（Java7之后的特性）
- final修饰的方法不能重写。static修饰的方法子类可以重新声明。
- 被子类重写的方法访问权限不能低于父类。

#### 反射：
通过Java反射机制可以操纵字节码文件(.class)
java.lang.Class:代表字节码文件
java.lang.reflect.Method:代表字节码中的方法字节码
java.lang.reflect.Constructor:代表字节码中的构造方法字节码
java.lang.reflect.Field:代表字节码中的属性字节码

下面这段代码会输出ture，因为类的字节码文件在程序编译之前就加载到方法区中， 字节码文件只有一个，因此无论通过何种方式获取的Class对象，只要类相同，都指向同一个方法区中的内存地址（即方法区中字节码文件的内存地址）
```java
Class c1 = null;//第一种获取方式
try {
    c1 = Class.forName("java.lang.String");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
System.out.println("".getClass() == c1);// true 
System.out.println(String.class == c1);//true
```

以下的例子是通过反射机制创建String对象。 其功能相当于``new String(String s);``
Method和Field使用方法类似
```java
public static String stringConstructor(String s) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class c1 = Class.forName("java.lang.String");  //获取String的字节码文件
    Constructor<String> stringConstructor = c1.getConstructor(String.class);//获取特定的构造方法，括号中传入的是Class类，代表形参的类型。

    return stringConstructor.newInstance(s);
}
```

反射的使用场景：1）注解就是通过反射实现的；2）很多框架都是基于反射实现的；3）其它

#### 注解（annotation）
如果属性名是value且只有一个，则注解时可以省略value
注解的属性只能是byte char int short long float double boolean String Class以及它们的数组形式
属性只能是public，所以可以省略public。
```java
@Retention(RetentionPolicy.RUNTIME) //表示该注解可以保留至运行时，即可被反射机制发现。还有CLASS（和SOURCE（源文件）
@Target(ElementType.TYPE)//表示注解可以放在Class, interface (including annotation interface), enum, or record declaration上面。
public @interface MyAnnotation {
    String value();
}
```
通过反射获得注解
```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Main main = new Main();
        main.method();
    }
    public void method() throws ClassNotFoundException {
        Class myClass = Class.forName("easy.MyClass");
        if (myClass.isAnnotationPresent(MyAnnotation.class)) { //判断这个类是否被MyAnnotation修饰
            MyAnnotation myAnnotation = (MyAnnotation) myClass.getAnnotation(MyAnnotation.class);//获取反射对象。
            String value = myAnnotation.value();  //获取值

            System.out.println("This Class is modified by MyAnnotation!\nThe value is " + value);
        } else {
            System.out.println("This Class is not modified by MyAnnotation");
        }
    }
}

@MyAnnotation("something")
class MyClass {}
```
输出如下：
```css
This Class is modified by MyAnnotation!
The value is something
```

#### 动态代理
- 动态代理只能对接口进行代理，对没有实现接口的类，或者不是接口中的方法，则不能进行动态代理
- 对所有类进行代理，可以看CGlib
下面的例子中，Student是Person的实现类。
```java
public interface Person {
    void eat();
    void drink();
}

public class Student implements Person {
    public void read() {
        System.out.println("student is reading...");
    }

    @Override
    public void eat() {
        System.out.println("student is eating...");
    }

    @Override
    public void drink() {
        System.out.println("student is drinking...");
    }
}
```
动态代理类，实现在Person的每个方法前后都加上打印输出
```java
public class LogProxy {
    private Object target;   //代理目标，将这个目标的所有接口方法前后都加上打印输出
    public LogProxy(Object target) {
        this.target = target;
    }

    public Object getInstance() {  //只能返回接口，不是接口运行阶段会报错
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), 
            target.getClass().getInterfaces(), (proxy, method, args) -> {
                System.out.println("start");
                method.invoke(target, args);
                System.out.println("end");
                return null;
        });
    }
}

public class MyTest {
    @Test
    public void test01() {
        PersonProxy handler = new PersonProxy(new Student());
        Person person = (Person) handler.getInstance();
        person.eat();
    }
}
```