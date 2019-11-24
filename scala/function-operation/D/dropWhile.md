# dropWhile

## definition

```scala
abstract def dropWhile(pred: (A) ⇒ Boolean): Repr
Drops longest prefix of elements that satisfy a predicate.

Note: might return different results for different runs, unless the underlying collection type is ordered.
pred
    The predicate used to test elements.
returns
    the longest suffix of this general collection whose first element does not satisfy the predicate p.
```

顾名思义，当满足条件时丢弃。返回最长子集，当第一个不满足条件的元素出现时，即停止。

## demo

```scala
val donuts: Seq[String] = Seq("Plain Donut 1", "Plain Donut 2", "Strawberry Donut", "Plain Donut 3", "Glazed Donut")

// declare a predicate function to be passed-through to the dropWhile function
val dropElementsPredicate: (String) => Boolean = (donutName) => donutName.charAt(0) == 'P'

val dropDonuts = donuts.dropWhile(dropElementsPredicate)
```

所以上面demo的输出结果是

```scala

scala> val dropDonuts = donuts.dropWhile(dropElementsPredicate)
dropDonuts: Seq[String] = List(Strawberry Donut, Plain Donut 3, Glazed Donut)
```
