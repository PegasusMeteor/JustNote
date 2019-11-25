# map

## definition

```scala
def map[B](f: (A) ⇒ B): Traversable[B]
```

## demo

```scala
val donuts1: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

val donuts2: Seq[String] = donuts1.map(_ + " Donut")

val donuts3: Seq[AnyRef] = Seq("Plain", "Strawberry", None)

val donuts4: Seq[String] = donuts3.map {
 case donut: String => donut + " Donut"
 case None => "Unknown Donut"
}
```

