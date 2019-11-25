# take

## definition

The take method takes an integer N as parameter and will use it to return a new collection consisting of the first N elements.

```scala
def take(n: Int): Repr
```

## demo

```scala
scala> val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut", "Hello World")
donuts: Seq[String] = List(Plain Donut, Strawberry Donut, Glazed Donut, Hello World)

scala> donuts.take(1)
res4: Seq[String] = List(Plain Donut)

scala> donuts.take(2)
res5: Seq[String] = List(Plain Donut, Strawberry Donut)

scala> donuts.take(3)
res6: Seq[String] = List(Plain Donut, Strawberry Donut, Glazed Donut)

```
