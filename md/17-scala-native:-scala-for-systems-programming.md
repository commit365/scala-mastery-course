# Scala Native: Scala for Systems Programming

Scala Native is an optimizing ahead-of-time compiler and lightweight managed runtime designed specifically for Scala. It enables developers to write fast, elegant programs that run directly on bare metal without the overhead of the JVM. This lesson covers how to compile Scala to native code, performance considerations, and how to interoperate with C libraries for low-level programming.

## Compiling Scala to Native Code

### Setting Up Scala Native

To start using Scala Native, you need to set up your project with the necessary dependencies. Here’s how to create a simple Scala Native project using sbt.

1. **Create a new sbt project**:

   Create a new directory for your project and navigate into it:

   ```bash
   mkdir my-scalanative-project
   cd my-scalanative-project
   ```

2. **Create a `build.sbt` file**:

   Add the following content to your `build.sbt` file:

   ```scala
   enablePlugins(ScalaNativePlugin)

   name := "MyScalaNativeProject"

   version := "0.1.0"

   scalaVersion := "2.13.6"

   libraryDependencies += "org.scala-native" %% "scala-native" % "0.5.4"
   ```

3. **Create a `src/main/scala` directory**:

   Create the necessary directory structure for your Scala Native code:

   ```bash
   mkdir -p src/main/scala
   ```

4. **Write your Scala Native code**:

   Create a file named `Main.scala` in `src/main/scala` with the following content:

   ```scala
   import scala.scalanative.unsafe._
   import scala.scalanative.unsigned._

   object Main {
     def main(args: Array[String]): Unit = {
       val message = c"Hello, Scala Native!"
       println(message)
     }
   }
   ```

5. **Compile and run your project**:

   Use sbt to compile your project and generate the native executable:

   ```bash
   sbt nativeCompile
   ```

   You can run the generated executable from the `target/scala-2.13` directory.

## Performance Considerations

One of the main advantages of Scala Native is its performance characteristics, which include:

1. **Instant Startup Time**: Scala Native compiles to native machine code using LLVM, resulting in faster startup times compared to JVM-based applications.

2. **Low-Level Control**: Scala Native provides low-level primitives, allowing you to control memory management, pointers, and other system-level features.

3. **Optimizations**: The Scala Native compiler applies various optimizations during the compilation process, improving runtime performance.

### Example of Performance Optimization

You can optimize memory usage and performance by using stack allocation for small objects:

```scala
import scala.scalanative.unsafe._

@main def main(): Unit = {
  val vec = stackalloc[CStruct3[Double, Double, Double]]()
  vec._1 = 10.0
  vec._2 = 20.0
  vec._3 = 30.0
  println(vec._1 + vec._2 + vec._3) // Outputs: 60.0
}
```

## Interoperability with C Libraries

Scala Native allows you to call C libraries directly, enabling you to leverage existing C code and libraries in your Scala applications.

### Importing C Libraries

You can use the `@extern` annotation to define external C functions in Scala Native.

```scala
import scala.scalanative.unsafe._
import scala.scalanative.libc.stdlib._

@extern
object MyLib {
  def malloc(size: CSize): Ptr[Byte] = extern
  def free(ptr: Ptr[Byte]): Unit = extern
}

@main def main(): Unit = {
  val size = 64.toCSize
  val ptr = MyLib.malloc(size) // Allocate memory
  if (ptr != null) {
    println(s"Allocated $size bytes at address: $ptr")
    MyLib.free(ptr) // Free memory
  }
}
```

### Using C Libraries

You can call C functions directly from your Scala Native code, allowing for seamless integration with existing C libraries.

```scala
@extern
object MathLib {
  def sin(x: Double): Double = extern
}

@main def main(): Unit = {
  val angle = 0.5
  val result = MathLib.sin(angle)
  println(s"Sin($angle) = $result")
}
```

### Linking with C Libraries

When using external C libraries, you may need to specify the libraries to link against in your `build.sbt` file.

```scala
// Example of linking with a C library
nativeLinking := Seq("m") // Link with the math library
```

## Low-Level Programming

Scala Native provides low-level programming capabilities, allowing you to work directly with pointers, arrays, and memory management.

### Working with Pointers

You can allocate and manipulate pointers directly in Scala Native.

```scala
@main def main(): Unit = {
  val array = stackalloc[Double](5) // Allocate an array of 5 doubles
  for (i <- 0 until 5) {
    array(i) = i.toDouble
  }

  // Print the array values
  for (i <- 0 until 5) {
    println(array(i)) // Outputs: 0.0, 1.0, 2.0, 3.0, 4.0
  }
}
```

### Memory Management

Scala Native allows you to manage memory manually, giving you fine-grained control over resource allocation.

```scala
@main def main(): Unit = {
  val size = 10.toCSize
  val ptr = MyLib.malloc(size) // Allocate memory
  if (ptr != null) {
    // Use the allocated memory
    // ...
    MyLib.free(ptr) // Free the allocated memory
  }
}
```

## Practice Exercises

1. Create a Scala Native application that allocates an array of integers, initializes it with values, and prints the values to the console.

2. Implement a Scala Native program that calls a C function from a library (e.g., `libm` for mathematical functions) and uses it to compute the square root of a number.

3. Write a Scala Native application that uses pointers to manipulate a C struct.

4. Create a simple command-line tool in Scala Native that reads input from the user and performs basic arithmetic operations.

## Advanced Example: Combining Features

Let’s create a more complex example that combines various features of Scala Native, including C interoperability and low-level programming:

```scala
import scala.scalanative.unsafe._
import scala.scalanative.libc.stdlib._

// Define a C struct
@CStruct
class Point(val x: Double, val y: Double) extends CStruct3[Double, Double, Double]

// Define external C functions
@extern
object GeometryLib {
  def distance(p1: Ptr[Point], p2: Ptr[Point]): Double = extern
}

@main def main(): Unit = {
  // Allocate two points
  val point1 = stackalloc[Point]()
  val point2 = stackalloc[Point]()

  // Initialize points
  point1._1 = 3.0 // x
  point1._2 = 4.0 // y
  point2._1 = 0.0 // x
  point2._2 = 0.0 // y

  // Calculate distance
  val dist = GeometryLib.distance(point1, point2)
  println(s"Distance between points: $dist")
}
```

In this example:
- We define a C struct called `Point`.
- We declare an external C function `distance` that calculates the distance between two points.
- We allocate memory for the points, initialize them, and call the C function to compute the distance.

## Conclusion

Scala Native is a powerful tool for systems programming that allows you to write high-performance applications without the overhead of the JVM. By compiling Scala to native code, you gain instant startup times and low-level control over memory management.

With the ability to interoperate with C libraries and perform low-level programming, Scala Native opens up new possibilities for building efficient applications. By mastering these concepts, you can leverage Scala's expressive syntax while working close to the metal, making it suitable for a wide range of applications, from system utilities to high-performance services. In the next lesson, we will explore more advanced topics in Scala, including best practices for building scalable applications.
