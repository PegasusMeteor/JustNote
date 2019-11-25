# par

## definition

```scala
def par: ParRepr
```

The par method is a member of the Parallelizable trait.

## demo

```scala
val donutFlavours: Seq[String] = Seq("Plain", "Strawberry", "Glazed")

// Convert the Immutable donut flavours Sequence into Parallel Collection
import scala.collection.parallel.ParSeq
val donutFlavoursParallel: ParSeq[String] = donutFlavours.par

val donuts: ParSeq[String] = donutFlavoursParallel.map(d => s"$d donut")
```
