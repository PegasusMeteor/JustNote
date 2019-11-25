# unzip

## definition

The unzip method will unzip and un-merge a collection consisting of element pairs or Tuple2 into two separate collections.

其实也就是zip的逆操作

```scala
def unzip[A1, A2](implicit asPair: (A) ⇒ (A1, A2)): (CC[A1], CC[A2])
```

## demo

```scala
scala> val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")
donuts: Seq[String] = List(Plain Donut, Strawberry Donut, Glazed Donut)

scala> val donutPrices = Seq[Double](1.5, 2.0, 2.5)
donutPrices: Seq[Double] = List(1.5, 2.0, 2.5)

scala> val zippedDonutsAndPrices: Seq[(String, Double)] = donuts zip donutPrices
zippedDonutsAndPrices: Seq[(String, Double)] = List((Plain Donut,1.5), (Strawberry Donut,2.0), (Glazed Donut,2.5))

scala> val unzipped: (Seq[String], Seq[Double]) = zippedDonutsAndPrices.unzip
unzipped: (Seq[String], Seq[Double]) = (List(Plain Donut, Strawberry Donut, Glazed Donut),List(1.5, 2.0, 2.5))


```
