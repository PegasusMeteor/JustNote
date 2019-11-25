# mkString

## definition

```scala
def mkString: String

def mkString(sep: String): String

def mkString(start: String, sep: String, end: String): String
```

## demo

```scala

val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

```

下面分别展示一下三种输出

```scala
scala> val donutsAsString: String = donuts.mkString(" and ")
donutsAsString: String = Plain Donut and Strawberry Donut and Glazed Donut

scala> val donutsWithPrefixAndSuffix: String = donuts.mkString("My favorite donuts namely ", " and ", " are very tasty!")
donutsWithPrefixAndSuffix: String = My favorite donuts namely Plain Donut and Strawberry Donut and Glazed Donut are very tasty!

scala> val donutsString = donuts.mkString
donutsString: String = Plain DonutStrawberry DonutGlazed Donut

```
