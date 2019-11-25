# reduce

## definition

The reduce method takes an associative binary operator function as parameter and will use it to collapse elements from the collection. Unlike the fold method, reduce does not allow you to also specify an initial value.

```scala
def reduce[A1 >: A](op: (A1, A1) ⇒ A1): A1
```

## demo

```scala
val donutPrices: Seq[Double] = Seq(1.5, 2.0, 2.5)

// find the sum of elements using reduce function explicitly
val sum: Double = donutPrices.reduce(_ + _)

val sum1: Double = donutPrices.reduce((a, b) => a + b)

// find the cheapest donut using reduce function
val priceMin = donutPrices.reduce(_ min _)



val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

// concatenate the elements from the sequence using reduce function

val donutsConcat = donuts.reduce((left, right) => left + ", " + right)
```

输出的结果如下

```scala
scala> val donutsConcat = donuts.reduce((left, right) => left + ", " + right)
donutsConcat: String = Plain Donut, Strawberry Donut, Glazed Donut
```
