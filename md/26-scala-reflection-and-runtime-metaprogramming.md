# Scala Reflection and Runtime Metaprogramming

Scala reflection allows you to inspect and manipulate classes, methods, and objects at runtime. This capability is useful for dynamic programming, where you may need to work with types and structures that are not known until runtime. Additionally, Scala supports bytecode manipulation and abstract syntax tree (AST) transformations for advanced metaprogramming. This lesson covers the basics of using reflection and runtime metaprogramming in Scala.

## Using Runtime Reflection for Dynamic Programming

### What is Reflection?

Reflection is the ability of a program to examine and modify its own structure and behavior at runtime. In Scala, reflection is provided through the `scala.reflect` package.

### Basic Reflection Example

Here’s a simple example of using reflection to inspect a class and its members:

```scala
import scala.reflect.runtime.universe._

case class Person(name: String, age: Int)

object ReflectionExample extends App {
  // Get the type of the Person class
  val personType = typeOf[Person]

  // Print the class name
  println(s"Class Name: ${personType.typeSymbol.name}")

  // Print the members of the class
  println("Members:")
  personType.decls.foreach { member =>
    println(s"- ${member.name}: ${member.typeSignature}")
  }
}
```

### Creating Instances Dynamically

You can use reflection to create instances of classes dynamically:

```scala
object DynamicInstanceExample extends App {
  val personClass = classOf[Person]
  val constructor = personClass.getConstructor(classOf[String], classOf[Int])
  val personInstance = constructor.newInstance("Alice", 30)

  println(personInstance) // Outputs: Person(Alice,30)
}
```

### Accessing and Modifying Fields

Reflection allows you to access and modify fields of an object dynamically:

```scala
object FieldAccessExample extends App {
  val person = Person("Bob", 25)

  // Accessing fields using reflection
  val nameField = person.getClass.getDeclaredField("name")
  nameField.setAccessible(true) // Make private field accessible

  // Print original name
  println(s"Original Name: ${nameField.get(person)}") // Outputs: Original Name: Bob

  // Modify the name
  nameField.set(person, "Charlie")
  println(s"Modified Name: ${nameField.get(person)}") // Outputs: Modified Name: Charlie
}
```

## Bytecode Manipulation and AST Transformations

### Bytecode Manipulation

Bytecode manipulation involves changing the compiled bytecode of classes at runtime. This can be done using libraries like ASM or Byte Buddy, but Scala also provides some capabilities through reflection.

### Example of Bytecode Manipulation with ASM

1. **Add the ASM dependency** to your `build.sbt`:

   ```sbt
   libraryDependencies += "org.ow2.asm" % "asm" % "9.2"
   ```

2. **Create a simple class and manipulate its bytecode**:

```scala
import org.objectweb.asm._
import java.io.FileOutputStream

object BytecodeManipulationExample {
  def main(args: Array[String]): Unit = {
    val classWriter = new ClassWriter(ClassWriter.COMPUTE_FRAMES)
    classWriter.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "HelloWorld", null, "java/lang/Object", null)

    // Add a default constructor
    val constructor = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "<init>", "()V", null, null)
    constructor.visitCode()
    constructor.visitVarInsn(Opcodes.ALOAD, 0)
    constructor.visitMethodInsn(Opcodes.INVOKESPECIAL, "java/lang/Object", "<init>", "()V", false)
    constructor.visitInsn(Opcodes.RETURN)
    constructor.visitMaxs(1, 1)
    constructor.visitEnd()

    // Add a method that prints "Hello, World!"
    val method = classWriter.visitMethod(Opcodes.ACC_PUBLIC, "sayHello", "()V", null, null)
    method.visitCode()
    method.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;")
    method.visitLdcInsn("Hello, World!")
    method.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false)
    method.visitInsn(Opcodes.RETURN)
    method.visitMaxs(2, 1)
    method.visitEnd()

    classWriter.visitEnd()

    // Write the generated class to a file
    val bytecode = classWriter.toByteArray
    val fos = new FileOutputStream("HelloWorld.class")
    fos.write(bytecode)
    fos.close()
  }
}
```

### AST Transformations

Scala allows you to manipulate ASTs (Abstract Syntax Trees) for metaprogramming. This is particularly useful when creating macros.

#### Example of AST Transformation

```scala
import scala.reflect.macros.blackbox.Context
import scala.language.experimental.macros

object MacroExample {
  def logMethodCall(): Unit = macro logMethodCallImpl

  def logMethodCallImpl(c: Context)(): c.Expr[Unit] = {
    import c.universe._
    val methodName = c.enclosingMethod.name
    reify {
      println(s"Method called: ${methodName.toString}")
    }
  }
}

// Usage
object Main extends App {
  MacroExample.logMethodCall() // Outputs: Method called: logMethodCall
}
```

## Performance Considerations

1. **Reflection Overhead**: Using reflection can introduce performance overhead. Avoid reflection in performance-critical paths.

2. **Bytecode Manipulation Complexity**: Bytecode manipulation can lead to complex issues if not handled carefully. Ensure thorough testing.

3. **AST Transformations**: Manipulating ASTs can result in increased compile times. Use macros judiciously and document their behavior.

## Practice Exercises

1. Create a Scala application that uses reflection to inspect a class and print its methods and fields.

2. Implement a macro that logs the execution time of a method.

3. Write a program that dynamically creates a class using bytecode manipulation and adds a method to it.

4. Use AST transformations to create a macro that generates boilerplate code for case classes.

## Advanced Example: Combining Reflection and AST Transformations

Let’s create a more complex example that combines reflection and AST transformations to generate logging code for methods automatically:

```scala
import scala.language.experimental.macros
import scala.reflect.macros.blackbox.Context

object LoggingMacro {
  def logExecution[T](method: => T): T = macro logExecutionImpl[T]

  def logExecutionImpl[T: c.WeakTypeTag](c: Context)(method: c.Expr[T]): c.Expr[T] = {
    import c.universe._
    val methodName = c.enclosingMethod.name
    val start = q"System.nanoTime()"
    val end = q"System.nanoTime()"
    
    reify {
      val startTime = start
      val result = method.splice
      val endTime = end
      println(s"Method ${methodName.toString} executed in: ${endTime - startTime} ns")
      result
    }
  }
}

// Usage
object Main extends App {
  def compute(): Int = {
    Thread.sleep(500) // Simulate a long computation
    42
  }

  val result = LoggingMacro.logExecution(compute())
  println(s"Result: $result") // Outputs the execution time and result
}
```

In this example:
- We create a macro that logs the execution time of a method.
- We use reflection to obtain the method name and measure the execution time.

## Conclusion

Scala's reflection and runtime metaprogramming capabilities provide powerful tools for dynamic programming, allowing developers to inspect and manipulate code at runtime. By leveraging bytecode manipulation and AST transformations, you can create flexible and reusable code that adapts to various scenarios.

Understanding these advanced features will enhance your ability to write expressive and maintainable Scala applications. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.