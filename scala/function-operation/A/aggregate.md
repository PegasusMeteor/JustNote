# aggregate

## definition

```scala
    abstract def aggregate[B](z: ⇒ B)(seqop: (B, A) ⇒ B, combop: (B, B) ⇒ B): B
```

## demo1

```scala
val donutBasket1: Set[String] = Set("Plain Donut", "Strawberry Donut")

// define an accumulator function to calculate the total length of the String elements
val donutLengthAccumulator: (Int, String) => Int = (accumulator, donutName) => accumulator + donutName.length

val totalLength = donutBasket1.aggregate(0)(donutLengthAccumulator, _ + _)

```

## demo2

```scala

val donutBasket2: Set[(String, Double, Int)] = Set(("Plain Donut", 1.50, 10), ("Strawberry Donut", 2.0, 10))

//define an accumulator function to calculate the total cost of Donuts
val totalCostAccumulator: (Double, Double, Int) => Double = (accumulator, price, quantity) => accumulator + (price * quantity)


val totalCost = donutBasket2.aggregate(0.0)((accumulator: Double, tuple: (String, Double, Int)) => totalCostAccumulator(accumulator, tuple._2, tuple._3), _ + _)

```
