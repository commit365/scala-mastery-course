# Scala Basics: Syntax and Data Types

## Variables, Values, and Type Inference

In Scala, you can declare variables using `var` and immutable values using `val`. Scala also features powerful type inference, often allowing you to omit explicit type declarations.

### Variables (var)

Variables declared with `var` can be reassigned.

```scala
var x = 5
x = 10 // This is allowed
```

### Values (val)

Values declared with `val` are immutable and cannot be reassigned.

```scala
val y = 20
// y = 30 // This would cause a compilation error
```

### Type Inference

Scala can often infer types, but you can also declare them explicitly:

```scala
val inferredInt = 42 // Type is inferred as Int
val explicitDouble: Double = 3.14
```

## Basic Data Types

Scala provides several basic data types:

1. **Int**: 32-bit signed integer
   ```scala
   val age: Int = 30
   ```

2. **Long**: 64-bit signed integer
   ```scala
   val population: Long = 7800000000L
   ```

3. **Float**: 32-bit IEEE 754 single-precision float
   ```scala
   val pi: Float = 3.14f
   ```

4. **Double**: 64-bit IEEE 754 double-precision float
   ```scala
   val e: Double = 2.71828
   ```

5. **Boolean**: true or false
   ```scala
   val isScalaFun: Boolean = true
   ```

6. **Char**: 16-bit unsigned Unicode character
   ```scala
   val initial: Char = 'S'
   ```

7. **String**: A sequence of characters
   ```scala
   val name: String = "Scala"
   ```

## Operations

Scala supports standard arithmetic, comparison, and logical operations.

### Arithmetic Operations

```scala
val sum = 5 + 3
val difference = 10 - 4
val product = 6 * 7
val quotient = 20 / 4
val remainder = 15 % 4
```

### Comparison Operations

```scala
val isEqual = (5 == 5)
val isNotEqual = (5 != 4)
val isGreater = (10 > 5)
val isLess = (3 < 7)
val isGreaterOrEqual = (5 >= 5)
val isLessOrEqual = (4 <= 4)
```

### Logical Operations

```scala
val andResult = true && false
val orResult = true || false
val notResult = !true
```

## String Interpolation

Scala offers powerful string interpolation features.

### s-interpolator

The `s` interpolator allows you to embed variables and expressions directly in strings:

```scala
val name = "Alice"
val age = 30
println(s"My name is $name and I am $age years old.")
println(s"In 5 years, I'll be ${age + 5} years old.")
```

### f-interpolator

The `f` interpolator allows for printf-style formatting:

```scala
val height = 1.75
val weight = 68.5
println(f"I am $height%.2f meters tall and weigh $weight%.1f kg.")
```

### raw-interpolator

The `raw` interpolator is similar to the `s` interpolator but does not escape special characters:

```scala
println(raw"This is a raw string with a newline character: \n")
```

## Practice Exercises

1. Declare a `val` for your age and a `var` for your weight. Try reassigning both and observe the results.

2. Create variables of each basic data type and perform some operations with them.

3. Write a program that calculates the area of a circle. Use string interpolation to print the result.

4. Experiment with different string interpolators to format and display information about yourself.

## Conclusion

Understanding Scala's basic syntax and data types is crucial for building a strong foundation in the language. As you progress, you'll see how these fundamentals combine with Scala's more advanced features to create powerful and expressive code.

In the next lesson, we'll explore control structures and expressions in Scala, which will allow you to add logic and flow control to your programs.