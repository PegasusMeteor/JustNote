# collect

## definition

```scala
def collect[B](pf: PartialFunction[A, B]): Traversable[B]
```

## demo

```scala

val donutNamesandPrices: Seq[Any] = Seq("Plain Donut", 1.5, "Strawberry Donut", 2.0, "Glazed Donut", 2.5)

// use collect function to cherry pick all the donut names
val donutNames: Seq[String] = donutNamesandPrices.collect{ case name: String => name }

// use collect function to cherry pick all the donut prices
val donutPrices: Seq[Double] = donutNamesandPrices.collect{ case price: Double => price }

```
