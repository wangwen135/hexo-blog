---
title: java8-Lambda表达式&函数接口&方法引用
date: 2024-05-18 23:41
tags: 
  - java
categories:
  - [java]
---

## Lambda表达式

java 8 的新特性，是一种语法糖，简化了只有一个方法的匿名内部类的编写，让编码更方便，代码更简洁。

> 只有一个方法的接口（又叫函数式接口）在写匿名内部类时看起来很啰嗦，真正有用的只是方法实现中的那一段代码。通常情况下我们使用匿名内部类是想将功能作为参数传递给另外的方法，例如点击按钮时应该采取的措施。Lambda表达式使我们能够执行此操作，将功能视为方法参数，或将代码视为数据。


个人理解：主要作用是为了更方便的创建 **函数接口** 的匿名内部类的**实例**，减少啰嗦的代码。

### 基本语法
主要是加了箭头符号 '->'  称为Lambda操作符或箭头操作符
```
(参数) -> {函数体}
```
左边是参数，中间是箭头符号，右边是方法体

这里以jdk内置Consumer 函数式接口示例：

**原来的匿名内部类写法**
```
Consumer<String> c = new Consumer<String>() {
    @Override
    public void accept(String str) {
        System.out.println("msg=" + str);
    }
};
```
**新的Lambda表达式写法**
```
Consumer<String> c1 = (String str) -> {
    System.out.println("msg=" + str);
};
```
可以看到代码简洁了很多

再比Runnable是一个函数接口，创建线程可以这样写
```
Thread t = new Thread(() -> {
    System.out.print("Lambda!");
    System.out.print("Do Something...");
});
t.run();
```

#### 更简化的写法

##### 参数可以声明类型，也可以根据类型推断而省略
如上面的c1可以省略参数类型String
```
Consumer<String> c1 = (str) -> {
    System.out.println("msg=" + str);
};
```
> 但是如果有两个参数不能只写一个参数的类型，要么都写类型，要么都不写类型

##### 只有一个参数时可以省略参数的小括号
```
Consumer<String> c2 = str -> {
    System.out.println("msg=" + str);
};
```

##### 方法体只有一条语句时花括号和语句的分号都可以省略
```
Consumer<String> c3 = str -> System.out.println(str);
```

##### 只有一条返回语句时return可以不写
如返回一个String
```
Supplier<String> s1 = () -> {
    return "Lambad!";
};
```
可以写成
```
Supplier<String> s2 = () -> "Lambad!";
```
返回对象可以写成
```
Supplier<User> s3 = () -> new User("张三", 20);
```

##### 使用方法引用时参数和箭头符号都可以省略
前提是：接口定义的参数 和 方法的参数匹配  

如：
```
Consumer<String> c4 = System.out::println;
```
> accept.accept(String t) 的参数与 System.out.println(String x)的参数匹配

**构造器引用**
```
Supplier<User> s4 = User::new;
```

**静态方法引用**  
User对象中有一个叫getInstance的静态方法
```
Supplier<User> s5 = User::getInstance;
```

**多参数+返回**  
传入两个int值返回较大的一个
```
BiFunction<Integer, Integer, Integer> bf = (x, y) -> x > y ? x : y;
# 可以写成
BiFunction<Integer, Integer, Integer> bf1 = Math::max;
```


**原则就是让编译器能识别，不产生歧义！**


### Lambda表达式序列化问题
一个Lambda能否序列化， 要以它捕获的参数以及target type能否序列化为准。lambda表达式是可以序列化的，但是就像内部类一样，强烈建议不要对lambda表达式进行序列化。

*ps：经测试 使用方法引用时不能序列化*

> 反对序列化内部类(包括局部类和匿名类)。当Java编译器编译某些构造时，比如内部类，它会创建合成构造；这些是在源代码中没有相应构造的类、方法、字段和其他构造。合成构造使Java编译器能够在不更改JVM的情况下实现新的Java语言特性。然而，合成构造可能因不同的Java编译器实现而不同，这意味着.class文件也可能因不同的实现而不同。因此，如果您序列化一个内部类，然后用不同的JRE实现反序列化它，您可能会有兼容性问题。有关编译内部类时生成的合成构造的更多信息，请参阅获取方法参数名称一节中的隐式和合成参数一节。
> 






-----

-----




## 接口默认方法和静态方法
从Java 8 开始，接口中引入了 **缺省方法**（default method） 和 **静态方法** （static method） 的特性。

**注意：接口中的方法锈蚀符号只能是 {抽象、默认、静态} 中的一种**

### 默认方法
默认方法就是接口可以有实现方法，不需要实现类主动去实现默认会自动继承，当接口中的默认方法不满足要求时实现类可以对其进行重写，调用时重写的方法优先于默认方法被调用。

默认方法用default关键字修饰，默认是public权限

添加此功能是为了实现向后兼容，默认方法不会破坏原有的接口实现还便于给程序添加新特性。比如某一天我们需要给线上某个接口增加一个方法，在 Java 8 之前，我们需要去改每一个实现类，就算这个实现根本用不到这个功能也得加一个空实现，否则编译不了，改动起来非常麻烦。Java 8 之后就只需要在接口上增加一个默认方法就可以了。

> java不支持多继承但是可以实现多个接口，有了默认方法之后我们可以将方法的相关逻辑写到接口中这样也能解决部分多继承的问题。

#### 继承特性
```
interface A {
     // 默认方法
    default void say() {
        System.out.println("hello");
    }
}

class C1 implements A {

}

public static void main(String[] args) {
    C1 c1 = new C1();
    // 自动继承
    c1.say();
}

```
输出：  
```
hello
```


##### 子类方法优先
C1重写say方法
```
class C1 implements A {
    @Override
    public void say() {
        System.out.println("C1 hello");
    }
}
```
输出：  
```
C1 hello
```

**子类的子类会优先继承父类的方法，父类没有重写时才会使用接口中的方法**

C2类继承C1类
```
class C2 extends C1{
    
}

public static void main(String[] args) {
   C2 c2 = new C2();
   c2.say();
}
```
输出：
```
C1 hello
```


##### 实现多个接口中有相同的方法时需指定
如子类实现两个接口，两个接口中有相同的方法（参数和方法名相同），实现类需要重写该，否则编译器不知道该调用哪一个
```
interface A {
    default void say() {
        System.out.println("[interface A] say hello");
    }
}

interface B {
    default void say() {
        System.out.println("[interface B] say hello");
    }
}

class C1 implements A, B {
    @Override
    public void say() {
        System.out.println("需要重写say方法，否则编译器不知道是调用 A.say 还是 B.say");
    }
}
```

##### 显示引用接口的默认实现
通过使用super，可以显式的引用被继承接口的默认实现，语法如下：InterfaceName.super.methodName()

```
class C1 implements A, B {
    @Override
    public void say() {
        A.super.say();
    }
}
```
输出：
```
[interface A] say hello
```



### 静态方法
接口中的静态方法和类中定义的静态方法一样，不属于特定对象，所以它们不是实现接口的api的一部分，必须使用InterfaceName.staticMethod来调用它们。  
> 接口中的所有方法声明（包括静态方法）都是隐式的public，因此可以省略public修饰符

```
interface A {
    static String STR = "静态字段可以被继承";
    static void sayA() {
        System.out.println("static sayA");
    }
}

class C1 implements A {
}

//main
public static void main(String[] args) {
    // 通过接口名称调用接口的静态方法
    A.sayA();

    // 不能通过子类实例调用 new C1().sayA()
    // 也不能通过子类调用 C1.sayA()
    
    //可以通过子类实例获取静态字段值
    System.out.println(new C1().STR);
}
```
**实现接口的类或者子接口不会继承接口中的静态方法**，只能是静态方法所属的接口来调用。但是接口中静态字段可以被继承（默认用public static final修饰）。


> 在java8中很多接口中都使用默认方法和静态方法进行了增强，如：Comparator，接口静态方法适合于提供实用方法，例如空检查、集合排序等。





-----
-----





## 函数式接口

**只有一个抽象方法的接口**

可以有多个默认方法或者静态方法等

接口默认继承java.lang.Object，所以如果接口显示声明覆盖了Object类中的public方法，那么 也不算抽象方法，因为任何接口的实现都会从其父类Object或其它地方获得这些方法的实现。

如：
```
@FunctionalInterface
public interface FunctionalInterfaceT1<T> {

    // 抽象方法
    void doSomething(T t);

    // Object类中的方法
    boolean equals(Object obj);

    // 默认方法
    default void printT(T t) {
        System.out.println(t);
    }

    // 默认方法2
    default void printT2(T t) {
        System.out.println(t);
    }

    // 静态方法
    public static void print() {
        System.out.println("FunctionalInterfaceT1.print");
    }
}
```
### @FunctionalInterface 注解
该注解不是必须的，如果一个接口符合"函数式接口"定义，那么加不加该注解都没有影响。加上该注解能够更好地让编译器进行检查。如果编写的不是函数式接口，但是加上了@FunctionInterface，那么编译器会报错。

### 默认方法不能覆盖Object中的方法
接口不能实现Object的toString、equals、hashCode等方法，因为接口多继承如果多个接口都实现了equals方法会不知道调用哪一个

编译过不了



### 内置的核心函数式接口

java8中内置的函数式接口位于包：**java.util.function**

#### 4大常用的函数式接口

##### Consumer<T>
**消费型 接口**

方法：void accept(T t)   
用来消费 T 类型的对象

##### Supplier<T>
**供给型 接口**

方法： T get()  
提供 T 类型的对象


##### Function<T,R>
**函数型 接口**

方法： R apply(T t)  
对类型为 T 的对象进行操作返回 R 类型的对象

##### Predicate<T>
**断言型 接口**

方法: boolean test(T t)  
判断 T 类型的对象是否满足某些约束


#### 其他类型的函数式接口
也都在 java.util.function 包下，大体上都差不多，记住上面常用的4个就行了

##### BiFunction<T, U, R>   
参数为 T，U，返回值为 R  
方法为 R apply(T t, U u)

##### UnaryOperator<T> (Function子接口)  
参数为 T，对参数为T的对象进行一元操作，并返回T类型结果  
方法为 T apply(T t)

##### BinaryOperator<T> (BiFunction子接口)  
参数为T，对参数为T得对象进行二元操作，并返回T类型得结果  
方法为 T apply（T t1， T t2）

##### BiConsumer(T, U)   
参数为 T，U 无返回值  
方法为 void accept(T t, U u)

##### BiPredicate<T, U>
参数为 T，U 返回boolean值
方法为： boolean test(T t, U u)

##### ToIntFunction<T>、ToLongFunction<T>、ToDoubleFunction<T>  
参数类型为T，返回值分别为int，long，double  
分别计算int，long，double的函数
方法为：int applyAsInt(T value)；long applyAsLong(T t, U u)

##### IntFunction<R>、LongFunction<R>、DoubleFunction<R>  
参数分别为int，long，double，返回值为R。
方法为： R apply(int value); R apply(long value);





---

---





## 方法引用

方法引用是一个新特性，方法类似指针一样可以被直接引用。  
新的操作符"::"（两个冒号）用来引用 **类** 或者 **实例** 的方法。

方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

### 静态方法引用
```
BiFunction<Integer, Integer, Integer> bf = Math::max;
System.out.println(bf.apply(1, 3));
```

### 实例方法引用
```
Consumer<String> c = System.out::println;
c.accept("hello");
```

### 任意实例引用
```
String[] stringArray = { "Barbara", "James", "Mary", "John", "Patricia", "Robert", "Michael", "Linda" };
Arrays.sort(stringArray, String::compareToIgnoreCase);
Arrays.stream(stringArray).forEach(System.out::println);
```
可以引用任意的一个类型的实例。等价的lambda表达式的参数列表为(String a, String b)，方法引用会调用a.compareToIgnoreCase(b)。

### 构造函数引用
```
Supplier<User> s = User::new;
```
对构造函数的引用类似对静态方法的引用，只不过方法名是new。 一个类有多个构造函数， 会根据target type选择最合适的构造函数。



