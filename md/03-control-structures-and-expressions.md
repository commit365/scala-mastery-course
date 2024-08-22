# Control Structures and Expressions in Scala

## If-Else Expressions

In Scala, if-else statements are expressions that return a value. This allows for more concise and functional-style code.

### Basic If-Else

```scala
val x = 10
val result = if (x > 5) "Greater than 5" else "Less than or equal to 5"
println(result) // Outputs: Greater than 5
```

### Multi-Branch If-Else

```scala
val y = 15
val category = if (y < 0) "Negative"
               else if (y == 0) "Zero"
               else if (y < 10) "Small positive"
               else "Large positive"
println(category) // Outputs: Large positive
```

## Pattern Matching

Pattern matching is a powerful feature in Scala that allows you to match against different patterns of data.

### Basic Pattern Matching

```scala
val day = "Monday"
val dayType = day match {
  case "Saturday" | "Sunday" => "Weekend"
  case "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" => "Weekday"
  case _ => "Invalid day"
}
println(dayType) // Outputs: Weekday
```

### Pattern Matching with Types

```scala
def describe(x: Any): String = x match {
  case i: Int if i > 0 => "Positive integer"
  case 0 => "Zero"
  case s: String => s"A string: $s"
  case _ => "Something else"
}

println(describe(42))    // Outputs: Positive integer
println(describe("Hello")) // Outputs: A string: Hello
```

## Loops

Scala provides several ways to implement loops, including while loops and for loops.

### While Loop

```scala
var i = 0
while (i < 5) {
  println(s"i is $i")
  i += 1
}
```

### For Loop

```scala
for (i <- 0 until 5) {
  println(s"i is $i")
}
```

### Nested For Loops

```scala
for {
  i <- 1 to 3
  j <- 1 to 3
} println(s"($i, $j)")
```

## For Comprehensions

For comprehensions in Scala provide a powerful way to work with collections and are often used instead of traditional loops.

### Basic For Comprehension

```scala
val numbers = List(1, 2, 3, 4, 5)
val doubled = for (n <- numbers) yield n * 2
println(doubled) // Outputs: List(2, 4, 6, 8, 10)
```

### For Comprehension with Guards

```scala
val evenDoubled = for {
  n <- numbers
  if n % 2 == 0
} yield n * 2
println(evenDoubled) // Outputs: List(4, 8)
```

### Nested For Comprehension

```scala
val pairs = for {
  x <- List(1, 2, 3)
  y <- List('a', 'b', 'c')
} yield (x, y)
println(pairs) // Outputs: List((1,a), (1,b), (1,c), (2,a), (2,b), (2,c), (3,a), (3,b), (3,c))
```

## The Yield Keyword

The `yield` keyword is used in for comprehensions to create a new collection based on the results of the comprehension.

```scala
val squares = for (i <- 1 to 5) yield i * i
println(squares) // Outputs: Vector(1, 4, 9, 16, 25)
```

## Practice Exercises

1. Write a function that uses pattern matching to classify numbers as "positive", "negative", or "zero".

2. Create a for comprehension that generates all pairs of numbers (i, j) where 1 ≤ i < j ≤ 5.

3. Use a while loop to compute the factorial of a number.

4. Write a for comprehension that filters a list of strings, keeping only those that start with a vowel, and converts them to uppercase.

## Advanced Example: Combining Concepts

Let's combine several concepts we've learned in a more complex example:

```scala
def processNumbers(numbers: List[Int]): List[String] = {
  for {
    n <- numbers
    if n % 2 == 0 || n % 3 == 0
    result = n match {
      case x if x % 2 == 0 && x % 3 == 0 => s"$x is divisible by both 2 and 3"
      case x if x % 2 == 0 => s"$x is even"
      case x if x % 3 == 0 => s"$x is divisible by 3"
    }
  } yield result
}

val nums = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12)
val processed = processNumbers(nums)
processed.foreach(println)
```

This example demonstrates:
- A for comprehension with a guard
- Pattern matching within the for comprehension
- Yield to create a new list
- Use of if-else expressions within pattern matching

## Conclusion

Control structures and expressions in Scala offer powerful tools for controlling program flow and working with data. The functional nature of if-else expressions and the versatility of pattern matching provide concise and expressive ways to write code. For comprehensions and the yield keyword offer elegant solutions for working with collections, often replacing traditional loops with more readable and maintainable code.

As you continue to work with Scala, you'll find that these constructs become integral to writing clean, efficient, and expressive code. In the next lesson, we'll explore functions in Scala, which will build upon these concepts and introduce new ways to structure your code.