# Advanced Collections and Data Structures in Scala

Scala’s collection library is rich and versatile, supporting both mutable and immutable data structures. This lesson focuses on advanced collections, including persistent data structures, specialized collections, and custom collection types.

## Persistent Data Structures

Persistent data structures are immutable data structures that preserve the previous version of themselves when modified. This allows for efficient sharing of structure between versions, which is particularly useful in functional programming.

### Characteristics of Persistent Data Structures

1. **Immutability**: Once created, the structure cannot be changed. Any modification results in a new version of the structure.

2. **Efficiency**: Persistent data structures are designed to share as much of their structure as possible between versions, minimizing memory overhead.

3. **Functional Programming**: They align well with functional programming principles, making it easier to reason about code.

### Example: Persistent List

Scala’s standard `List` is an example of a persistent data structure. When you prepend an element to a list, a new list is created that shares the tail of the original list.

```scala
val originalList = List(2, 3, 4)
val newList = 1 :: originalList // Prepend 1

println(originalList) // Outputs: List(2, 3, 4)
println(newList)      // Outputs: List(1, 2, 3, 4)
```

### Implementing a Persistent Data Structure

You can implement your own persistent data structures. Here’s an example of a simple persistent stack:

```scala
sealed trait PersistentStack[+A] {
  def push[B >: A](value: B): PersistentStack[B]
  def pop: (Option[A], PersistentStack[A])
}

case object EmptyStack extends PersistentStack[Nothing] {
  def push[B](value: B): PersistentStack[B] = new NonEmptyStack(value, this)
  def pop: (Option[Nothing], PersistentStack[Nothing]) = (None, this)
}

case class NonEmptyStack[A](top: A, rest: PersistentStack[A]) extends PersistentStack[A] {
  def push[B >: A](value: B): PersistentStack[B] = new NonEmptyStack(value, this)
  def pop: (Option[A], PersistentStack[A]) = (Some(top), rest)
}

// Usage
val stack = EmptyStack.push(1).push(2).push(3)
val (top, newStack) = stack.pop
println(top) // Outputs: Some(3)
```

## Specialized Collections

Scala provides specialized collections that are optimized for specific use cases, improving performance and memory usage.

### Specialized Collections in Scala

1. **Vector**: A general-purpose immutable sequence that offers fast random access. It is optimized for performance and is often used as a drop-in replacement for lists.

2. **ArrayBuffer**: A mutable sequence that provides fast random access and efficient appending. It is implemented as a resizable array.

3. **HashSet**: An immutable set that uses a hash table for fast lookups, insertions, and deletions.

4. **TreeSet**: An immutable sorted set that maintains its elements in a sorted order, providing logarithmic time complexity for add, remove, and contains operations.

5. **Map**: Immutable maps provide key-value associations. `HashMap` is optimized for fast lookups, while `TreeMap` maintains sorted order.

### Example of Specialized Collections

```scala
import scala.collection.immutable.{Vector, HashSet, TreeMap}

// Vector example
val vector = Vector(1, 2, 3, 4)
val updatedVector = vector :+ 5 // Append 5
println(updatedVector) // Outputs: Vector(1, 2, 3, 4, 5)

// HashSet example
val hashSet = HashSet(1, 2, 3)
val updatedSet = hashSet + 4
println(updatedSet) // Outputs: HashSet(1, 2, 3, 4)

// TreeMap example
val treeMap = TreeMap("b" -> 2, "a" -> 1)
val updatedMap = treeMap + ("c" -> 3)
println(updatedMap) // Outputs: TreeMap(a -> 1, b -> 2, c -> 3)
```

## Custom Collection Types

You can create custom collection types in Scala to meet specific requirements. This allows you to define your own behavior and optimizations.

### Implementing a Custom Collection

Here’s an example of a simple custom collection that represents a stack:

```scala
class MyStack[A] {
  private var elements: List[A] = List.empty

  def push(element: A): Unit = {
    elements = element :: elements
  }

  def pop(): Option[A] = {
    elements match {
      case Nil => None
      case head :: tail =>
        elements = tail
        Some(head)
    }
  }

  def peek: Option[A] = elements.headOption

  def isEmpty: Boolean = elements.isEmpty
}

// Usage
val myStack = new MyStack[Int]
myStack.push(1)
myStack.push(2)
println(myStack.peek) // Outputs: Some(2)
println(myStack.pop()) // Outputs: Some(2)
println(myStack.pop()) // Outputs: Some(1)
println(myStack.pop()) // Outputs: None
```

### Benefits of Custom Collections

- **Tailored Behavior**: You can define specific behaviors and optimizations that suit your application’s needs.
- **Encapsulation**: Custom collections can encapsulate complex logic while providing a simple interface.

## Performance Optimization Techniques

When working with collections, consider the following optimization techniques:

1. **Choose the Right Collection**: Use specialized collections that fit your use case (e.g., `Vector` for random access, `List` for sequential access).

2. **Avoid Unnecessary Copies**: Use mutable collections when performance is critical, but be cautious of side effects.

3. **Use Views**: For large collections, consider using views to avoid unnecessary materialization.

4. **Batch Operations**: When performing multiple updates, batch them to reduce overhead.

## Benchmarking Collections

To measure the performance of different collections, you can use benchmarking libraries like JMH (Java Microbenchmark Harness).

### Example of Benchmarking

1. **Add JMH Dependency** to your `build.sbt`:

```sbt
libraryDependencies += "org.openjdk.jmh" % "jmh-core" % "1.34"
libraryDependencies += "org.openjdk.jmh" % "jmh-generator-annprocess" % "1.34"
```

2. **Create a Benchmark Class**:

```scala
import org.openjdk.jmh.annotations._

@State(Scope.Thread)
class CollectionBenchmark {
  val list = (1 to 1000).toList
  val vector = (1 to 1000).toVector

  @Benchmark
  def benchmarkListAppend(): List[Int] = {
    list :+ 1001
  }

  @Benchmark
  def benchmarkVectorAppend(): Vector[Int] = {
    vector :+ 1001
  }
}
```

3. **Run the Benchmark**:

You can run the benchmark using sbt:

```bash
sbt "jmh:run -i 10 -wi 10 -f1 -t1"
```

## Practice Exercises

1. Implement a custom collection type that represents a queue with enqueue and dequeue operations.

2. Create a persistent data structure for a binary tree and implement insertion and traversal methods.

3. Write a benchmark comparing the performance of different Scala collections (e.g., List, Vector, ArrayBuffer) for various operations.

4. Implement a specialized collection that maintains a sorted order of elements as they are added.

## Advanced Example: Combining Advanced Collections

Let’s create a more complex example that combines persistent data structures and custom collection types:

```scala
sealed trait PersistentList[+A] {
  def prepend[B >: A](value: B): PersistentList[B]
  def headOption: Option[A]
  def tail: PersistentList[A]
}

case object Empty extends PersistentList[Nothing] {
  def prepend[B](value: B): PersistentList[B] = new NonEmptyList(value, this)
  def headOption: Option[Nothing] = None
  def tail: PersistentList[Nothing] = this
}

case class NonEmptyList[A](head: A, tail: PersistentList[A]) extends PersistentList[A] {
  def prepend[B >: A](value: B): PersistentList[B] = new NonEmptyList(value, this)
  def headOption: Option[A] = Some(head)
}

// Usage
val list = Empty.prepend(1).prepend(2).prepend(3)
println(list.headOption) // Outputs: Some(3)
```

In this example:
- We define a persistent list data structure with methods for prepending elements and retrieving the head.
- We implement both empty and non-empty cases for the list.

## Conclusion

Scala's advanced collections and data structures provide powerful tools for building efficient and expressive applications. By understanding persistent data structures, specialized collections, and custom collection types, you can leverage Scala's capabilities to create robust software solutions.

Mastering these advanced concepts will enhance your ability to write clean, maintainable, and high-performance code. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.