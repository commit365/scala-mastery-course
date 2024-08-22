# Functional Programming Concepts in Scala

Functional programming is a core paradigm in Scala. This lesson covers key functional programming concepts and their implementation in Scala.

## Pure Functions

Pure functions are functions that:
1. Always produce the same output for the same input
2. Have no side effects

Example of a pure function:

```scala
def add(a: Int, b: Int): Int = a + b

println(add(3, 4)) // Always outputs: 7
```

Example of an impure function:

```scala
var count = 0
def incrementAndGet(): Int = {
  count += 1
  count
}

println(incrementAndGet()) // Outputs: 1
println(incrementAndGet()) // Outputs: 2
```

## Immutability

Immutability means that once an object is created, it cannot be changed. In Scala, we use `val` for immutable variables and `var` for mutable ones.

```scala
val immutableList = List(1, 2, 3)
// This creates a new list, doesn't modify the original
val newList = 0 :: immutableList
println(immutableList) // List(1, 2, 3)
println(newList)       // List(0, 1, 2, 3)

var mutableVar = 5
mutableVar += 1
println(mutableVar) // 6
```

## Referential Transparency

An expression is referentially transparent if it can be replaced with its value without changing the program's behavior.

```scala
def pureFunction(x: Int): Int = x * 2

val result = pureFunction(5) + pureFunction(5)
// This is equivalent to:
val sameResult = 10 + 10

println(result == sameResult) // true
```

## Recursion

Recursion is a technique where a function calls itself to solve a problem.

```scala
def factorial(n: Int): Int = {
  if (n <= 1) 1
  else n * factorial(n - 1)
}

println(factorial(5)) // 120
```

## Tail Recursion

Tail recursion is a special form of recursion where the recursive call is the last operation in the function. Scala can optimize tail-recursive functions to prevent stack overflow.

```scala
import scala.annotation.tailrec

@tailrec
def factorialTailRec(n: Int, acc: Int = 1): Int = {
  if (n <= 1) acc
  else factorialTailRec(n - 1, n * acc)
}

println(factorialTailRec(5)) // 120
```

The `@tailrec` annotation ensures that the function is tail-recursive. If it's not, the compiler will throw an error.

## Trampolining

Trampolining is a technique to simulate tail-recursion when direct tail-recursion is not possible. It's useful for mutually recursive functions or when you need to make tail-recursive calls to other methods.

Scala provides the `TailRec` type and `tailcall` function in the `scala.util.control.TailCalls` package for trampolining.

```scala
import scala.util.control.TailCalls._

def isEven(n: Int): TailRec[Boolean] = {
  if (n == 0) done(true)
  else tailcall(isOdd(n - 1))
}

def isOdd(n: Int): TailRec[Boolean] = {
  if (n == 0) done(false)
  else tailcall(isEven(n - 1))
}

println(isEven(10000).result) // true
println(isOdd(10001).result)  // true
```

## Practice Exercises

1. Write a pure function to calculate the nth Fibonacci number.

2. Implement a tail-recursive function to reverse a list.

3. Create a function that uses trampolining to determine if a number is prime.

4. Write a recursive function to flatten a nested list structure.

## Advanced Example: Combining Functional Concepts

Let's create a more complex example that combines various functional programming concepts:

```scala
import scala.annotation.tailrec
import scala.util.control.TailCalls._

// Define a Tree structure
sealed trait Tree[+A]
case class Leaf[A](value: A) extends Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]

object TreeOperations {
  // Pure function to create a tree
  def createTree(depth: Int): Tree[Int] = {
    if (depth <= 0) Leaf(1)
    else Branch(createTree(depth - 1), createTree(depth - 1))
  }

  // Tail-recursive function to count leaves
  def countLeaves[A](tree: Tree[A]): Int = {
    @tailrec
    def count(trees: List[Tree[A]], acc: Int): Int = trees match {
      case Nil => acc
      case Leaf(_) :: rest => count(rest, acc + 1)
      case Branch(left, right) :: rest => count(left :: right :: rest, acc)
    }
    count(List(tree), 0)
  }

  // Trampolined function to find tree depth
  def depth[A](tree: Tree[A]): TailRec[Int] = tree match {
    case Leaf(_) => done(1)
    case Branch(left, right) =>
      for {
        leftDepth <- tailcall(depth(left))
        rightDepth <- tailcall(depth(right))
      } yield math.max(leftDepth, rightDepth) + 1
  }

  // Higher-order function to transform tree
  def map[A, B](tree: Tree[A])(f: A => B): Tree[B] = tree match {
    case Leaf(value) => Leaf(f(value))
    case Branch(left, right) => Branch(map(left)(f), map(right)(f))
  }
}

// Usage
val tree = TreeOperations.createTree(4)
println(s"Leaf count: ${TreeOperations.countLeaves(tree)}")
println(s"Tree depth: ${TreeOperations.depth(tree).result}")

val doubledTree = TreeOperations.map(tree)(_ * 2)
println(s"Doubled tree leaf count: ${TreeOperations.countLeaves(doubledTree)}")
```

This example demonstrates:
- Pure functions (`createTree`, `map`)
- Immutable data structures (`Tree`)
- Tail recursion (`countLeaves`)
- Trampolining (`depth`)
- Higher-order functions (`map`)

It combines these concepts to work with a tree structure, showing how functional programming techniques can be applied to solve complex problems.

## Conclusion

Functional programming concepts like pure functions, immutability, and recursion are fundamental to Scala programming. They promote code that is easier to reason about, test, and parallelize. Techniques like tail recursion and trampolining allow you to write efficient recursive algorithms that can handle large inputs without stack overflow issues.

As you continue to work with Scala, you'll find that these functional programming concepts become increasingly natural and lead to cleaner, more maintainable code. In the next lesson, we'll explore more advanced functional programming concepts, building on the foundation laid here.