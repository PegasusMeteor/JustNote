# union

## definition

The union method takes a Set as parameter and will merge its elements with the elements from the current Set.

```scala
def union(that: GenSet[A]): This
```

## demo

```scala
scala> val donuts1: Set[String] = Set("Plain Donut", "Strawberry Donut", "Glazed Donut")
donuts1: Set[String] = Set(Plain Donut, Strawberry Donut, Glazed Donut)

scala> val donuts2: Set[String] = Set("Plain Donut", "Chocolate Donut", "Vanilla Donut")
donuts2: Set[String] = Set(Plain Donut, Chocolate Donut, Vanilla Donut)

scala> donuts1 union donuts2
res9: scala.collection.immutable.Set[String] = Set(Vanilla Donut, Plain Donut, Chocolate Donut, Strawberry Donut, Glazed Donut)

scala> donuts2 union donuts1
res10: scala.collection.immutable.Set[String] = Set(Vanilla Donut, Plain Donut, Chocolate Donut, Strawberry Donut, Glazed Donut)

scala> donuts2 ++ donuts1
res11: scala.collection.immutable.Set[String] = Set(Vanilla Donut, Plain Donut, Chocolate Donut, Strawberry Donut, Glazed Donut)

scala> donuts1 ++ donuts2
res12: scala.collection.immutable.Set[String] = Set(Vanilla Donut, Plain Donut, Chocolate Donut, Strawberry Donut, Glazed Donut)
```
