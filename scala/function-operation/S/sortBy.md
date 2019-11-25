# sortBy

## definition

```scala
def sortBy[B](f: (A) ⇒ B)(implicit ord: math.Ordering[B]): Repr
```

## demo

```scala
scala> case class Donut(name: String, price: Double)
defined class Donut

scala> val donuts: Seq[Donut] = Seq(Donut("Plain Donut", 1.5), Donut("Strawberry Donut", 2.0), Donut("Glazed Donut", 2.5))
donuts: Seq[Donut] = List(Donut(Plain Donut,1.5), Donut(Strawberry Donut,2.0), Donut(Glazed Donut,2.5))

scala> donuts.sortBy(donut => donut.price)
res3: Seq[Donut] = List(Donut(Plain Donut,1.5), Donut(Strawberry Donut,2.0), Donut(Glazed Donut,2.5))
```
