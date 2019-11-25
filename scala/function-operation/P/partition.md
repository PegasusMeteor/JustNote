# partion

## definition

The partition method takes a predicate function as its parameter and will use it to return two collections: one collection with elements that satisfied the predicate function and another collection with elements that did not match the predicate function.

```scala
def partition(p: (A) ⇒ Boolean): (Repr, Repr)
```

## demo

```scala

val donutNamesAndPrices: Seq[Any] = Seq("Plain Donut", 1.5, "Strawberry Donut", 2.0, "Glazed Donut", 2.5)
val namesAndPrices: (Seq[Any], Seq[Any]) = donutNamesAndPrices.partition {
  case name: String => true
  case price: Double => false
}

// access the donut prices sequence from namesAndPrices
val prices = namesAndPrices._2


```

输出结果如下所示

```scala
scala> val namesAndPrices: (Seq[Any], Seq[Any]) = donutNamesAndPrices.partition {
     |   case name: String => true
     |   case price: Double => false
     | }
namesAndPrices: (Seq[Any], Seq[Any]) = (List(Plain Donut, Strawberry Donut, Glazed Donut),List(1.5, 2.0, 2.5))


```