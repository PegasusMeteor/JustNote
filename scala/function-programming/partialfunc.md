# 偏函数

## 偏函数不是函数，而是一个triat

首先来看下偏函数的定义

## Demo

```scala

def main(args: Array[String]): Unit = {
    val list = List(1,3,4,5,"lits")


    // 输入的 是 Any ，返回的 Int
    // 如果返回true，则调用apply 创建对象实例，如果返回false，过滤
    val partialFunction = new PartialFunction[Any,Int] {
      override def isDefinedAt(x: Any): Boolean =  x.isInstanceOf[Int]

      // 对 传入 的值加 1 返回
      override def apply(v1: Any): Int = {
        v1.asInstanceOf[Int] + 1
      }
    }

    // 如果使用 偏函数，就要使用 collect
    // 遍历 list ，如果 true ，调用 apply，如果 false，不调用。
    val list2 = list.collect(partialFunction)
    println(list2)

  }

```

```scala
 def main(args: Array[String]): Unit = {

    val list = List(1,3,4.0,5,"lits")

    // 简写
    // isDefine apply 这两个函数 可以 不用写，直接写 case
    // case 后面的 表达式的意思 其实就相当于 isDefine 和 apply 的逻辑
    // 即 判断是否是某个类型，然后进行操作。
    // 这样的话，就可以写多个逻辑判断了
    def  f1:PartialFunction[Any,Int]={
        //   isDefine  apply
        case i:Int => i +1
        case j:Double => (j * 2).toInt
    }

    val list1 = list.collect(f1)
    println(list1)

    // 第二种简写形式,就是将 case 语句直接放入到 collect中。

    val list2 = list.collect{case i:Int => i +1}
    println(list2)

  }
```
