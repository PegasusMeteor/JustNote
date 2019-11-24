# 集合的filter操作

<!-- TOC -->

- [集合的filter操作](#%e9%9b%86%e5%90%88%e7%9a%84filter%e6%93%8d%e4%bd%9c)
  - [函数的定义](#%e5%87%bd%e6%95%b0%e7%9a%84%e5%ae%9a%e4%b9%89)
  - [作用](#%e4%bd%9c%e7%94%a8)

<!-- /TOC -->

## 函数的定义

首先以 Seq 的filter函数定义来进行理解。

```scala

def filter(p: (A) ⇒ Boolean): Seq[A]
  Selects all elements of this traversable collection which satisfy a predicate.

  p
  the predicate used to test elements.

  returns
  a new traversable collection consisting of all elements of this traversable collection that satisfy the given predicate p. The order of the elements is preserved.

```

## 作用

将集合中符合条件的 元素 筛选出来，放入到一个新的集合中，并返回这个新的集合。
这些操作，对原来的集合是没有影响的。

```scala
object FilterDemo1 {

  def main(args: Array[String]): Unit = {
    val list = List("Tom", "Aerry", "Jack").filter(startA)

    println(list)

  }

  def startA(string: String):Boolean ={
    string.startsWith("A")
  }

}
```

```scala

object FilterDemo2 {

  def main(args: Array[String]): Unit = {
    val list = List("Tom", "Aerry", "Jack").filter(_.startsWith("A"))

    println(list)
  }

}
```
