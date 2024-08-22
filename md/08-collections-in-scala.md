# Collections in Scala

Scala provides a rich and powerful collections library. Understanding the collection hierarchy and the operations available on collections is crucial for effective Scala programming.

## Collection Hierarchy

Scala collections are divided into two main categories: immutable and mutable.

### Immutable Collections

Immutable collections cannot be modified after creation. The main immutable collections are:

- `List`: A linked list
- `Vector`: An indexed sequence
- `Set`: A collection of unique elements
- `Map`: A key-value store

```scala
import scala.collection.immutable._

val numbers = List(1, 2, 3, 4, 5)
val vector = Vector("a", "b", "c")
val uniqueNumbers = Set(1, 2, 3, 3, 4, 5)
val nameAges = Map("Alice" -> 30, "Bob" -> 25)
```

### Mutable Collections

Mutable collections can be modified after creation. The main mutable collections are:

- `ArrayBuffer`: A growable array
- `ListBuffer`: A mutable linked list
- `mutable.Set`: A mutable set of unique elements
- `mutable.Map`: A mutable key-value store

```scala
import scala.collection.mutable

val growableArray = mutable.ArrayBuffer(1, 2, 3)
val mutableList = mutable.ListBuffer("a", "b", "c")
val mutableSet = mutable.Set(1, 2, 3)
val mutableMap = mutable.Map("x" -> 1, "y" -> 2)
```

## Common Collection Operations

Scala collections share many common operations. Here are some of the most frequently used:

### Transformations

```scala
val numbers = List(1, 2, 3, 4, 5)

// map: Apply a function to each element
val doubled = numbers.map(_ * 2)
println(doubled) // List(2, 4, 6, 8, 10)

// filter: Keep elements that satisfy a predicate
val evens = numbers.filter(_ % 2 == 0)
println(evens) // List(2, 4)

// flatMap: Map and flatten the results
val pairs = numbers.flatMap(n => List(n, n))
println(pairs) // List(1, 1, 2, 2, 3, 3, 4, 4, 5, 5)
```

### Aggregations

```scala
val numbers = List(1, 2, 3, 4, 5)

// reduce: Combine elements using a binary operation
val sum = numbers.reduce(_ + _)
println(sum) // 15

// fold: Like reduce, but with an initial value
val sumPlus10 = numbers.fold(10)(_ + _)
println(sumPlus10) // 25

// aggregate: More general form of fold for different result types
val (count, sum) = numbers.aggregate((0, 0))({ case ((c, s), x) => (c + 1, s + x) }, { case ((c1, s1), (c2, s2)) => (c1 + c2, s1 + s2) })
println(s"Count: $count, Sum: $sum") // Count: 5, Sum: 15
```

### Queries

```scala
val numbers = List(1, 2, 3, 4, 5)

// find: Return the first element matching a predicate
val firstEven = numbers.find(_ % 2 == 0)
println(firstEven) // Some(2)

// exists: Check if any element satisfies a predicate
val hasEven = numbers.exists(_ % 2 == 0)
println(hasEven) // true

// forall: Check if all elements satisfy a predicate
val allPositive = numbers.forall(_ > 0)
println(allPositive) // true
```

## Performance Characteristics

Different collections have different performance characteristics:

- `List`: Constant time prepend and head/tail access. Linear time for other operations.
- `Vector`: Effectively constant time for append, prepend, and random access.
- `Set` and `Map`: Near constant time contains and add operations.
- `ArrayBuffer`: Constant time append and random access. Amortized constant time prepend.

```scala
import scala.collection.mutable

// List: fast prepend
val list = 1 :: 2 :: 3 :: Nil
println(list) // List(1, 2, 3)

// Vector: fast random access
val vector = Vector(1, 2, 3, 4, 5)
println(vector(2)) // 3

// Set: fast contains check
val set = Set(1, 2, 3, 4, 5)
println(set.contains(3)) // true

// ArrayBuffer: fast append
val buffer = mutable.ArrayBuffer(1, 2, 3)
buffer += 4
println(buffer) // ArrayBuffer(1, 2, 3, 4)
```

## Choosing the Right Collection

- Use `List` for sequences that are accessed from the head and for recursive algorithms.
- Use `Vector` for sequences that require fast random access or are large.
- Use `Set` when you need to ensure uniqueness of elements.
- Use `Map` for key-value associations.
- Use mutable collections when you need to build up a collection incrementally and performance is critical.

## Practice Exercises

1. Create a `List` of integers and use `map`, `filter`, and `reduce` to calculate the sum of the squares of even numbers.

2. Implement a function that takes a `List` of strings and returns a `Map` where the keys are the lengths of the strings and the values are `Set`s of strings with that length.

3. Use `foldLeft` to reverse a `List` without using the built-in `reverse` method.

4. Create a mutable `Set` and demonstrate adding, removing, and checking for elements.

## Advanced Example: Combining Collection Operations

Let's create a more complex example that combines various collection operations:

```scala
case class Person(name: String, age: Int, city: String)

val people = List(
  Person("Alice", 25, "New York"),
  Person("Bob", 30, "San Francisco"),
  Person("Charlie", 35, "London"),
  Person("David", 28, "New York"),
  Person("Eve", 22, "San Francisco")
)

// Group people by city, then calculate the average age for each city
val averageAgeByCity = people
  .groupBy(_.city)
  .view
  .mapValues(cityPeople => cityPeople.map(_.age).sum.toDouble / cityPeople.length)
  .toMap

println("Average age by city:")
averageAgeByCity.foreach { case (city, avgAge) =>
  println(f"$city: $avgAge%.1f")
}

// Find the names of people older than the average age of their city
val aboveAveragePeople = people.filter { person =>
  person.age > averageAgeByCity(person.city)
}.map(_.name)

println("\nPeople older than their city's average age:")
aboveAveragePeople.foreach(println)

// Create a map of the oldest person in each city
val oldestByCity = people
  .groupBy(_.city)
  .view
  .mapValues(cityPeople => cityPeople.maxBy(_.age))
  .toMap

println("\nOldest person in each city:")
oldestByCity.foreach { case (city, person) =>
  println(s"$city: ${person.name} (${person.age})")
}

// Calculate the overall age distribution
val ageDistribution = people
  .map(_.age)
  .groupBy(age => age / 10 * 10) // Group by decade
  .view
  .mapValues(_.length)
  .toMap
  .toList
  .sortBy(_._1)

println("\nAge distribution:")
ageDistribution.foreach { case (decade, count) =>
  println(s"${decade}s: ${"*" * count}")
}
```

This example demonstrates:
- Use of case classes with collections
- Grouping and aggregating data
- Transforming collections into different shapes
- Combining multiple collection operations to perform complex analyses
- Using view to optimize intermediate operations

## Conclusion

Scala's collection library is rich and powerful, offering a wide range of immutable and mutable data structures. Understanding the different types of collections, their operations, and performance characteristics is crucial for writing efficient and idiomatic Scala code.

The functional nature of Scala's collections, with operations like `map`, `filter`, and `reduce`, allows for expressive and concise code. By choosing the right collection for your use case and leveraging the appropriate operations, you can write clean, efficient, and maintainable code.

As you continue to work with Scala, you'll find that mastering collections is key to solving many programming challenges elegantly. In the next lesson, we'll explore functional programming concepts in more depth, building on the foundation laid by Scala's powerful collection library.