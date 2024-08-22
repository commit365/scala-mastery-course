# Best Practices and Design Patterns in Scala

In Scala, combining functional and object-oriented programming paradigms allows for expressive and maintainable code. This lesson covers best practices and design patterns in Scala, along with performance optimization techniques and benchmarking methods.

## Functional and Object-Oriented Design Patterns

### Functional Design Patterns

1. **Functor**: A functor is a type class that allows you to apply a function to a wrapped value. It provides a `map` method.

   ```scala
   trait Functor[F[_]] {
     def map[A, B](fa: F[A])(f: A => B): F[B]
   }
   ```

2. **Monad**: A monad is a type class that allows for chaining operations. It provides `flatMap` and `map`.

   ```scala
   trait Monad[F[_]] extends Functor[F] {
     def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
     def pure[A](a: A): F[A]
   }
   ```

3. **Case Classes**: Use case classes for immutable data structures. They automatically provide `equals`, `hashCode`, and `toString` methods.

   ```scala
   case class User(name: String, age: Int)
   ```

4. **Higher-Order Functions**: Functions that take other functions as parameters or return them. They promote code reuse and abstraction.

   ```scala
   def applyTwice(f: Int => Int, x: Int): Int = f(f(x))
   ```

### Object-Oriented Design Patterns

1. **Singleton**: Use the singleton pattern to ensure a class has only one instance. In Scala, you can use the `object` keyword.

   ```scala
   object Database {
     def connect(): Unit = println("Connected to the database.")
   }
   ```

2. **Factory**: The factory pattern provides a way to create objects without specifying the exact class. Use companion objects for factory methods.

   ```scala
   class Animal(val name: String)

   object Animal {
     def apply(name: String): Animal = new Animal(name)
   }
   ```

3. **Decorator**: The decorator pattern allows behavior to be added to individual objects without affecting the behavior of other objects from the same class.

   ```scala
   trait Coffee {
     def cost: Double
   }

   class SimpleCoffee extends Coffee {
     def cost: Double = 2.0
   }

   class MilkDecorator(coffee: Coffee) extends Coffee {
     def cost: Double = coffee.cost + 0.5
   }
   ```

4. **Strategy**: The strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable.

   ```scala
   trait SortingStrategy {
     def sort(data: List[Int]): List[Int]
   }

   class QuickSort extends SortingStrategy {
     def sort(data: List[Int]): List[Int] = data.sorted // Simplified for example
   }
   ```

## Performance Optimization Techniques

1. **Use Immutable Collections**: Immutable collections are generally more efficient in concurrent environments and can lead to fewer bugs.

2. **Avoid Unnecessary Boxing**: When working with primitive types, avoid boxing and unboxing, which can introduce overhead.

3. **Use Tail Recursion**: Scala optimizes tail-recursive functions to prevent stack overflow and improve performance.

   ```scala
   @scala.annotation.tailrec
   def factorial(n: Int, acc: Int = 1): Int = {
     if (n <= 1) acc
     else factorial(n - 1, n * acc)
   }
   ```

4. **Leverage Parallel Collections**: For CPU-bound tasks, consider using parallel collections to take advantage of multi-core processors.

   ```scala
   val numbers = (1 to 1000000).toList
   val sum = numbers.par.sum // Using parallel collection
   ```

5. **Profile and Benchmark**: Use profiling tools to identify bottlenecks and optimize critical sections of your code.

## Benchmarking Techniques

1. **JMH (Java Microbenchmark Harness)**: Use JMH to create accurate and reliable benchmarks for your Scala code.

   ```scala
   import org.openjdk.jmh.annotations._

   @State(Scope.Benchmark)
   class MyBenchmark {
     @Benchmark
     def testMethod(): Int = {
       // Code to benchmark
       (1 to 1000).sum
     }
   }
   ```

2. **Measure Execution Time**: For simple cases, you can measure execution time using `System.nanoTime`.

   ```scala
   val start = System.nanoTime()
   // Code to measure
   val end = System.nanoTime()
   println(s"Execution time: ${(end - start) / 1e6} ms")
   ```

3. **Use Profilers**: Tools like VisualVM, YourKit, or JProfiler can help analyze performance and memory usage.

## Practice Exercises

1. Implement a functor type class for a custom data type and demonstrate its usage.

2. Create a simple factory pattern for creating different types of shapes (e.g., Circle, Rectangle).

3. Write a decorator for a logging mechanism that adds logging functionality to an existing service.

4. Benchmark a recursive Fibonacci function and optimize it using memoization.

5. Use JMH to compare the performance of different sorting algorithms implemented in Scala.

## Advanced Example: Combining Design Patterns and Optimization

Let’s create a more complex example that combines several design patterns and optimization techniques:

```scala
// Define a trait for a calculation strategy
trait CalculationStrategy {
  def calculate(a: Int, b: Int): Int
}

// Implement concrete strategies
class Addition extends CalculationStrategy {
  def calculate(a: Int, b: Int): Int = a + b
}

class Multiplication extends CalculationStrategy {
  def calculate(a: Int, b: Int): Int = a * b
}

// Create a context that uses a strategy
class Calculator(strategy: CalculationStrategy) {
  def execute(a: Int, b: Int): Int = strategy.calculate(a, b)
}

// Use a factory to create calculators
object CalculatorFactory {
  def createCalculator(operation: String): Calculator = {
    operation match {
      case "add" => new Calculator(new Addition)
      case "multiply" => new Calculator(new Multiplication)
      case _ => throw new IllegalArgumentException("Unknown operation")
    }
  }
}

// Usage
val addCalculator = CalculatorFactory.createCalculator("add")
println(addCalculator.execute(5, 3)) // Outputs: 8

val multiplyCalculator = CalculatorFactory.createCalculator("multiply")
println(multiplyCalculator.execute(5, 3)) // Outputs: 15
```

In this example:
- We define a `CalculationStrategy` trait for different calculation strategies.
- We implement concrete strategies for addition and multiplication.
- We create a `Calculator` class that uses a strategy to perform calculations.
- We use a factory to create calculators based on the desired operation.

## Conclusion

Scala's advanced language features, such as functional and object-oriented design patterns, provide powerful tools for building scalable and maintainable applications. By following best practices and employing performance optimization techniques, you can write efficient and robust code.

Mastering these concepts enables you to leverage Scala's full potential, making it easier to design and implement complex systems. In the next lesson, we will explore additional advanced topics in Scala, focusing on building scalable applications and architectural principles.