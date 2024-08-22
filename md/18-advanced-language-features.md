# Advanced Language Features in Scala

Scala provides powerful advanced language features that enable developers to write expressive and flexible code. This lesson covers macros, quasiquotes, metaprogramming, dependent types, phantom types, and type-level programming.

## Macros

Macros allow you to generate code at compile time, enabling powerful metaprogramming capabilities. They can be used to automate repetitive tasks, enforce constraints, or optimize performance.

### Creating a Simple Macro

To create a macro, you need to define a method annotated with `@macro` and implement the logic for code generation.

1. **Add the macro library dependency** to your `build.sbt`:

   ```sbt
   libraryDependencies += "org.scalamacros" %% "paradise" % "2.1.1" cross CrossVersion.full
   ```

2. **Create a macro**:

   Here’s an example of a simple macro that generates a method to print the name of a variable:

   ```scala
   import scala.language.experimental.macros
   import scala.reflect.macros.blackbox.Context

   object MacroExample {
     def printName(variable: Any): Unit = macro printNameImpl

     def printNameImpl(c: Context)(variable: c.Expr[Any]): c.Expr[Unit] = {
       import c.universe._
       val name = showCode(variable.tree)
       reify {
         println(s"Variable name: $name")
       }
     }
   }

   // Usage
   object Main extends App {
     val x = 42
     MacroExample.printName(x) // Outputs: Variable name: x
   }
   ```

### Limitations of Macros

- Macros can make code harder to understand and maintain.
- They can lead to complex compilation errors if not used carefully.
- Macros are a compile-time feature and cannot be used in runtime scenarios.

## Quasiquotes

Quasiquotes provide a way to manipulate and generate code using a concise and readable syntax. They are often used in conjunction with macros for metaprogramming.

### Using Quasiquotes

Quasiquotes allow you to construct and deconstruct abstract syntax trees (ASTs) in a more readable manner.

```scala
import scala.reflect.macros.blackbox.Context
import scala.language.experimental.macros

object QuasiquoteExample {
  def printTree(tree: Any): Unit = macro printTreeImpl

  def printTreeImpl(c: Context)(tree: c.Expr[Any]): c.Expr[Unit] = {
    import c.universe._
    val treeString = showCode(tree.tree)
    reify {
      println(s"Tree: $treeString")
    }
  }
}

// Usage
object Main extends App {
  QuasiquoteExample.printTree(1 + 2) // Outputs: Tree: 1 + 2
}
```

### Benefits of Quasiquotes

- They provide a more readable way to manipulate code.
- They simplify the process of working with ASTs.
- They can be used to generate boilerplate code automatically.

## Metaprogramming

Metaprogramming in Scala allows you to write code that generates or manipulates other code. This can be achieved through macros and quasiquotes.

### Example of Metaprogramming

Here’s an example of using macros to generate a simple logging mechanism:

```scala
import scala.language.experimental.macros
import scala.reflect.macros.blackbox.Context

object Logger {
  def log(message: String): Unit = macro logImpl

  def logImpl(c: Context)(message: c.Expr[String]): c.Expr[Unit] = {
    import c.universe._
    reify {
      println(s"[LOG] ${message.splice}")
    }
  }
}

// Usage
object Main extends App {
  Logger.log("This is a log message!") // Outputs: [LOG] This is a log message!
}
```

## Dependent Types

Dependent types allow types to depend on values, enabling more expressive type systems. This feature can help enforce invariants in your code.

### Example of Dependent Types

```scala
class Vector[T](val elements: Array[T]) {
  def apply(i: Int): T = elements(i)
}

class SizedVector[T](elements: Array[T]) {
  require(elements.length > 0, "Vector must be non-empty")
  val vector: Vector[T] = new Vector(elements)
}

val sizedVector = new SizedVector(Array(1, 2, 3))
println(sizedVector.vector(0)) // Outputs: 1
```

In this example, the `SizedVector` class ensures that the underlying `Vector` is non-empty at compile time.

## Phantom Types

Phantom types are types that do not have any runtime representation but can be used to enforce compile-time constraints. They are useful for adding type safety without affecting performance.

### Example of Phantom Types

```scala
class SafeString[S]

object SafeString {
  def create(s: String): SafeString[String] = new SafeString[String]
}

def processString(s: SafeString[String]): Unit = {
  println("Processing safe string")
}

// Usage
val safeStr = SafeString.create("Hello")
processString(safeStr) // Outputs: Processing safe string
```

In this example, `SafeString` is a phantom type that provides type safety without adding any runtime overhead.

## Type-Level Programming

Type-level programming allows you to perform computations at the type level, enabling more expressive and flexible code.

### Example of Type-Level Programming

You can use type-level programming to create type-safe data structures.

```scala
trait Nat
class Zero extends Nat
class Succ[N <: Nat] extends Nat

type One = Succ[Zero]
type Two = Succ[One]
type Three = Succ[Two]

// Type-level addition
trait Plus[A <: Nat, B <: Nat]
object Plus {
  type Aux[A <: Nat, B <: Nat, C <: Nat] = Plus[A, B] { type Out = C }

  implicit def zeroPlus[B <: Nat]: Plus.Aux[Zero, B, B] = new Plus[Zero, B] {}
  implicit def succPlus[A <: Nat, B <: Nat](implicit plus: Plus.Aux[A, B, C]): Plus.Aux[Succ[A], B, Succ[C]] = new Plus[Succ[A], B] {}
}
```

In this example, we define natural numbers using types and perform type-level addition.

## Practice Exercises

1. Create a macro that generates a method to log the execution time of a function.

2. Implement a quasiquote that transforms a simple expression into a more complex one.

3. Write a Scala program that uses dependent types to create a type-safe stack.

4. Implement a phantom type that enforces a non-empty list at compile time.

5. Explore type-level programming by creating a type-safe representation of a binary tree.

## Advanced Example: Combining Features

Let’s create a more complex example that combines macros, quasiquotes, and type-level programming:

```scala
import scala.language.experimental.macros
import scala.reflect.macros.blackbox.Context

object MacroUtils {
  def logMethodExecutionTime[A](method: => A): A = macro logMethodExecutionTimeImpl[A]

  def logMethodExecutionTimeImpl[A: c.WeakTypeTag](c: Context)(method: c.Expr[A]): c.Expr[A] = {
    import c.universe._
    val start = q"System.nanoTime()"
    val end = q"System.nanoTime()"
    val result = q"""
      val startTime = $start
      val result = $method
      val endTime = $end
      println(s"Execution time: ${endTime - startTime} ns")
      result
    """
    c.Expr[A](result)
  }
}

// Usage
object Main extends App {
  def someComputation(): Int = {
    Thread.sleep(1000) // Simulate a long computation
    42
  }

  val result = MacroUtils.logMethodExecutionTime(someComputation())
  println(s"Result: $result") // Outputs: Result: 42
}
```

In this example:
- We create a macro that logs the execution time of a method.
- We use quasiquotes to generate the logging code dynamically.

## Conclusion

Advanced language features in Scala, such as macros, quasiquotes, dependent types, phantom types, and type-level programming, empower developers to write expressive and flexible code. These features enable metaprogramming, compile-time checks, and type-safe abstractions, enhancing the robustness and maintainability of your applications.

By mastering these advanced concepts, you can leverage Scala's full potential and create powerful, type-safe, and efficient programs. In the next lesson, we will explore best practices for building scalable applications in Scala, focusing on design patterns and architectural principles.