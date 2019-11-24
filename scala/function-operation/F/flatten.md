# flatten

## definition

```scala
def flatten[B]: Traversable[B]
```

## demo

```scala
val donuts1: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

val donuts2: Seq[String] = Seq("Vanilla Donut", "Glazed Donut")

val listDonuts: List[Seq[String]] = List(donuts1, donuts2)

// return a single list of donut using the flatten function
val listDonutsFromFlatten: List[String] = listDonuts.flatten

```
