泛型-新特性（lamdba）-反射-IO-多线程-集合类-JVM

JDK1.5新特性

1.可变参数：长度不定->本质还是数组

数据类型 ... 参数名称

要求：一个方法声明中只能有一个可变参数、且必须位于方法声明最后一个参数

2.for-each

3.静态导入：可导入其他包中方法，若静态导入，即将此方法的所有静态域一并导入，可和本类定义的方法一样直接调用。

4.泛型：java语法糖

泛型类

```java
class MyClass<T>{
    T t;
}
```

泛型方法

```java
public <T> T test(T t){//第一个<T>表示该方法为泛型方法 第二个T为返回值类型为T 第三个T为参数类型为T
    return t;
}
```

泛型接口

```java
interface IInterface<T>{}
//1
class InterfaceImpl<T> implements IInterface<T>{}
//2
class InterfaceImpl implements IInterface<String>{}
```

#### 通配符-解决泛型参数统一化问题

1.**?**通配符  作用于方法参数声明

```java
class Myclass<T>{
    private T t;

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
}
public class Study {
    public static void main(String[] args) {
        Myclass<String> myclass=new Myclass<>();
        myclass.setT("好好学习");
        print(myclass);
        Myclass<Integer>myclass1=new Myclass<>();
        myclass1.setT(10);
        print(myclass1);
    }
    public static void print(Myclass<?> myclass){//表示能接收任意类型的Myclass对象
        System.out.println(myclass.getT());
    }
}

```

由于无法确定入参的类型，因此？通配符下的泛型参数，只能取得类中属性值，无法进行属性值的设置。

```java
public static void print(Myclass<?> myclass){//表示能接收任意类型的Myclass对象
    //该方法中不能设置属性值内容，因不清楚传入类型故无法设置，只能正常获取属性值
        System.out.println(myclass.getT());
    }
```

**2.设置泛型上限-用于泛型类的声明，也可用于方法参数**（面试重点）

泛型类声明：T extends 类（T<=类，T还是任意类型但此时的T类型必须是这个类本身或者子类）（继承）

方法参数：？ extends 类

eg:？ extends Number:

表示方法入参只能接收Number以及其子类对象

```java
class Myclass<T extends Number>{//此时T还是任意类型，但必须是Number或者其子类，不可接收无继承关系的类型，如String
    private T t;

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
}
public class Study {
    public static void main(String[] args) {
        Myclass<Integer>myclass=new Myclass<>();
        Myclass<Number>myclass1=new Myclass<>();
    }
    public static void print(Myclass<? extends Number> myclass){//此时方法参数只能接受Number及其子类类型，其他无继承关系的类不可接收
        //此时依旧不可设置属性值内容，因只知道是Number类或其子类，不知道具体是哪个类，若设置为Number类就存在向下转型，线程不安全问题，子类不一定能使用
        System.out.println(myclass.getT());
    }
}

```

**方法参数设置泛型上限仍然只能取得类中属性值，而无法设置值，因为设置父类值子类不一定能使用（父类不一定能发生向下转型变为子类）**

#### 且方法上限与类上限是独立存在的，互不影响

**3.设置泛型下限-只能用于方法参数**（面试重点）

？super 类（>=类）

表示方法入参只能接受类以及其父类对象

```java
class Myclass<T>{
    private T t;

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
}
public class Study {
    public static void main(String[] args) {
        Myclass<Object> myclass=new Myclass<>();
        print(myclass);
    }
    //方法参数设置泛型下限不仅可以取得类中属性值，还可以设置属性值。因为子类可以天然向上转型变为父类
    public static void print(Myclass<? super Integer> myclass){
        myclass.setT(10);
        System.out.println(myclass.getT());
    }
}

```

#### 方法参数设置泛型下限不仅可以取得类中属性值，还可以设置属性值。因为子类可以天然向上转型变为父类。

### 泛型接口

```java
interface Iinterface<T>{
    T test(T t);
}
```

```java
//1.子类实现接口时继续保留泛型
class IinterfaceImpl<T> implements Iinterface<T>{}//前面的<T>是为了表明该类为泛型，后面的<T>是指接口的T
//2.子类实现接口时直接确定类型
class Iinterface2Impl implements Iinterface<String>{}//直接确定类型
两者区别：
第一种因在具体实例化对象时才能确定类型，故可有多个类型的对象；
第二种因在子类实现时直接确定类型，故只有一种类型的对象
```

```java
interface Iinterface<T>{//接口
    T test(T t);//抽象方法返回值为任意类型，方法参数类型也为任意类型
}
//1.子类实现接口时继续保留泛型
class IinterfaceImpl<T> implements Iinterface<T>{//子类也为泛型类，即可接受任意类型，方法类型具体确定在实例化子类对象时确定

    @Override
    public T test(T t) {
        return t;
    }
}
//2.子类实现接口时直接确定类型
class Iinterface2Impl implements Iinterface<String>{

    @Override
    public String test(String s) {
        return s;
    }
}
public class Study {
    public static void main(String[] args) {
       Iinterface<String>iinterface=new IinterfaceImpl<>();//1
        System.out.println(iinterface.test("好好学习"));
        Iinterface iinterface1=new Iinterface2Impl();//2
        //也可不进行强转
        System.out.println(((Iinterface2Impl) iinterface1).test("除了学习，别无爱好！"));
    }
}

```

### 5.类型擦除（面试重点）

**泛型信息仅存在代码编译阶段，进入JVM之前，与泛型相关的信息会被擦除掉，专业术语就叫类型擦除。**

换句话，泛型类与普通类在JVM中没有任何区别。

**泛型类进入JVM之前会进行类型擦除，泛型类的类型参数如果没有指定类型上限，则擦出成为Object类；如果类型参数指定类型上限，擦除为相应类型上限。**

```java
<T>->Object

<T extends String>->String
```

测试：

```java
class Myclass<T>{
    public T t;
}
public class Study {
    public static void main(String[] args) {
        Myclass<String> myclass=new Myclass<>();
        Myclass<Integer>myclass1=new Myclass<>();
        System.out.println(myclass.getClass()==myclass1.getClass());
    }
}
//true

import java.lang.reflect.Field;

class Myclass<T,E>{
    public T t;
    public E e;
}
public class Study {
    public static void main(String[] args) {
        //定义时为String、Integer，进入JVM均为Object类，因在进入JVM之前进行了类型擦除
        Myclass<String,Integer> myclass=new Myclass<>();
        Field[] fields=myclass.getClass().getDeclaredFields();
        for(Field field:fields){
            System.out.println(field.getType());
        }
    }
}
//class java.lang.Object
//class java.lang.Object

import java.lang.reflect.Field;

class Myclass<T,E extends Number>{
    public T t;
    public E e;
}
public class Study {
    public static void main(String[] args) {
        Myclass<String,Integer> myclass=new Myclass<>();
        Field[] fields=myclass.getClass().getDeclaredFields();
        for(Field field:fields){
            System.out.println(field.getType());
        }
    }
}
//class java.lang.Object
//class java.lang.Number//擦除为相应上限
```



#### 枚举

```java
//利用多例模式实现有限个数
class Color{
    private String title;
    public static final int RED_FLAG=1;
    public static final int GREEN_FLAG=5;
    public static final int BLUUE_FLAG=10;
    private static final Color RED=new Color("red");
    private static final Color GREEN=new Color("green");
    private static final Color BLUE=new Color("blue");
    private Color(String title){
        this.title=title;
    }
    public static Color getInstance(int flag){
//        if(flag==1){
//            return RED;
//        }else if(flag==5){
//            return GREEN;
//        }else if(flag==10){
//            return BLUE;
//        }
        switch (flag){
            case RED_FLAG:
                return RED;
            case BLUUE_FLAG:
                return BLUE;
            case GREEN_FLAG:
                return GREEN;
                default:
                    return null;
        }
    }
    @Override
    public String toString() {
        return this.title;
    }
}
public class Study {
    public static void main(String[] args) {
        Color color=Color.getInstance(1);
        Color color1=Color.getInstance(Color.BLUUE_FLAG);
        System.out.println(color);
        System.out.println(color1);
    }
}

```

```java
//枚举解决同样问题
enum Color{//本质上是个枚举类
    RED,BLUE,GREEN;
}
public class Study {
    public static void main(String[] args) {
       Color color=Color.BLUE;
        System.out.println(color);
    }
}
```

Java中枚举使用enum关键字定义枚举。

#### 枚举就是一种多例设计模式。

#### 1.Enum类

JDK1.5新增的enum枚举结构并不是新的结构。**使用enum定义的枚举本质上是一个类**，默认继承java.lang.Enum类。

取得枚举名字：

public final String name()

取得枚举编号：

public final int ordinal()

取得所有枚举对象：//与foreach循环搭配

枚举类.values():Enum[]            //静态方法 返回所有的枚举数

```java
enum Color{
    RED,BLUE,GREEN;
}
public class Study {
    public static void main(String[] args) {
        Color color = Color.RED;
        for (Color color1 : Color.values()) {
            System.out.print(color1 + " ");
        }
    }
}
//RED BLUE GREEN
```

#### 面试：请解释enum和Enum的区别

enum是一个关键字，使用enum定义的枚举本质上相对于一个类继承了Enum这个抽象类而已。

#### 2.枚举中定义其他结构（枚举本质是类，类有的枚举均可以）--枚举对象必须放在首行声明

（1）构造方法、普通属性

```java
enum Color{
    RED("红红"),BLUE("蓝蓝"),GREEN("绿绿");
    private String title;
    private Color(String title) {//枚举就是多例模式，故构造方法必须私有化
        this.title = title;
    }
    public String getTitle(){
        return this.title;
    }

    //枚举继承了Enum抽象类，父类中已经覆写toString()方法，若枚举类中不覆写则输出对象名（RED BLUE GREEN），枚举类中覆写toString()方法则输出括号内内容（红红 蓝蓝 绿绿）
    @Override
    public String toString() {
        return this.title;
    }
}
public class Study {
    public static void main(String[] args) {
        Color color = Color.RED;
        for (Color color1 : Color.values()) {
            System.out.print(color1 + " ");
        }
    }
}
```

##### （2）枚举还可以实现接口，此时枚举中的每个对象都自动升为接口对象；

```java
interface IColor{
    String color();
}
enum ColorImpl implements IColor{
    RED("红红"),BLUE("蓝蓝"),GREEN("绿绿");
    private String title;
    private ColorImpl(String title) {
        this.title = title;
    }
    @Override
    public String toString() {
        return this.title;
    }

    @Override
    public String color() {
        return this.title;
    }
}
public class Study {
    public static void main(String[] args) {
        IColor iColor=ColorImpl.BLUE;
        System.out.println(iColor.color());
    }
}
```



### 注解（Annotion）--@interface（表示注解）

**1.准确覆写@Override**

检查当前类中的覆写方法与父类定义的同名方法是否相同，如果有任何一个地方不同，编译报错。

**2.声明过期处理@Deprecated**

应用场景：原有类或者方法在旧版本没有问题，但是在新版本不推荐使用，可以加上@Deprecated注解，明确表示不建议用户再使用此类、方法，但用户若还想继续用也可以。

**3.压制警告@SuppressWarnings**:解决右边黄线，可使用压制警告进行取消黄线

当调用某些操作可能产生问题的时候就会出现警告信息，但警告信息并不是异常。



### JDK1.8的新特性

JDK1.8开始接口允许出现以下两种方法

1.可以使用default来定义普通方法，需要通过对象来调用。（default不可省略，若省略就变为抽象方法）

2.可以使用static来定义静态方法，通过接口名来调用。

```java
interface IInterfece{
    void test();
    default void print(){//接口中的普通方法
        System.out.println("接口中所有子类对象都有此方法！");
    }
    static void fun(){
        System.out.println("接口的静态方法！");
    }
}
class InterfaceImpl implements IInterfece{

    @Override
    public void test() {
        System.out.println("抽象方法的方法覆写！");
    }
}
public class Study {
    public static void main(String[] args) {
       IInterfece iInterfece=new InterfaceImpl();
       iInterfece.print();
       iInterfece.test();
       IInterfece.fun();//接口静态方法调用
    }
}
```

#### 但不到迫不得已还是将接口定义为纯粹的抽象类（抽象方法和全局变量）



### Lambda表达式--函数式编程

```java
@FunctionalInterface
interface IInterfece{
    int add(int x,int y);
}
public class Study {
    public static void main(String[] args) {
//       IInterfece iInterfece=new IInterfece() {//匿名的方法内部类
//           @Override
//           public int add(int x, int y) {
//               return x+y;
//           }
        //多行代码需要用括号括起来（抽象方法有返回值时必须由return语句返回）
//        IInterfece iInterfece=(x,y)-> {
//            System.out.println(x+y);
//            System.out.println("好好学习！");
//            return x+y;
//        };
        //抽象方法有返回值时，可直接写，return可以省略
        IInterfece interfece=(x,y)->x+y;
        System.out.println(interfece.add(10,3));
    }
}
```

**使用函数式编程前提：接口必须只有一个方法。如果接口中存在两个以上的方法，无法使用函数式编程语法！**

**如果现在某个接口就是为了函数式编程而生，@FunctionalInterface定义在接口上，检查此接口是否只存在一个方法。**



#### lambda表达式语法：

1.单行无返回值、单行有返回值

（）-> ...方法代码；

2.多行有返回值

（）->{

....

....

return 返回值语句；

}；

```java
interface IMessage{
     int add(int x,int y);
}
IMessage message=(x,y)->x+y;
message.add(10,20);
```





### 方法引用

