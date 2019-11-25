# reverse

## definition

The reverse method will create a new sequence with the elements in reversed order.

```scala
def reverse: Repr
```

## demo

```scala
val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

donuts.reverse.foreach(donut => println(s"donut = $donut"))
```
