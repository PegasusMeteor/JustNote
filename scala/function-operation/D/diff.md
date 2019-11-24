# diff

## definition

```scala
abstract def diff(that: GenSeq[A]): GenSeq[A]
```

## demo

```scala
val donutBasket1: Set[String] = Set("Plain Donut", "Strawberry Donut", "Glazed Donut")
val donutBasket2: Set[String] = Set("Glazed Donut", "Vanilla Donut")

val diffDonutBasket1From2: Set[String] = donutBasket1 diff donutBasket2

val diffDonutBasket2From1 :Set[String] = donutBasket2 diff donutBasket1

```

可以看下下面的输出

```scala

scala> val diffDonutBasket2From1 :Set[String] = donutBasket2 diff donutBasket1
diffDonutBasket2From1: Set[String] = Set(Vanilla Donut)

scala> val diffDonutBasket1From2: Set[String] = donutBasket1 diff donutBasket2
diffDonutBasket1From2: Set[String] = Set(Plain Donut, Strawberry Donut)

```
