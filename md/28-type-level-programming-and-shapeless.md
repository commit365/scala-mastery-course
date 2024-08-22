# Type-Level Programming and Shapeless

Type-level programming in Scala allows developers to perform computations and encode logic at the type level, enabling more expressive and type-safe code. Shapeless is a powerful library that provides tools for generic programming and type-level computations. This lesson covers key concepts such as HLists, Coproducts, and type-level computations using Shapeless.

## HLists

HLists (Heterogeneous Lists) are lists that can hold elements of different types. Unlike regular Scala lists, which are homogeneous (all elements must be of the same type), HLists allow for more flexibility in how data is represented.

### Defining HLists

You can define HLists using Shapeless as follows:

1. **Add the Shapeless dependency** to your `build.sbt`:

   ```sbt
   libraryDependencies += "com.chuusai" %% "shapeless" % "2.3.7"
   ```

2. **Creating HLists**:

```scala
import shapeless._

val hlist = 42 :: "Hello" :: true :: HNil // An HList with Int, String, and Boolean
```

### Accessing HList Elements

You can access elements of an HList using the `::` operator and the `HList` type class:

```scala
val intValue = hlist.head // Access the first element
val tail = hlist.tail // Get the tail of the HList
```

### Manipulating HLists

Shapeless provides various operations for manipulating HLists, such as `map`, `flatMap`, and `foldLeft`.

```scala
val hlist2 = hlist.map(_.toString) // Convert all elements to strings
println(hlist2) // Outputs: List(42, "Hello", "true")
```

## Coproducts

Coproducts represent a type that can be one of several types, similar to a sum type or union type in other languages. In Shapeless, Coproducts are used to define types that can hold values of different types.

### Defining Coproducts

You can define a Coproduct using Shapeless:

```scala
import shapeless._

val coproduct: Int :+: String :+: Boolean :+: CNil = Inl(42) // An Int Coproduct
```

### Pattern Matching on Coproducts

You can use pattern matching to extract values from a Coproduct:

```scala
def process(coprod: Int :+: String :+: Boolean :+: CNil): String = coprod match {
  case Inl(i) => s"Integer: $i"
  case Inr(Inl(s)) => s"String: $s"
  case Inr(Inr(Inl(b))) => s"Boolean: $b"
}

// Usage
println(process(Inl(42))) // Outputs: Integer: 42
println(process(Inr(Inl("Hello")))) // Outputs: String: Hello
```

## Generic Programming

Shapeless allows for generic programming, enabling you to write code that can operate on types without knowing their specific structure at compile time.

### Implementing Generic Programming

You can define a generic representation of a case class using Shapeless’s `Generic` type class:

```scala
case class Person(name: String, age: Int)

import shapeless.Generic

type PersonGen = Generic[Person]
val personGen = PersonGen()

val person = Person("Alice", 30)
val hlistRepresentation = personGen.to(person) // Convert to HList
println(hlistRepresentation) // Outputs: List("Alice", 30)

val backToPerson = personGen.from(hlistRepresentation) // Convert back to Person
println(backToPerson) // Outputs: Person(Alice, 30)
```

## Type-Level Computations and Proofs

Type-level programming allows you to perform computations and proofs at compile time, ensuring correctness and safety in your code.

### Type-Level Computations

You can define type-level computations using Shapeless’s type classes and implicit resolution.

```scala
import shapeless._
import shapeless.ops.hlist._

trait Length[L <: HList] {
  def length(l: L): Int
}

object Length {
  implicit def hnilLength: Length[HNil] = new Length[HNil] {
    def length(l: HNil): Int = 0
  }

  implicit def hconsLength[H, T <: HList](implicit tailLength: Length[T]): Length[H :: T] = new Length[H :: T] {
    def length(l: H :: T): Int = 1 + tailLength.length(l.tail)
  }
}

// Usage
val length = new Length[Int :: String :: HNil] {}
println(length.length(42 :: "Hello" :: HNil)) // Outputs: 2
```

### Type-Level Proofs

You can also use Shapeless to encode proofs and constraints at the type level.

```scala
import shapeless._
import shapeless.ops.nat._

def ensurePositive[N <: Nat](n: N)(implicit ev: N <:< Nat): String = {
  "The number is positive."
}

// Usage
val positiveProof = ensurePositive(3.nat) // This will compile
// val negativeProof = ensurePositive(-1.nat) // This will not compile
```

## Practice Exercises

1. Create an HList that contains different types of data and implement a function that prints each element.

2. Define a Coproduct type that can hold either an `Int` or a `String` and implement a function that processes the Coproduct.

3. Implement a generic representation for a case class representing a book with title and author fields.

4. Write a type-level computation that calculates the sum of two natural numbers using Shapeless.

## Advanced Example: Combining Concepts

Let’s create a more complex example that combines HLists, Coproducts, and generic programming:

```scala
import shapeless._
import shapeless.ops.coproduct._

sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape

object ShapeProcessor {
  def area[S <: Shape](shape: S)(implicit ev: S =:= Circle): Double = shape match {
    case Circle(radius) => Math.PI * radius * radius
    case Rectangle(width, height) => width * height
  }
}

// Example of using HLists and Coproducts
def processShapes(shapes: Shape :+: HNil): Unit = shapes match {
  case Inl(Circle(radius)) => println(s"Circle area: ${Math.PI * radius * radius}")
  case Inl(Rectangle(width, height)) => println(s"Rectangle area: ${width * height}")
}

// Usage
val shapes: Shape :+: HNil = Inl(Circle(5.0))
processShapes(shapes)
```

In this example:
- We define a `Shape` trait with `Circle` and `Rectangle` case classes.
- We implement a method to calculate the area of different shapes using pattern matching.
- We demonstrate the use of Coproducts to represent a list of shapes.

## Conclusion

Type-level programming in Scala, supported by libraries like Shapeless, allows for powerful abstractions and expressive code. By understanding and implementing concepts such as HLists, Coproducts, and generic programming, you can leverage Scala’s type system to create robust and type-safe applications.

Mastering these advanced concepts will enhance your ability to write clean, maintainable, and efficient code. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.