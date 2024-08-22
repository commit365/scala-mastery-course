# Scala's Type System

Scala has a rich and powerful type system that provides strong compile-time guarantees and enables flexible code design. This lesson covers some of the advanced features of Scala's type system.

## Generics

Generics allow you to write flexible, reusable code that works with different types.

```scala
class Box[T](var content: T) {
  def put(newContent: T): Unit = { content = newContent }
  def get: T = content
}

val intBox = new Box[Int](5)
val stringBox = new Box[String]("Hello")

println(intBox.get) // Outputs: 5
stringBox.put("World")
println(stringBox.get) // Outputs: World
```

## Variance

Variance defines how subtyping between more complex types relates to subtyping between their components.

### Covariance (+T)

Covariance means that if A is a subtype of B, then Box[A] is a subtype of Box[B].

```scala
class Animal
class Dog extends Animal
class Cat extends Animal

class CovariantBox[+T](val content: T)

val animalBox: CovariantBox[Animal] = new CovariantBox[Dog](new Dog)
```

### Contravariance (-T)

Contravariance means that if A is a subtype of B, then Box[B] is a subtype of Box[A].

```scala
trait Printer[-T] {
  def print(value: T): Unit
}

class AnimalPrinter extends Printer[Animal] {
  def print(animal: Animal): Unit = println("Animal")
}

val dogPrinter: Printer[Dog] = new AnimalPrinter
dogPrinter.print(new Dog) // Outputs: Animal
```

### Invariance

Invariance means that Box[A] and Box[B] are never subtypes of one another, regardless of the relationship between A and B.

```scala
class InvariantBox[T](var content: T)

// This won't compile:
// val animalBox: InvariantBox[Animal] = new InvariantBox[Dog](new Dog)
```

## Advanced Types

### Structural Types

Structural types allow you to define types based on structure rather than name.

```scala
def useCloseable(closeable: { def close(): Unit }): Unit = {
  closeable.close()
}

class Database {
  def close(): Unit = println("Closing database")
}

useCloseable(new Database)
```

### Compound Types

Compound types allow you to combine multiple types.

```scala
trait Resettable {
  def reset(): Unit
}

trait Growable[T] {
  def add(x: T): Unit
}

def useResetGrow(x: Resettable with Growable[String]): Unit = {
  x.reset()
  x.add("Hello")
}
```

### Type Aliases

Type aliases allow you to give a new name to an existing type.

```scala
type IntPair = (Int, Int)

def addPair(pair: IntPair): Int = pair._1 + pair._2

println(addPair((3, 4))) // Outputs: 7
```

### Path-Dependent Types

Path-dependent types are types that are nested in other types and depend on specific instances.

```scala
class Outer {
  class Inner
  def createInner: Inner = new Inner
}

val outer1 = new Outer
val outer2 = new Outer

val inner1: outer1.Inner = outer1.createInner
// This won't compile:
// val inner2: outer2.Inner = outer1.createInner
```

## Practice Exercises

1. Create a generic `Stack[T]` class with `push`, `pop`, and `isEmpty` methods.

2. Implement a contravariant `Comparator[-T]` trait and use it to sort a list of different types.

3. Define a structural type for "anything that has a `length` property" and write a function that uses it.

4. Create a compound type that combines multiple traits and use it in a function.

## Advanced Example: Combining Type System Features

Let's create a more complex example that combines various features of Scala's type system:

```scala
// Type alias for a function that processes a value and returns a boolean
type Predicate[-T] = T => Boolean

// Trait with a type parameter and a path-dependent type
trait Container[+A] {
  type ItemType = A
  def getItem: ItemType
  def processItem(pred: Predicate[ItemType]): Boolean
}

// Covariant class implementing the Container trait
class Box[+T](value: T) extends Container[T] {
  def getItem: ItemType = value
  def processItem(pred: Predicate[ItemType]): Boolean = pred(value)
}

// Structural type for anything with a 'size' method returning an Int
type Sizeable = {
  def size: Int
}

// Function using a compound type
def processContainer[A <: Sizeable](container: Container[A] with Sizeable): Int = {
  container.size
}

// Usage
val intBox = new Box(5) with Sizeable {
  def size: Int = 1
}

val stringBox = new Box("Hello") with Sizeable {
  def size: Int = 5
}

println(processContainer(intBox)) // Outputs: 1
println(processContainer(stringBox)) // Outputs: 5

// Using the covariant nature of Box
val numBox: Box[Number] = new Box[Int](10)

// Using the contravariant Predicate
val numPredicate: Predicate[Number] = (n: Number) => n.intValue() > 0
println(numBox.processItem(numPredicate)) // Outputs: true
```

This example demonstrates:
- Type aliases (`Predicate[-T]`)
- Covariance (`Container[+A]`, `Box[+T]`)
- Contravariance (`Predicate[-T]`)
- Path-dependent types (`Container.ItemType`)
- Structural types (`Sizeable`)
- Compound types (`Container[A] with Sizeable`)

## Conclusion

Scala's type system is powerful and flexible, allowing for precise modeling of complex relationships between types. Features like generics, variance, and advanced types enable you to write more expressive and type-safe code. As you become more comfortable with these concepts, you'll be able to leverage Scala's type system to create robust and reusable abstractions.

Understanding and effectively using Scala's type system is crucial for writing idiomatic Scala code and taking full advantage of the language's capabilities. In the next lesson, we'll explore pattern matching and case classes, which work hand in hand with Scala's type system to provide powerful data manipulation capabilities.