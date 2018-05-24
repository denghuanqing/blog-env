---
title: java8新特性01：lambda表达式
date: 2018-05-24 23:42:04
categories: java基础
---
&nbsp; &nbsp;java8 也在项目中用过，但是并没有用到什么新特性，对于code来说只是更换了版本号，并没有了解过底层的变化，这次准备系统的学习一下java8的新特性。
<!-- more -->
#### 语法基础 M -> N
左边M 是参数，参数若是有多个，需要用小括号。
右边N是方法体，方法体里面若是有多条语句，需要加大括号。
lambda表达式需要函数式接口支持
函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。抽象方法的实现可以用lambda表达式来调用。看下面的例子：
传统的线程实现方式
```
Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("线程例子演示");
            }
        };

        new Thread(runnable).run();
```
我们采用lambda表达式之后的写法：
```
  //这里 表达式的左边是小括号表示Runnable接口中的抽象方法没有参数，表达式的右边是方法体。
  // 整个表达式意思就是new了一个Runnable接口的实现类，并且覆盖了run方法。
  Runnable runnable = () -> System.out.println("线程例子演示" + Thread.currentThread().getName());

 new Thread(runnable).start();
```
除此之外，lambda表达式还有**方法引用** **构造函数引用**写法。方法引用有几种方式：类::静态方法名 类::实例方法名 对象::实例方法名 ，类::new
```
// 所有的方法引用要求：引用方法的参数，返回值和函数接口抽象方法的参数，返回值要一致。（类似于重写的概念）
Consumer<Boolean> consume= System.out::println;
consume.accept(true);

// 由于Supplier中的抽象方法没有参数，所以调用的是String的无参构造器
Supplier<String> ss= String::new;
System.out.println(ss.get());

// 由于Function中的抽象方法有参数，所以调用的是String的带参构造器
Function<String,String> sss=String::new;
System.out.println(sss.apply("java niubi"));
```
#### 函数接口（Function Interface）
lambda表达式依赖于函数接口，它返回的对象就是等同于new 一个接口的实现类，复写其抽象方法。并且支持自定义方法体内容。减少了代码量。
- 自定义函数接口
```
@FunctionalInterface
public interface Calcator {
    int calc(int a,int b);
}

 Calcator addCalcator=(a,b)->{
            return a+b;
        };

        Calcator reduceCalcator=(a,b)->{
            return a-b;
        };

        System.out.println(addCalcator.calc(10,5));
        System.out.println(reduceCalcator.calc(10,5));
```
- java7已经支持的函数接口
    - java.lang.Runnable
    - java.util.concurrent.Callable
    - java.security.PrivilegedAction
    - java.util.Comparator
    - java.io.FileFilter
    - java.nio.file.PathMatcher
    - java.lang.reflect.InvocationHandler
    - java.beans.PropertyChangeListener
    - java.awt.event.ActionListener
    - javax.swing.event.ChangeListener
- java8新增函数接口
  `java.util.function `包中定义了几组类型的函数式接口以及针对基本数据类型的子接口
    - Predicate -- 断言：传入一个参数，返回一个bool结果， 方法为boolean test(T t)
    - Consumer -- 消费：传入一个参数，无返回值，纯消费。 方法为void accept(T t)
    - Function -- 函数：传入一个参数，返回一个结果，方法为R apply(T t)
    - Supplier -- 供给：无参数传入，返回一个结果，方法为T get()
    - UnaryOperator -- 一元操作符， 继承Function,传入参数的类型和返回类型相同。
    - BinaryOperator -- 二元操作符， 传入的两个参数的类型和返回类型相同， 继承BiFunction

#### 常见的应用场景

- 遍历集合(Iterable接口)
```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
// 实质上调用了Consumer接口的accept方法
numbers.forEach(x -> System.out.println(x));
```
- 代替Runnable实现线程
- 待补充...

