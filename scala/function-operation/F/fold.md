# fold

## definition

```scala
def fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
```

## demo

```scala

val prices: Seq[Double] = Seq(1.5, 2.0, 2.5)

// sum all the donut prices using fold function
val sum = prices.fold(0.0)(_ + _)

```

## demo2

```scala
val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

// create a String of all donuts using fold function
val singleString = donuts.fold("")((acc,s) => acc + s + " Donut ")


```

输出的内容是

```scala
scala> val singleString = donuts.fold("")((acc,s) => acc + s + " Donut ")
singleString: String = "Plain Donut Strawberry Donut Glazed Donut "
```

## demo3

```scala
val donuts: Seq[String] = Seq("Plain", "Strawberry", "Glazed")
// declare a value function to create the donut string

val concatDonuts:(String,String) => String = (acc,s) =>  acc + s + " Donut "

val singleString = donuts.fold("")(concatDonuts)
```

输出的结果与 demo2 等价，只不过将函数显式地定义在外面，比较好理解。

```scala
scala> val concatDonuts:(String,String) => String = (acc,s) =>  acc + s + " Donut "
concatDonuts: (String, String) => String = $$Lambda$1470/1463209038@32eea4f7

scala> val singleString = donuts.fold("")(concatDonuts)
singleString: String = "Plain Donut Strawberry Donut Glazed Donut "
```
