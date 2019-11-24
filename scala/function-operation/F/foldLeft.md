# foldLeft

## definition

```scala
def foldLeft[B](z: B)(op: (B, A) ⇒ B): B
```

## demo1

```scala
val prices: Seq[Double] = Seq(1.5, 2.0, 2.5)

// sum all the donut prices using foldLeft function
val sum = prices.foldLeft(0.0)(_ + _)

```

## demo2

```scala

val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

// create a String of all donuts using foldLeft function
val singleString = donuts.foldLeft("")((a, b) => a + b + " Donut ")

```

## demo3

```scala

val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

val concatDonuts: (String, String) => String = (a, b) => a + b + " Donut "

// create a String of all donuts using foldLeft function
val singleString = donuts.foldLeft("")(concatDonuts)

```
