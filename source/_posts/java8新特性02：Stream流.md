---
title: java8新特性02：Stream流
date: 2018-05-26 14:47:32
categories: java基础
---
&nbsp; &nbsp;Stream流自己不存储元素，不会改变源数据，操作是延迟执行（懒汉式）即中间操作不会执行，除非触发终止操作。  创建stream->中间操作->终止操作
<!-- more -->
# 创建Stream
- Collection集合的stream()或者parallelStream(0
- Arrays中的静态方法stream()获取数组流
- Stream的of()方法获取流
- 创建无限流
```
List<String> list = Arrays.asList("java", "python", "c");

        Stream<String> stream = list.stream();
        Stream<String> stringStream = list.parallelStream();

        stream.forEach((m)-> {
            System.out.println(m+" ::线程名:: "+Thread.currentThread().getName());
        });

        stringStream.forEach((m)-> {
            // ? 并行流为什么是单线程
            System.out.println(m+" ::线程名:: "+Thread.currentThread().getName());
        });

        IntStream stream1 = Arrays.stream(new int[5]);

        Stream<String> ss = Stream.of("ss", "dd", "mm");
        
        Stream<Integer> iterate = Stream.iterate(0, (x) -> x + 2);
        iterate.limit(10).forEach((m) -> System.out.println(m));

        Stream<Double> generate = Stream.generate(() -> {
            return Math.random();
        });
        generate.limit(5).forEach((m) -> System.out.println(m));
```
# 中间操作

## 筛选与切片

- filter筛选
- limit 截断流
- skip(n) 跳过前N个元素 和 limit 互补
- distinct 去重 通过流元素的hashCode()和equals()去除重复元素
```
List<String> list = Arrays.asList("java", "python", "c");
        Stream<String> stream = list.stream();
        Stream<String> result = stream.filter((m) -> {
            if (m.equals("java")) {
                return true;
            } else {
                return false;
            }
        });
        result.forEach((n) -> System.out.println(n));

        Stream<String> skip = stream.skip(2);
        skip.forEach((n) -> System.out.println(n));
```

## 映射
- map 接收一个函数接口参数，把元素流中的每一个元素通过Function的操作，返回一个新的集合
- flatmap 瘦身嵌套流，返回每个嵌套流的最底层集合
区别：map就是把流加在新流中，flatmap就是把流中的元素加在新流中。map 对 flatMap 相当于 集合add对addAll

## 排序
- sorted自然排序
- sorted(Comparator<? super T> comparator) 自定义排序

# 终止操作（流实际执行阶段）

## 匹配&查找
- allMatch 检查是否匹配所有 返回boolean
- anyMatch是否至少匹配一个
- noneMatch 是否都不匹配
- findFirst 返回第一个元素
- findAny 并不是纯随机？需要结合并行流来使用？
- count 计数 返回long类型
- max (Comparator<? super T> comparator)   第一个
- min(Comparator<? super T> comparator) max&min都要求传入Compaertor 最后一个

## 规约
- reduce()求和？ 累计结果集 累加，累连接
```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> reduce1 = integers.stream().reduce((a, b) -> a + b);
Optional<Integer> reduce = integers.stream().reduce(Integer::sum);

List<String> strings = Arrays.asList("java", "php", "python");
Optional<String> reduce2 = strings.stream().reduce(String::concat);
System.out.println(reduce2.get());
```

## 收集
- collect() 把流转换为其他数据格式，比如list转换为map。collectors提供很多常用的实现(具体看源码注释)，eg：toList,toMap...

#  并行流与串行流
- java7提供了fork/join框架，java8直接集成到了并行流中，看下面代码
```
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 0);
integers.parallelStream().forEach(M -> {
	System.out.println(Thread.currentThread().getName() + "::" + M);
});
```
输出：
```
main::7
main::6
main::9
main::0
main::8
ForkJoinPool.commonPool-worker-1::3
main::2
ForkJoinPool.commonPool-worker-1::5
main::1
ForkJoinPool.commonPool-worker-1::4
```
