---
title: 《effective java》 读书笔记
date: 2018-05-26 17:21:02
categories: 读书笔记
---
优雅代码的第一步，读《effective java》
<!-- more -->
# 42条:慎用可变参数
今天遇到一个问题，String->字节数组->List ,发现没有很好的解决办法。看下面代码：
```
char[] chars = new char[]{'1','2','3'};
// 这里并没有返回我需要的List<char>
List<char[]> chars1 = Arrays.asList(chars);

Character[] chars2 = new Character[]{'1','2','3'};
List<Character> characters = Arrays.asList(chars2);
```
在Arrays.asList方法中T是参数类型，它必须为一个Object 类型，但是char是**基本类型**。所以不会返回正常的结果。直接打印会产生没有意义的字符串：@3e5d...
## 解决方案;
1.使用Apache Commons Lang吧，可能你的项目正在使用它，类似下面这样使用ArrayUtils.toObject：
2.手动for循环
3.Arrays.asList()方法并不支持基本类型（int,char,long等）的数据转换成list。只支持Object类型，即包装类型。
## 思考：
其实Arrays.asList方法底层就是可变参数的实现案例，书上说我们每次调用可变参数的方法时都会导致数组分配和初始化，所以会有一定的性能成本。想起来之前有个同事在组装发送微信模板消息的代码时自定了一个可变参数的方法。那么其实优化策略可以如下：
若是实际的业务 95%的调用会带三个或三个以下的参数，还有极少数的情况会带有可变参数，我们做成重载的方法以减少对可变参数的调用。
```
public void senMsg(){}
public void senMsg(int a){}
public void senMsg(int a,int b){}
public void senMsg(int a,int b,int c){}
// 以减少这个方法的调用次数，但是又不失灵活性
public void senMsg(int a,int b,int c,int ... array){}
```
测试了一下可变参数调用效率
```
int num = 500000000;
        Profiler profiler = new Profiler("one");
        profiler.start("A");
        for (int i = 0; i < num; i++) {
            senMsg(1, 2, 3);
        }

        profiler.start("B");
        for (int i = 0; i < num; i++) {
			// 5，6 作为可变参数传入
            senMsg(1, 2, 3, 5, 6);
        }
```
求和运算|1000| 5000|100000000
- | :-: | :-: | -: 
不带可变参数 |52.338 microseconds| 3.216 milliseconds|2.953
带可变参数 | 126.294| 15.760|4593.421
倍数|2.423 |4.90|1555.5


