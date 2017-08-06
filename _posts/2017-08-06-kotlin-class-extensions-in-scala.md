---
layout: post
title: Scala模拟Kotlin的类扩展（Extensions）功能
date: 2017-08-06 10:52:20 +08:00
author: "izumo"
tags:
    - Scala
    - Kotlin
---

之前学习了一下Kotlin，发现Kotlin的类扩展（Extensions）功能还是十分好用的，要是在Scala下也可以这么使用就太好了。于是本人看了下相关内容，用Scala的隐式（implicit）成功模拟了Kotlin的类扩展功能。

## 简单实现

假设要给`String`类添加`getSecondChar`和`repeat`方法。我们可以创建一个`StringWrapper`类， 相当于对`String`类的扩展：

    class StringWrapper(val s: String) {
      def getSecondChar: Option[Char] = if (s.length >= 2) Some(s.charAt(1)) else None
      def repeat: String = s * 2
    }

然后在需要扩展类的地方添加一个隐式函数`toStringWrapper`：

    implicit def toStringWrapper(s: String): StringWrapper = new StringWrapper(s)

这样就完成了`String`类的扩展。我们来试一试：

    val ss: String = "hello"
    
    println(ss.getSecondChar) // Some(e)
    println(ss.repeat) // hellohello

## 泛化实现

上面的方法已经可以简单地实现类扩展，但如果需要扩展多个类，把这些扩展类全部放在其他代码下面未免显得有些臃肿。我们可以把这些扩展类提取出来，放在一个单独的文件下。

首先创建一个`Wrapper`特质（trait），所有的扩展类都需要混入这个特质：

    trait Wrapper[A] {
      val it: A
      def wrap(data: A): Wrapper[A]
    }

接下来在`Wrapper`对象内创建扩展类：

    object Wrapper {
    
      // 共同的隐式函数
      implicit def toWrapper[A: Wrapper](data: A): Wrapper[A] = implicitly[Wrapper[A]].wrap(data)
    
      // 扩展String类
      implicit class StringWrapper(val it: String) extends Wrapper[String] {
        override def wrap(data: String): Wrapper[String] = new StringWrapper(data)
        def getSecondChar: Option[Char] = if (it.length >= 2) Some(it.charAt(1)) else None
        def repeat: String = it * 2
      }
    
      // 扩展Int类
      implicit class IntWrapper(val it: Int) extends Wrapper[Int] {
        override def wrap(data: Int): Wrapper[Int] = new IntWrapper(data)
        def add(other: Int): Int = it + other
      }
    
    }

然后在需要扩展类的地方导入`Wrapper`对象的全部内容：

    import Wrapper._

这样就完成了`String`类和`Int`类的扩展。我们来试一试：

    val s: String = "abc"
    val i: Int = 3
    
    println(s.repeat) // abcabc
    println(i.add(4)) // 7

我们还可以给泛型类添加扩展。在`Wrapper`对象内添加`DoubleListWrapper`类和`VectorWrapper`类：

    // 扩展List[Double]
    implicit class DoubleListWrapper(val it: List[Double]) extends Wrapper[List[Double]] {
      override def wrap(data: List[Double]): Wrapper[List[Double]] = new DoubleListWrapper(data)
      def addEach(d: Double): List[Double] = it.map(_ + d)
    }
    
    // 扩展Vector[A]
    implicit class VectorWrapper[A](val it: Vector[A]) extends Wrapper[Vector[A]] {
      override def wrap(data: Vector[A]): Wrapper[Vector[A]] = new VectorWrapper[A](data)
      def takeFirstTwo: Option[Vector[A]] = if (it.length >= 2) Some(it.take(2)) else None
    }

我们来试一试：

    val dl: List[Double] = List(1.1, 2.2, 3.3)
    val sv: Vector[String] = Vector("hello", "world", "!")
    val fv: Vector[Float] = Vector(4f, 5f, 6f)
    
    println(dl.addEach(10.0)) // List(11.1, 12.2, 13.3)
    println(sv.takeFirstTwo) // Some(Vector(hello, world))
    println(fv.takeFirstTwo) // Some(Vector(4.0, 5.0))

## 总结

本文只是对Kotlin的类扩展功能在Scala下进行了一个简单的模拟，并没有实现类扩展的所有功能，比如对原有类方法的重载。实在需要方法重载的话，可以给`Wrapper`特质添加一个`unit`方法：

    def unit: Wrapper[A]

然后在具体的扩展类里显示地把类`A`转换成类`AWrapper`，并在扩展类下创建同名方法以实现方法的“重载”。这样做还可以防止对原有的类造成破坏。

