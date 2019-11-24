# groupBy

## definition

```scala
groupBy[K](f: (A) ⇒ K): immutable.Map[K, Repr]
```

## demo

```scala

val donuts: Seq[String] = Seq("Plain Donut", "Plain Donut2","Strawberry Donut", "Glazed Donut")

// group elements in a sequence using the groupBy function
val donutsGroup: Map[Char, Seq[String]] = donuts.groupBy(_.charAt(0))


```

按照首字母进行分组,分组的结果需要梳理清楚。

输出结果为

```scala

scala> val donutsGroup: Map[Char, Seq[String]] = donuts.groupBy(_.charAt(0))
donutsGroup: Map[Char,Seq[String]] = Map(S -> List(Strawberry Donut), G -> List(Glazed Donut), P -> List(Plain Donut, Plain Donut2))

```

## demo1

```scala

case class Donut(name: String, price: Double)

val donuts: Seq[Donut] = Seq(Donut("Plain Donut", 1.5), Donut("Strawberry Donut", 2.0), Donut("Glazed Donut", 2.5))

// group case classes donut objects by the name property
val donutsGroup:Map[String,Seq[Donut]] = donuts.groupBy(_.name)


```

输出结果为

```scala

scala> val donutsGroup:Map[String,Seq[Donut]] = donuts.groupBy(_.name)
donutsGroup: Map[String,Seq[Donut]] = Map(Glazed Donut -> List(Donut(Glazed Donut,2.5)), Plain Donut -> List(Donut(Plain Donut,1.5)), Strawberry Donut -> List(Donut(Strawberry Donut,2.0)))


```
