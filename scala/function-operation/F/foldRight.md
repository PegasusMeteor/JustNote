# foldRight

## definition

```scala

def foldRight[B](z: B)(op: (A, B) ⇒ B): B

```

## demo1

```scala
val prices: Seq[Double] = Seq(1.5, 2.0, 2.5)

// sum all the donut prices using foldLeft function
val sum = prices.foldRight(0.0)(_ + _)

```

## demo2

```scala

val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

// create a String of all donuts using foldRight function
val singleString = donuts.foldRight("")((a, b) => a  + " Donut " + b)

```

## demo3

```scala

val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

val concatDonuts: (String, String) => String = (a, b) => a  + " Donut " + b

// create a String of all donuts using foldRight function
val singleString = donuts.foldRight("")(concatDonuts)

```
