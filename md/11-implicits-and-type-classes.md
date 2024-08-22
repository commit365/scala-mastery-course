# Implicits and Type Classes in Scala

Implicits and type classes are powerful features in Scala that enable extensible and flexible code. This lesson covers their usage and implementation.

## Implicit Parameters

Implicit parameters allow you to pass parameters to a function without explicitly specifying them.

```scala
def greet(name: String)(implicit greeting: String): Unit = {
  println(s"$greeting, $name!")
}

implicit val defaultGreeting: String = "Hello"

greet("Alice") // Outputs: Hello, Alice!

// You can override the implicit value
implicit val spanishGreeting: String = "Hola"
greet("Bob") // Outputs: Hola, Bob!
```

## Implicit Conversions

Implicit conversions automatically convert one type to another when needed.

```scala
case class Meter(value: Double)
case class Foot(value: Double)

implicit def meterToFoot(meter: Meter): Foot = Foot(meter.value * 3.28084)

val distance: Meter = Meter(5)
val heightRequirement: Foot = Foot(6)

// This works because of the implicit conversion
if (distance > heightRequirement) println("Tall enough!")
else println("Too short!")
```

Note: Implicit conversions should be used sparingly as they can make code harder to understand.

## Implicit Classes

Implicit classes allow you to add methods to existing types without modifying their source code.

```scala
object StringExtensions {
  implicit class RichString(val s: String) extends AnyVal {
    def isPalindrome: Boolean = s == s.reverse
  }
}

import StringExtensions._

println("racecar".isPalindrome) // true
println("hello".isPalindrome)   // false
```

## Type Classes

Type classes provide a way to add behavior to types after they are defined. They consist of three parts:

1. The type class itself
2. Instances of the type class
3. Interface methods that use the type class

Here's an example of a `Show` type class:

```scala
// 1. Type class definition
trait Show[A] {
  def show(a: A): String
}

// 2. Type class instances
object Show {
  implicit val intShow: Show[Int] = new Show[Int] {
    def show(a: Int): String = a.toString
  }
  
  implicit val boolShow: Show[Boolean] = new Show[Boolean] {
    def show(a: Boolean): String = if (a) "yes" else "no"
  }
}

// 3. Interface methods
object ShowSyntax {
  implicit class ShowOps[A](val a: A) extends AnyVal {
    def show(implicit sh: Show[A]): String = sh.show(a)
  }
}

// Usage
import ShowSyntax._

println(42.show)    // 42
println(true.show)  // yes
```

## Context Bounds

Context bounds provide a shorthand syntax for declaring implicit parameters of a type class.

```scala
def printShow[A: Show](a: A): Unit = {
  println(a.show)
}

// This is equivalent to:
// def printShow[A](a: A)(implicit sh: Show[A]): Unit = {
//   println(sh.show(a))
// }

printShow(42)    // 42
printShow(true)  // yes
```

## Implementing a Type Class

Let's implement a more complex type class for sorting:

```scala
trait Ordering[A] {
  def compare(x: A, y: A): Int
}

object Ordering {
  implicit val intOrdering: Ordering[Int] = new Ordering[Int] {
    def compare(x: Int, y: Int): Int = x.compare(y)
  }
  
  implicit val stringOrdering: Ordering[String] = new Ordering[String] {
    def compare(x: String, y: String): Int = x.compare(y)
  }
}

def sort[A](xs: List[A])(implicit ord: Ordering[A]): List[A] = {
  xs.sortWith((x, y) => ord.compare(x, y) < 0)
}

// Using context bounds
def sortWithContextBound[A: Ordering](xs: List[A]): List[A] = {
  implicit val ord = implicitly[Ordering[A]]
  sort(xs)
}

println(sort(List(3, 1, 4, 1, 5, 9, 2, 6, 5, 3)))
// [1, 1, 2, 3, 3, 4, 5, 5, 6, 9]

println(sortWithContextBound(List("banana", "apple", "cherry")))
// [apple, banana, cherry]
```

## Practice Exercises

1. Create an implicit class that adds a `squared` method to `Int`.

2. Implement a `Monoid` type class with instances for `Int` (under addition) and `String` (under concatenation).

3. Write a generic `sum` function that works with any type that has a `Monoid` instance.

4. Create a `JsonWriter` type class with instances for `Int`, `String`, and `List[A]` (where `A` has a `JsonWriter` instance).

## Advanced Example: Combining Implicits and Type Classes

Let's create a more complex example that combines various aspects of implicits and type classes:

```scala
// Type class for JSON serialization
trait JsonWriter[A] {
  def toJson(value: A): String
}

object JsonWriter {
  implicit val stringWriter: JsonWriter[String] = new JsonWriter[String] {
    def toJson(value: String): String = s""""$value""""
  }
  
  implicit val intWriter: JsonWriter[Int] = new JsonWriter[Int] {
    def toJson(value: Int): String = value.toString
  }
  
  implicit def listWriter[A](implicit writer: JsonWriter[A]): JsonWriter[List[A]] = new JsonWriter[List[A]] {
    def toJson(value: List[A]): String = value.map(writer.toJson).mkString("[", ",", "]")
  }
  
  implicit def optionWriter[A](implicit writer: JsonWriter[A]): JsonWriter[Option[A]] = new JsonWriter[Option[A]] {
    def toJson(value: Option[A]): String = value.map(writer.toJson).getOrElse("null")
  }
}

// Syntax extension
object JsonSyntax {
  implicit class JsonWriterOps[A](val a: A) extends AnyVal {
    def toJson(implicit writer: JsonWriter[A]): String = writer.toJson(a)
  }
}

// Data model
case class Person(name: String, age: Int, emails: List[String])

// Type class instance for Person
object PersonJsonWriter {
  implicit val personWriter: JsonWriter[Person] = new JsonWriter[Person] {
    def toJson(person: Person): String = {
      import JsonSyntax._
      s"""{
         |  "name": ${person.name.toJson},
         |  "age": ${person.age.toJson},
         |  "emails": ${person.emails.toJson}
         |}""".stripMargin
    }
  }
}

// Usage
import JsonSyntax._
import PersonJsonWriter._

val person = Person("Alice", 30, List("alice@example.com", "alice@work.com"))
println(person.toJson)

val people = List(
  Person("Bob", 25, List("bob@example.com")),
  Person("Charlie", 35, List("charlie@example.com", "charlie@work.com"))
)
println(people.toJson)

val maybePerson: Option[Person] = Some(Person("David", 40, List("david@example.com")))
println(maybePerson.toJson)
```

This example demonstrates:
- Definition of a `JsonWriter` type class
- Implicit instances for basic types and containers (`List` and `Option`)
- Use of context bounds in the `listWriter` and `optionWriter` instances
- Syntax extensions using implicit classes
- Combining type class instances to create more complex instances (for `Person`)
- Use of implicitly defined type class instances

## Conclusion

Implicits and type classes are powerful features in Scala that enable you to write extensible and flexible code. Implicits allow you to pass parameters and perform conversions automatically, while type classes provide a way to add behavior to types after they are defined.

These concepts are widely used in Scala libraries and frameworks, enabling powerful abstractions and clean, expressive code. As you continue to work with Scala, you'll find that mastering implicits and type classes opens up new possibilities for designing flexible and reusable components.

In the next lesson, we'll explore concurrent programming in Scala, where we'll see how some of these functional programming concepts can be applied to writing safe and efficient concurrent code.