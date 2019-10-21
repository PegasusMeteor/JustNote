# 集合的化简 reduceLeft 操作

<!-- TOC -->

- [集合的化简 reduceLeft 操作](#%e9%9b%86%e5%90%88%e7%9a%84%e5%8c%96%e7%ae%80-reduceleft-%e6%93%8d%e4%bd%9c)
  - [函数的定义](#%e5%87%bd%e6%95%b0%e7%9a%84%e5%ae%9a%e4%b9%89)
  - [Demo](#demo)

<!-- /TOC -->

## 函数的定义

首先以 Seq 来查看一下函数的定义。

```scala
def reduceLeft[B >: A](op: (B, A) ⇒ B): B

    Applies a binary operator to all elements of this traversable or iterator, going left to right.

    Note: will not terminate for infinite-sized collections.

    Note: might return different results for different runs, unless the underlying collection type is ordered or the operator is associative and commutative.

    B
    the result type of the binary operator.

    op
    the binary operator.

    returns
    the result of inserting op between consecutive elements of this traversable or iterator, going left to right:

        op( op( ... op(x_1, x_2) ..., x_{n-1}), x_n)

    where x1, ..., xn are the elements of this traversable or iterator.
```

1. reduceLeft 接收的函数类型为 `op: (B, A) ⇒ B`
2. 运行规则是，从左边开始执行，将得到的结果返回给第一个参数,然后继续和下一个元素运算，将得到的结果继续返回给第一个参数,继续执行。

## Demo

```scala

```

```scala

object ReduceLeftDemo1 {
  def main(args: Array[String]): Unit = {

    val list = List(1,2,3,5,6)

    val list1 = list.reduceLeft(_+_)
    println(list1)
  }

}
```

上面两个demo 的运行结果，其实也就是相当于下面的表达式

```scala

((((1 + 2) + 3) +5) +6)

```
