# 集合的化简 foldLeft 操作

<!-- TOC -->

- [集合的化简 foldLeft 操作](#%e9%9b%86%e5%90%88%e7%9a%84%e5%8c%96%e7%ae%80-foldleft-%e6%93%8d%e4%bd%9c)
  - [函数的定义](#%e5%87%bd%e6%95%b0%e7%9a%84%e5%ae%9a%e4%b9%89)
  - [Demo](#demo)

<!-- /TOC -->

## 函数的定义

首先以Seq为例查询一下函数的定义

```scala
def foldLeft[B](z: B)(op: (B, A) ⇒ B): B
    Applies a binary operator to a start value and all elements of this traversable or iterator, going left to right.

    Note: will not terminate for infinite-sized collections.

    Note: might return different results for different runs, unless the underlying collection type is ordered or the operator is associative and commutative.

    B
    the result type of the binary operator.

    z
    the start value.

    op
    the binary operator.

    returns
    the result of inserting op between consecutive elements of this traversable or iterator, going left to right with the start value z on the left:

    op(...op(z, x_1), x_2, ..., x_n)
    where x1, ..., xn are the elements of this traversable or iterator. Returns z if this traversable or iterator is empty.

```

fold 函数将上一步返回的值，作为函数的第一个参数继续参与运算。直到List中的所有的元素被遍历。

## Demo

```scala
object FoldDemo1 {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 5, 6)

    val list1 = list.fold(5)(minus)
    println(list1)

    val  list2 = list.foldLeft(5)(minus)
    println(list2)

    val list3 = list.foldRight(5)(minus)
    println(list3)
  }

  def minus(n1: Int, n2: Int): Int = {
    n1 - n2
  }
}
```

上面的示例中，list1和list2的执行结果相当于

```shell
(((((5-1)-2)-3)-5)-6)
```

list3的执行结果相当于

```shell
(1-(2-(3-(5-(6-5)))))
```
