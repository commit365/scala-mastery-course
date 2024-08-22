# Pattern Matching and Case Classes

Pattern matching is a powerful feature in Scala that allows you to match against different patterns of data. Case classes are special classes that are optimized for use in pattern matching.

## Pattern Matching Syntax

Pattern matching in Scala is done using the `match` keyword.

```scala
def describe(x: Any): String = x match {
  case 0 => "zero"
  case i: Int => s"integer: $i"
  case s: String => s"string: $s"
  case _ => "unknown"
}

println(describe(0))     // Outputs: zero
println(describe(42))    // Outputs: integer: 42
println(describe("hello")) // Outputs: string: hello
println(describe(3.14))  // Outputs: unknown
```

## Pattern Matching with Case Classes

Case classes are particularly useful with pattern matching.

```scala
case class Person(name: String, age: Int)

def greet(person: Person): String = person match {
  case Person("Alice", _) => "Hello, Alice!"
  case Person(name, age) if age < 18 => s"Hey, young $name!"
  case Person(name, _) => s"Hello, $name!"
}

println(greet(Person("Alice", 30)))  // Outputs: Hello, Alice!
println(greet(Person("Bob", 15)))    // Outputs: Hey, young Bob!
println(greet(Person("Charlie", 40))) // Outputs: Hello, Charlie!
```

## Extractors

Extractors allow you to define custom patterns for pattern matching.

```scala
object Even {
  def unapply(n: Int): Option[Int] = if (n % 2 == 0) Some(n) else None
}

object Twice {
  def unapply(n: Int): Option[Int] = Some(n / 2)
}

def describe(n: Int): String = n match {
  case Even(Twice(x)) => s"$n is two times $x and even"
  case Even(_) => s"$n is even"
  case _ => s"$n is odd"
}

println(describe(16)) // Outputs: 16 is two times 8 and even
println(describe(10)) // Outputs: 10 is even
println(describe(15)) // Outputs: 15 is odd
```

## Custom Matchers

You can create custom matchers for more complex pattern matching scenarios.

```scala
class Email(val username: String, val domain: String)

object Email {
  def apply(email: String): Email = {
    val parts = email.split("@")
    new Email(parts(0), parts(1))
  }

  def unapply(email: Email): Option[(String, String)] = {
    Some((email.username, email.domain))
  }
}

def printEmailInfo(email: Any): Unit = email match {
  case Email(username, domain) => println(s"Username: $username, Domain: $domain")
  case _ => println("Not an email")
}

printEmailInfo(Email("user@example.com")) // Outputs: Username: user, Domain: example.com
printEmailInfo("not an email") // Outputs: Not an email
```

## Case Classes

Case classes are classes that are optimized for use in pattern matching. They automatically get several useful features.

```scala
case class Point(x: Int, y: Int)

val point = Point(1, 2)

// Automatic toString
println(point) // Outputs: Point(1,2)

// Automatic equals and hashCode
println(point == Point(1, 2)) // Outputs: true

// Automatic copy method
val point2 = point.copy(y = 3)
println(point2) // Outputs: Point(1,3)

// Pattern matching
point match {
  case Point(0, 0) => println("Origin")
  case Point(x, y) => println(s"Point($x, $y)")
}
```

## Case Objects

Case objects are similar to case classes, but they are objects (singletons) instead of classes.

```scala
sealed trait Day
case object Monday extends Day
case object Tuesday extends Day
case object Wednesday extends Day
case object Thursday extends Day
case object Friday extends Day
case object Saturday extends Day
case object Sunday extends Day

def isWeekend(day: Day): Boolean = day match {
  case Saturday | Sunday => true
  case _ => false
}

println(isWeekend(Saturday)) // Outputs: true
println(isWeekend(Monday))   // Outputs: false
```

## Sealed Traits

Sealed traits are useful for creating closed hierarchies that can be exhaustively pattern matched.

```scala
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape
case class Triangle(base: Double, height: Double) extends Shape

def area(shape: Shape): Double = shape match {
  case Circle(r) => math.Pi * r * r
  case Rectangle(w, h) => w * h
  case Triangle(b, h) => 0.5 * b * h
}

println(area(Circle(5)))        // Outputs: 78.53981633974483
println(area(Rectangle(4, 5)))  // Outputs: 20.0
println(area(Triangle(3, 4)))   // Outputs: 6.0
```

## Practice Exercises

1. Create a `BinaryTree` sealed trait with `Node` and `Leaf` case classes. Implement a function to calculate the depth of the tree using pattern matching.

2. Define a custom extractor for validating email addresses, and use it in a pattern matching expression.

3. Implement a simple expression evaluator using case classes for different types of expressions (e.g., `Add`, `Subtract`, `Multiply`) and pattern matching to evaluate them.

4. Create a sealed trait hierarchy for a card game, with case objects for ranks and suits. Implement functions to compare cards using pattern matching.

## Advanced Example: Combining Pattern Matching and Case Classes

Let's create a more complex example that combines various aspects of pattern matching and case classes:

```scala
// Sealed trait for expressions
sealed trait Expr
case class Num(value: Double) extends Expr
case class Add(left: Expr, right: Expr) extends Expr
case class Subtract(left: Expr, right: Expr) extends Expr
case class Multiply(left: Expr, right: Expr) extends Expr
case class Divide(left: Expr, right: Expr) extends Expr

// Custom extractor for even numbers
object Even {
  def unapply(n: Double): Option[Double] = if (n % 2 == 0) Some(n) else None
}

// Evaluator using pattern matching
def evaluate(expr: Expr): Double = expr match {
  case Num(n) => n
  case Add(l, r) => evaluate(l) + evaluate(r)
  case Subtract(l, r) => evaluate(l) - evaluate(r)
  case Multiply(l, r) => evaluate(l) * evaluate(r)
  case Divide(l, r) => 
    val denominator = evaluate(r)
    if (denominator != 0) evaluate(l) / denominator
    else throw new ArithmeticException("Division by zero")
}

// Simplifier using pattern matching
def simplify(expr: Expr): Expr = expr match {
  case Add(Num(0), r) => simplify(r)
  case Add(l, Num(0)) => simplify(l)
  case Multiply(Num(1), r) => simplify(r)
  case Multiply(l, Num(1)) => simplify(l)
  case Multiply(Num(0), _) => Num(0)
  case Multiply(_, Num(0)) => Num(0)
  case Add(l, r) => Add(simplify(l), simplify(r))
  case Subtract(l, r) => Subtract(simplify(l), simplify(r))
  case Multiply(l, r) => Multiply(simplify(l), simplify(r))
  case Divide(l, r) => Divide(simplify(l), simplify(r))
  case n @ Num(Even(_)) => n
  case Num(n) => Num(n)
}

// Usage
val expr = Add(Multiply(Num(2), Num(3)), Subtract(Num(5), Num(1)))
println(s"Original: $expr")
println(s"Evaluated: ${evaluate(expr)}")
println(s"Simplified: ${simplify(expr)}")

val complexExpr = Add(Multiply(Num(0), Num(5)), Add(Num(0), Multiply(Num(1), Num(10))))
println(s"Complex: $complexExpr")
println(s"Simplified: ${simplify(complexExpr)}")
```

This example demonstrates:
- Sealed traits and case classes for representing expressions
- Pattern matching for evaluation and simplification of expressions
- Custom extractor (`Even`) used in pattern matching
- Nested pattern matching
- Guard conditions in pattern matching

## Conclusion

Pattern matching and case classes are powerful features in Scala that work together to provide expressive and type-safe ways of working with data. Pattern matching allows for concise and readable code when dealing with complex data structures, while case classes provide a convenient way to create immutable data types that work well with pattern matching.

These features are fundamental to idiomatic Scala programming and are widely used in Scala libraries and frameworks. As you continue to work with Scala, you'll find that mastering pattern matching and case classes will greatly enhance your ability to write clear, concise, and robust code.

In the next lesson, we'll explore Scala collections, which provide a rich set of immutable and mutable data structures that work seamlessly with the pattern matching and functional programming features we've covered so far.