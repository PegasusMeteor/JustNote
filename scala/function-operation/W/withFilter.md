# withFilter

## definition

The withFilter method takes a predicate function and will restrict the elements to match the predicate function. withFilter does not create a new collection while filter() method will create a new collection.

预测函数，限制元素匹配，不创建新的集合

```scala
def withFilter(p: (A) ⇒ Boolean): FilterMonadic[A, Repr]
```

## demo

```scala
val donuts: Seq[String] = List("Plain Donut", "Strawberry Donut", "Glazed Donut")
donuts.withFilter(_.charAt(0) == 'P').foreach(donut => println(s"Donut starting with letter P = $donut"))
```
