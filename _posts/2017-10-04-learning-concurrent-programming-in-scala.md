---
layout: post
title: Scala 并发编程笔记
date: 2017-10-04 12:28:55 +08:00
author: "izumo"
tags:
    - Scala
---

本文是对《Scala并发编程》一书的阅读笔记。

## Future

    Future[T] {
      => T
    }

用`Future[T].foreach(f: (T) => Unit)`设定结果正确返回之后要执行的操作；用`Future[T].failed.foreach(f: (T) => Unit)`设定结果返回失败时要执行的操作。

## Try

    Try[T] {
      => T
    }

`Try[T]`或者返回正确的值` Success[T]`，或者返回失败`Failure[Throwable]`

    a match {
      case Success(i) => println(i)
      case Failure(e) => println(e.getMessage)
    }

同理，用`Try[T].foreach(f: (T) => Unit)`设定结果正确返回之后要执行的操作；用`Try[T].failed.foreach(f: (T) => Unit)`设定结果返回失败时要执行的操作。

## Promise

    val t = Promise[T] // 设置一个空的Promise[T]对象
    // 以下两个操作只能执行一次，否则抛出异常
    p.success[T](f: => T) // 完善Promise[T]对象
    p.failure(t: => Throwable) // 完善Promise[T]对象，但是令其返回失败
    // 以下两个操作可以执行任意次，但只有第一次可以执行成功
    p.trySuccess[T](f: => T): Boolean // 尝试完善Promise[T]对象，返回是否执行成功
    p.tryFailure(t: => Throwable) // 尝试完善Promise[T]对象，但是令其返回失败，返回是否执行成功

使用`p.future`返回Future[T]对象，之后的操作同Future。
    
## Scala Async

    compile group: 'org.scala-lang.modules', name: 'scala-async_2.12', version: '0.9.7'

`async`用于开启一个新线程，相当于Future[T].apply(f:  => T)；`await`用于在线程内等待一个`Future[T]`的返回值T，不会阻塞基础线程（因为是在新线程内等待）。

    async {
      // do something
      await { // Future[T] }
      // do something
    }

## 数据并行集合

### 集合的继承顺序

+ `Seq`、`Map`、`Set`继承自`Iterable`

+ `ParSeq`、`ParMap`、`ParSet`继承自`ParIterable`

+ `Iterable`、`ParIterable`继承自`GenIterable`

+ `Seq`、`Map`、`Set`、`ParSeq`、`ParMap`、`ParSet`对应继承自`GenSeq`、`GenMap`、`GenSet`（例如，可以使用`GenSeq`同时操作`Seq`和`ParSeq`）

+ `GenSeq`、`GenMap`、`GenSet`继承自`GenIterable`

### 在JVM中度量性能

Java字节码在JVM中运行时，首先会使用解释模式。只有当JVM确定这些字节码被执行的次数足够多时，才会将其编译为机器码，在处理器中直接执行它们。

因此，在测试代码稳定性能时，应事先运行多次。

### 使用并行集合的注意事项

#### 非可并行化集合

并行集合使用`Splitter[T]`代表的分离器，并提供并行操作。`Splitter[T]`继承自`Iterator[T]`，同样拥有`hasNest`、`next`方法；同时提供`split`方法可以将分离器S分解为遍历分离器S部分内容的分离器序列。

    def split: Seq[Spliter[T]]

该方法允许多个独立的处理器遍历输入集合的各个组成部分。

许多Scala集合的操作都是可以并行化的，包括：`Array`、`ArrayBuffer`、`HashMap`、`HashSet`、`Range`、`Vector`。它们调用`.par`后，会创建一个新的并行集合，该集合会与原集合共享相同的基础数据集，无需复制任何元素，而且转换速度非常快。

其它的集合属于**非可并行化集合**，调用`.par`方法时，需要将元素复制到新集合中。

#### 非可并行化操作

有些操作天生具有顺序性，而且语义也不允许以并行的方式执行它们，如`foldLeft`（虽然并行集合存在这些方法，但这些方法并不能产生预期的效果）。此时应使用`aggregate`方法，可以减少并行操作的运行时间。

#### 并行操作副作用

在不使用同步机制的情况下，多个线程无法正确地修改共享内存的内容。

`foreach`或`for`操作无法获得正确的结果，如果是要进行计数操作，则应使用`count`方法（且性能更优）。

#### 不确定的并行操作

多线程程序会有不确定性，相同的输入，因其执行语句的次序，会有不同的输出结果。如`find`方法，语义上是返回第一个匹配成功的元素，但多线程中，集合片段执行的顺序不确定，因此返回的元素也不确定。

此时应使用`indexWhere`方法。

> 只要并行集合操作的操作符是**纯函数**，那么除`find`之外的其他并行集合操作就都是确定的。




