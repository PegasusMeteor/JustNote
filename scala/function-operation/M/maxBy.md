# maxBy

## definition

```scala
def maxBy[B](f: (A) ⇒ B): A
```

## demo

```scala
case class Donut(name: String, price: Double)

val donuts: Seq[Donut] = Seq(Donut("Plain Donut", 1.5), Donut("Strawberry Donut", 2.0), Donut("Glazed Donut", 2.5))

// find the maximum element in a sequence of case classes objects using the maxBy function
val max1 = donuts.maxBy(donut => donut.price)


// declare a value predicate function for maxBy function
val donutsMaxBy: (Donut) => Double = (donut) => donut.price
val max2 = donuts.maxBy(donutsMaxBy)

```
