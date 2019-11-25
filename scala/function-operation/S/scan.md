# scan

## definition

The scan method takes an associative binary operator function as parameter and will use it to collapse elements from the collection to create a running total from each element. Similar to the fold method, scan allows you to also specify an initial value.

```scala
def scan[B >: A, That](z: B)(op: (B, B) ⇒ B)(implicit cbf: CanBuildFrom[Repr, B, That]): That
```

## demo

```scala
scala> val numbers: Seq[Int] = Seq(1, 2, 3, 4, 5)
numbers: Seq[Int] = List(1, 2, 3, 4, 5)

scala> val runningTotal: Seq[Int] = numbers.scan(0)(_ + _)
runningTotal: Seq[Int] = List(0, 1, 3, 6, 10, 15)

scala> val runningTotal2: Seq[Int] = numbers.scan(0)((a, b) => a + b)
runningTotal2: Seq[Int] = List(0, 1, 3, 6, 10, 15)
```

如果以前不太熟悉这个方法的话，可能有点懵。
下面是这个方法的执行过程。

```scala
Scan method iterations
0 + 1             =  1
1 + 2             =  3
1 + 2 + 3         =  6
1 + 2 + 3 + 4     = 10
1 + 2 + 3 + 4 + 5 = 15
```
