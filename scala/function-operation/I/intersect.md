# intersect

## definition

顾明思议，相交。求出两个集合的交集。

```scala
def intersect(that: GenSet[A]): Repr
```

## demo

```scala

val donuts1: Set[String] = Set("Plain Donut", "Strawberry Donut", "Glazed Donut")
val donuts2: Set[String] = Set("Plain Donut", "Chocolate Donut", "Vanilla Donut")

// find the common elements between two Sets using intersect function
val inter =  donuts1 intersect donuts2

// find the common elements between two Sets using & function
val common = donuts1 & donuts2

```

两个集合的交集，那就只有一个,所以输出结果如下。

```scala
scala> val inter =  donuts1 intersect donuts2
inter: scala.collection.immutable.Set[String] = Set(Plain Donut)

scala> val common = donuts1 & donuts2
common: scala.collection.immutable.Set[String] = Set(Plain Donut)
```
