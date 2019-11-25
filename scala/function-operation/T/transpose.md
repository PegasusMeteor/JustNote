# transpose

## definition

The transpose method will pair and overlay elements from another collections into a single collection.

可以理解为矩阵转置

```scala
def transpose[B](implicit asTraversable: (A) ⇒ GenTraversableOnce[B]): CC[CC[B]]
```

## demo

```scala
scala> val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")
donuts: Seq[String] = List(Plain Donut, Strawberry Donut, Glazed Donut)

scala> val prices: Seq[Double] = Seq(1.50, 2.0, 2.50)
prices: Seq[Double] = List(1.5, 2.0, 2.5)

scala> val donutList = List(donuts, prices)
donutList: List[Seq[Any]] = List(List(Plain Donut, Strawberry Donut, Glazed Donut), List(1.5, 2.0, 2.5))

scala> donutList.transpose
res7: List[List[Any]] = List(List(Plain Donut, 1.5), List(Strawberry Donut, 2.0), List(Glazed Donut, 2.5))

```

## demo2

```scala

scala> val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")
donuts: Seq[String] = List(Plain Donut, Strawberry Donut, Glazed Donut)

scala> val prices: Seq[Double] = Seq(1.50, 2.0, 2.50)
prices: Seq[Double] = List(1.5, 2.0, 2.5)

scala> val colors: Seq[String] = Seq("Green", "Blue", "Red")
color: Seq[String] = List(Green, Blue, Red)

scala> val donutList = List(donuts, prices, colors)
donutList: List[Seq[Any]] = List(List(Plain Donut, Strawberry Donut, Glazed Donut), List(1.5, 2.0, 2.5), List(Green, Blue, Red))

scala> donutList.transpose
res8: List[List[Any]] = List(List(Plain Donut, 1.5, Green), List(Strawberry Donut, 2.0, Blue), List(Glazed Donut, 2.5, Red))

```
