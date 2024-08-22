# Functions in Scala

## Function Declaration

In Scala, functions are first-class citizens. They can be assigned to variables, passed as arguments, and returned from other functions.

### Basic Function Declaration

```scala
def greet(name: String): String = {
  s"Hello, $name!"
}

println(greet("Alice")) // Outputs: Hello, Alice!
```

### Single-Expression Functions

For functions with a single expression, you can omit the curly braces and the `return` keyword:

```scala
def square(x: Int): Int = x * x

println(square(5)) // Outputs: 25
```

### Default and Named Parameters

Scala supports default and named parameters:

```scala
def greetWithTitle(name: String, title: String = "Mr."): String = {
  s"Hello, $title $name!"
}

println(greetWithTitle("Smith")) // Outputs: Hello, Mr. Smith!
println(greetWithTitle("Johnson", "Dr.")) // Outputs: Hello, Dr. Johnson!
println(greetWithTitle(title = "Ms.", name = "Brown")) // Outputs: Hello, Ms. Brown!
```

## Anonymous Functions (Lambda Expressions)

Anonymous functions, also known as lambda expressions, are functions without a name.

```scala
val add = (x: Int, y: Int) => x + y
println(add(3, 4)) // Outputs: 7

val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(x => x * 2)
println(doubled) // Outputs: List(2, 4, 6, 8, 10)
```

## Partial Functions

Partial functions are functions that are only defined for a subset of possible inputs.

```scala
val divide: PartialFunction[Int, Int] = {
  case d: Int if d != 0 => 100 / d
}

println(divide.isDefinedAt(0)) // Outputs: false
println(divide.isDefinedAt(5)) // Outputs: true
println(divide(5)) // Outputs: 20
```

## Higher-Order Functions

Higher-order functions are functions that take other functions as parameters or return functions.

```scala
def operate(x: Int, y: Int, f: (Int, Int) => Int): Int = f(x, y)

val sum = operate(5, 3, (a, b) => a + b)
val product = operate(5, 3, (a, b) => a * b)

println(sum) // Outputs: 8
println(product) // Outputs: 15
```

## Closures

A closure is a function that captures the state of its surrounding scope.

```scala
def makeAdder(x: Int): Int => Int = {
  (y: Int) => x + y
}

val add5 = makeAdder(5)
println(add5(3)) // Outputs: 8
println(add5(7)) // Outputs: 12
```

## Currying

Currying is the technique of converting a function with multiple arguments into a sequence of functions, each with a single argument.

```scala
def multiply(x: Int)(y: Int): Int = x * y

val timesFive = multiply(5)_
println(timesFive(3)) // Outputs: 15

// Equivalent to:
println(multiply(5)(3)) // Outputs: 15
```

## Function Composition

Scala allows you to compose functions using the `andThen` or `compose` methods.

```scala
val double = (x: Int) => x * 2
val addOne = (x: Int) => x + 1

val doubleThenAddOne = double andThen addOne
val addOneThenDouble = double compose addOne

println(doubleThenAddOne(3)) // Outputs: 7
println(addOneThenDouble(3)) // Outputs: 8
```

## Practice Exercises

1. Write a higher-order function that takes a list of integers and a function, and applies the function to each element of the list.

2. Create a curried function that takes two strings and an integer, and repeats the concatenation of the strings the given number of times.

3. Implement a partial function that calculates the square root of its input, but is only defined for non-negative numbers.

4. Write a closure that generates a sequence of power functions (square, cube, etc.) based on an input exponent.

## Advanced Example: Combining Concepts

Let's combine several functional programming concepts in a more complex example:

```scala
// Higher-order function that returns a closure
def createFormatter(prefix: String, suffix: String): String => String = {
  (s: String) => prefix + s + suffix
}

// Partial function for processing strings
val processString: PartialFunction[String, String] = {
  case s if s.nonEmpty => s.capitalize
}

// Curried function for repeating strings
def repeat(n: Int)(s: String): String = s * n

// Composing functions
val format = createFormatter("<<", ">>")
val process = processString.lift // Convert partial function to total function
val repeatTwice = repeat(2)_

// Combine all operations
val complexOperation = (s: String) => repeatTwice(format(process(s).getOrElse("EMPTY")))

// Test the complex operation
println(complexOperation("hello")) // Outputs: <<Hello>><<Hello>>
println(complexOperation("")) // Outputs: <<EMPTY>><<EMPTY>>
```

This example demonstrates:
- Higher-order functions
- Closures
- Partial functions
- Currying
- Function composition

It creates a complex operation that formats a string (if it's not empty), wraps it in brackets, capitalizes it, and then repeats it twice.

## Conclusion

Functions in Scala are powerful and flexible, allowing for expressive and concise code. The ability to treat functions as first-class citizens, combined with features like higher-order functions, closures, and currying, enables a functional programming style that can lead to more modular and reusable code.

As you continue to work with Scala, you'll find that mastering these functional programming concepts will greatly enhance your ability to write clean, efficient, and maintainable code. In the next lesson, we'll explore object-oriented programming in Scala, which complements these functional programming features to provide a hybrid paradigm.