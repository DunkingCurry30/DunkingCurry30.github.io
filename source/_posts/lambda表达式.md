---
title: Java中的lambda表达式
date: 2022/7/12 20:00:00
tags: 
  - java
categories: 
  - Java后端	
---

> 前言：异步的文章中，线程的创建用到了 `lambda` 表达式，作为 JDK 8的新特性同时也是**函数式编程**的代表实践，趁此机会学习一下

# 一、函数式编程思想概述

在数学中，函数就是有输入量、输出量的一套计算方案，也就是“拿数据做操作”

- [面向对象](https://so.csdn.net/so/search?q=面向对象&spm=1001.2101.3001.7020)思想强调“必须通过对象的形式来做事情”

- 函数式思想则强调尽量忽略面向对象的复杂语句：“强调做什么，而不是以什么形式去做”

而接下来要讲的Lambda表达式就是**函数式思想**的体现



# 二、Lambda 表达式基础



## 1. 为什么要使用 Lambda 表达式

这里还是以创建执行一个线程举例，有以下三种方式

**方式1**，使用传统方式创建新对象：

- 定义一个类MyRunnable接口，重写run方法
- 创建MyRunnable类的对象
- 创建Thread类对象，把MyRunnable的对象作为构造参数传递
- 启动线程

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("执行一个新线程");
    }
}
```

```java
MyRunnable myRunnable = new MyRunnable();
Thread thread = new Thread(myRunnable);
thread.start();
```

**方式2**，在方式1的基础上做改进，使用**匿名内部类**：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("执行一个新线程");
    }
}).start();
```

**方式3**，使用lambda做进一步简化：

```java
new Thread(() -> {
    System.out.println("执行一个新线程");
}).start();
```

通过以上三种方式，相信你已经看出来了，lambda表达式能使代码轻量化，极大地优化代码结构，它只关心入参和方法体（即做事的逻辑），其他的（返回类型，参数类型，方法名）都不关心。



## 2. Lambda 表达式的标准格式

###  Lambda表达式的代码分析 

- 格式：`(形式参数...) -> {代码块}`  

- `()`：包含方法的入参，里面若没有内容，可以看成是方法形式参数为空；若只有一个入参，可以省略 `()`
- `->`：用箭头指向后面要做的事情
- `{}`：包含一段代码，我们称之为代码块，可以看成是方法体中的内容；若只有一行内容，可以省略 `{}` 、`;` ，有返回值可以省略 `return`  



### Lambda表达式使用示例

需先定义一个接口，包含一个加法的抽象方法：

```java
public interface Operator {
    int add(int x,int y);
}
```

定义外部方法调用接口：

```java
private void addFunc(Operator operator){
    int sum = operator.add(10,20);
    System.out.println(sum);
}
```

调用外部方法，以lambda表达式替代传入的接口对象

```java
//一般写法
add((x, y) -> {
    return x + y;
});
//省略return和{}
addFunc((x, y) -> x+y);
//一个参数可省略()，一行方法可省略{}和;
print(s -> System.out.println(s))  
```

 

###  Lambda表达式的注意事项

1. 使用Lambda必须要有接口，并且要求接口中**有且仅有一个**抽象的方法

2. 必须有**上下文环境**，才能推导出Lambda对应的接口

- 根据局部变量的赋值得知Lambda对应的接口：

```java
Runnable r =() ->System.out.println(“Lambda表达式”);
```


- 根据调用方法的参数得知Lambda对应的接口：

```java
new Thread(()->System.out.println(“Lambda表达式”)).start();
```



## 3. Lambda表达式和匿名内部类的区别

### 所需类型不同

- 匿名内部类：可以是接口，也可以是抽象类，还可以是具体类
- Lambda表达式：只能是接口

### 使用限制不同

- 如果接口中有且仅有一个抽象方法，可以使用Lambda表达式，也可以使用匿名内部类

- 如果接口中多于一个抽象方法，只能使用匿名内部类，而不能使用Lambda表达式

### 实现原理不同

- 匿名内部类：编译之后，产生一个单独的.class字节码文件
- Lambda表达式：编译之后，没有一个单独的.class字节码文件，对应的字节码会在运行的时候动态生成



>参考文章： [Java 函数式编程](https://blog.csdn.net/hua226/article/details/124409889?ops_request_misc=%7B%22request%5Fid%22%3A%22165742542816780366560942%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=165742542816780366560942&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-124409889-null-null.142^v32^pc_rank_34,185^v2^control&utm_term=函数式编程&spm=1018.2226.3001.4187) 