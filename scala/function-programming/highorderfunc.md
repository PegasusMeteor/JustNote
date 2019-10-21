# 高阶函数

可以接收函数作为参数的函数就是高阶函数。

## Demo

```scala

object HighOrderFuncDemo01 {

  def main(args: Array[String]): Unit = {
    val res = test(sum, 2.0)
    println(res)

    val f1 = myPrint _
    f1()
  }

  def myPrint(): Unit = {
    println("Hello World!")
  }

  def test(f: Double => Double, n1: Double) = {
    f(n1)
  }

  def sum(double: Double) = {
    println("sum 被调用")
    double + double
  }
}
```

1. test 就是一个高阶函数
2. f: Double => Double 表示一个接收Double类型，并返回Double类型的函数
3. f(n1) 表示执行传入的函数。在这个示例中就是  `sum`。
4. `val f1 = myPrint _` 可以将函数直接赋值给一个变量。 后面的下划线 表示 这个函数不执行，只赋值。
