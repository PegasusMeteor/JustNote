# 集合的 flatmap操作

<!-- TOC -->

- [集合的 flatmap操作](#%e9%9b%86%e5%90%88%e7%9a%84-flatmap%e6%93%8d%e4%bd%9c)
  - [函数的定义](#%e5%87%bd%e6%95%b0%e7%9a%84%e5%ae%9a%e4%b9%89)
  - [作用](#%e4%bd%9c%e7%94%a8)

<!-- /TOC -->

## 函数的定义

将 二元函数 引用于集合中的函数

首先以 Seq 的flatmap函数定义来进行理解。

```scala

def flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): Seq[B]
[use case]
Builds a new collection by applying a function to all elements of this sequence and using the elements of the resulting collections.

B       the element type of the returned collection.

f       the function to apply to each element.

returns     a new sequence resulting from applying the given collection-valued function f to each element of this sequence and concatenating the results.

```

## 作用

将集合中每个**元素的子元素**映射(**转换**)到某个函数并返回新的集合。

```scala

object FlatMapDemo01 {

  def main(args: Array[String]): Unit = {

    // 对所有元素进行扁平化

    val list = List("Tom", "Jerry", "Jack")

    val list2 = list.flatMap(upper)

    println(list2)
  }

  def upper(string: String):String={
    string.toUpperCase
  }

}


```
