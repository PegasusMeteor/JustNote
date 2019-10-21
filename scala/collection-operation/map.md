# 集合的Map操作

<!-- TOC -->

- [集合的Map操作](#%e9%9b%86%e5%90%88%e7%9a%84map%e6%93%8d%e4%bd%9c)
  - [Demo1 过程式实现](#demo1-%e8%bf%87%e7%a8%8b%e5%bc%8f%e5%ae%9e%e7%8e%b0)
  - [集合元素map映射操作](#%e9%9b%86%e5%90%88%e5%85%83%e7%b4%a0map%e6%98%a0%e5%b0%84%e6%93%8d%e4%bd%9c)
  - [Demo2 函数式实现](#demo2-%e5%87%bd%e6%95%b0%e5%bc%8f%e5%ae%9e%e7%8e%b0)
  - [Demo3 编程实现map操作](#demo3-%e7%bc%96%e7%a8%8b%e5%ae%9e%e7%8e%b0map%e6%93%8d%e4%bd%9c)

<!-- /TOC -->

## Demo1 过程式实现

给定一个 `List(1, 2, 3)` 将其中的每个元素都 乘以 2 ，并返回一个新的集合。  

```scala
object MapOperateDemo1 {

  def main(args: Array[String]): Unit = {
    val list1 = List(1, 2, 3)
    var list2 = List[Int]()
    for (item <- list1) {
      list2 = list2 :+ item * 2
    }
    println("list2:" + list2)
  }

}
```

上面的Demo有几个问题:

1. 不够简洁、高效
2. 没有函数式编程
3. 不利于处理复杂的数据处理业务

由此我们引出下面的map映射操作。

## 集合元素map映射操作

将集合中的每一个元素通过指定功能(**函数**) 映射（**转换**）成新的结果集。  

以 `Seq` 为例，可以看到它的map方法定义。

```scala
def map[B](f: (A) ⇒ B): Seq[B]
    [use case]
    Builds a new collection by applying a function to all elements of this sequence.

    B           the element type of the returned collection.

    f           the function to apply to each element.

    returns     a new sequence resulting from applying the given function f to each element of this sequence and collecting the results.
```

## Demo2 函数式实现

```scala
object MapOperateDemo2 {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3)

    val list2 = list.map(multiple)

    println(list2)

    // 简化写法
    val list3 = list.map(_ * 2)

    println(list3)

    val list4 = list.map(n => n * 2)

    println(list4)

  }

  def multiple(n: Int): Int = {
    println("被调用") //调用三次
    n * 2
  }
}

```

1. 将list中的元素全部遍历出来
2. 将遍历出来的元素传递给multiple
3. 将得到的值,放入到一个新的集合并返回

## Demo3 编程实现map操作

```scala
object MapOperateDemo2 {
  def main(args: Array[String]): Unit = {

    // 模拟 map 的实现机制
    val myList1 = DemoList()
    val myList2 = myList1.map(_ * 2)

    println(myList2)
  }
}

class DemoList {
  val list1 = List(1, 4, 5, 6)

  var list2 = List[Int]()

  def map(f: Int => Int): List[Int] = {
    // 遍历这个集合
    for (elem <- list1) {
      list2 = list2 :+ f(elem)
    }

    list2
  }

}

object DemoList {
  def apply(): DemoList = new DemoList()
}
```
