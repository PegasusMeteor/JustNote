# filter and filterNot

## definition

```scala
    def filter(p: (A) ⇒ Boolean): Repr

    def filterNot(p: (A) ⇒ Boolean): Repr
```

## demo

```scala
val donuts: Seq[String] = Seq("Plain Donut", "Strawberry Donut", "Glazed Donut", "Vanilla Donut")

val sequenceWithPlainAndGlazedDonut = donuts.filter { donutName =>
  donutName.contains("Plain") || donutName.contains("Glazed")
}

val sequenceWithoutVanillaDonut = donuts.filterNot(_  == "Vanilla Donut" )

```
