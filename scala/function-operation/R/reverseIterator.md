# reverseIterator

## definition

The reverseIterator method returns an iterator which you can use to traverse the elements of a collection in reversed order.

简言之，就是返回了一个迭代器，可以直接进行遍历。而不是像 reverse那样返回一个逆序的集合。

## demo

```scala
val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

val reverseIterator: Iterator[String] = donuts.reverseIterator
reverseIterator.foreach(donut => println(s"donut = $donut"))

```

输出的内容如下所示

```scala
scala> reverseIterator.foreach(donut => println(s"donut = $donut"))
donut = Glazed Donut
donut = Strawberry Donut
donut = Plain Donut
```
