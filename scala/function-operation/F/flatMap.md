# flatMap

## definition

```scala

def flatMap[B](f: (A) ⇒ GenTraversableOnce[B]): TraversableOnce[B]

```

将 A 展开 => 集合B 。 最后返回一个整体的集合B。

## demo

```scala

val donuts1: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

val donuts2: Seq[String] = Seq("Vanilla Donut", "Glazed Donut")

val listDonuts: List[Seq[String]] = List(donuts1, donuts2)

// return a single list of donut using the flatMap function

val listDonutsFromFlatMap: List[String] = listDonuts.flatMap(seq => seq)

// 二者等价
val listDonutsFromFlatMap: List[String] = listDonuts.flatMap(_.toSeq)



```
