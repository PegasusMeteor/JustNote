# find

## definition

```scala
    def find(p: (A) ⇒ Boolean): Option[A]
```

## demo

```scala
val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut")

// find a particular element in the sequence using the find function
val plainDonut: Option[String] = donuts.find(_ == "Plain Donut")


```
