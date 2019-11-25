# view

## definition

The view method will create a non-strict version of the collection which means that the elements of the collection will only be made available at access time.

可以将view 方法理解为一个集合取值时的懒加载行为。

```scala
def view: TraversableView[A, Repr]
```

## demo

```scala
// create a large numeric range and take the first 10 odd numbers

scala> val largeOddNumberList: List[Int] = (1 to 1000000).filter(_ % 2 != 0).take(10).toList
largeOddNumberList: List[Int] = List(1, 3, 5, 7, 9, 11, 13, 15, 17, 19)

// lazily create a large numeric range and take the first 10 odd numbers
scala> val lazyLargeOddNumberList = (1 to 1000000).view.filter(_ % 2 != 0).take(10).toList
lazyLargeOddNumberList: List[Int] = List(1, 3, 5, 7, 9, 11, 13, 15, 17, 19)
```

虽然从输出的结果上看，二者是一致的。但是从执行的效率上看，使用view方法的执行效率明显更快。
从定义中我们可以看出，view 方法的执行，只在访问到元素的时候，才去创建。
