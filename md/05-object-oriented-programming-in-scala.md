# Object-Oriented Programming in Scala

Scala combines object-oriented and functional programming paradigms. This lesson focuses on the object-oriented aspects of Scala.

## Classes

Classes in Scala are blueprints for creating objects. They can contain fields, methods, and other members.

### Basic Class Definition

```scala
class Person(var name: String, var age: Int) {
  def introduce(): Unit = {
    println(s"Hi, I'm $name and I'm $age years old.")
  }
}

val alice = new Person("Alice", 30)
alice.introduce() // Outputs: Hi, I'm Alice and I'm 30 years old.
```

### Primary Constructor

The primary constructor is part of the class definition. Parameters can be prefixed with `val` or `var` to automatically create fields.

```scala
class Book(val title: String, val author: String, var pageCount: Int)

val myBook = new Book("1984", "George Orwell", 328)
println(myBook.title) // Outputs: 1984
myBook.pageCount = 350
```

### Auxiliary Constructors

Auxiliary constructors are defined using `this` and must call another constructor as their first action.

```scala
class Person(var name: String, var age: Int) {
  def this(name: String) = this(name, 0)
  def this() = this("John Doe", 0)
}

val bob = new Person("Bob", 25)
val baby = new Person("Baby")
val unknown = new Person()
```

## Objects

Objects in Scala are singleton instances of their own implicit class. They're often used for holding utility functions or as companions to classes.

```scala
object MathUtils {
  def factorial(n: Int): Int = {
    if (n <= 1) 1 else n * factorial(n - 1)
  }
}

println(MathUtils.factorial(5)) // Outputs: 120
```

## Companion Objects

A companion object is an object with the same name as a class and is defined in the same file. It can access private members of the class.

```scala
class Circle(val radius: Double) {
  import Circle._
  def area: Double = calculateArea(radius)
}

object Circle {
  private def calculateArea(radius: Double): Double = math.Pi * radius * radius
  def apply(radius: Double): Circle = new Circle(radius)
}

val circle = Circle(5) // Using the apply method
println(circle.area) // Outputs: 78.53981633974483
```

## Traits

Traits are like interfaces in Java, but they can also contain implemented methods and fields.

```scala
trait Greeting {
  def greet(name: String): Unit
}

trait Farewell {
  def sayGoodbye(name: String): Unit = println(s"Goodbye, $name!")
}

class EnglishPerson extends Greeting with Farewell {
  def greet(name: String): Unit = println(s"Hello, $name!")
}

val person = new EnglishPerson()
person.greet("Alice") // Outputs: Hello, Alice!
person.sayGoodbye("Bob") // Outputs: Goodbye, Bob!
```

## Inheritance

Scala supports single inheritance of classes, but multiple traits can be mixed in.

```scala
class Animal(val name: String) {
  def speak(): Unit = println("Some animal sound")
}

class Dog(name: String) extends Animal(name) {
  override def speak(): Unit = println("Woof!")
}

val dog = new Dog("Buddy")
dog.speak() // Outputs: Woof!
```

## Polymorphism

Polymorphism allows a single interface to be used for entities of different types.

```scala
trait Shape {
  def area: Double
}

class Circle(radius: Double) extends Shape {
  def area: Double = math.Pi * radius * radius
}

class Rectangle(width: Double, height: Double) extends Shape {
  def area: Double = width * height
}

def printArea(shape: Shape): Unit = {
  println(s"The area is ${shape.area}")
}

val circle = new Circle(5)
val rectangle = new Rectangle(4, 6)

printArea(circle) // Outputs: The area is 78.53981633974483
printArea(rectangle) // Outputs: The area is 24.0
```

## Method Overloading

Method overloading allows multiple methods with the same name but different parameter lists.

```scala
class Calculator {
  def add(x: Int, y: Int): Int = x + y
  def add(x: Double, y: Double): Double = x + y
  def add(x: Int, y: Int, z: Int): Int = x + y + z
}

val calc = new Calculator()
println(calc.add(1, 2)) // Outputs: 3
println(calc.add(1.5, 2.7)) // Outputs: 4.2
println(calc.add(1, 2, 3)) // Outputs: 6
```

## Practice Exercises

1. Create a `BankAccount` class with methods to deposit, withdraw, and check balance. Include a companion object with an `apply` method to create accounts with an initial balance.

2. Define a trait `Drawable` with a `draw` method, and create several classes (e.g., `Circle`, `Rectangle`, `Triangle`) that extend this trait.

3. Implement a simple class hierarchy for a library system, including a base `LibraryItem` class and derived classes like `Book` and `DVD`. Use method overriding to customize behavior.

4. Create a `Person` class with overloaded constructors to handle different ways of creating a person (e.g., with or without a middle name, with or without an age).

## Advanced Example: Combining OOP Concepts

Let's create a more complex example that combines various OOP concepts:

```scala
// Trait for items that can be sold
trait Sellable {
  def price: Double
  def discountedPrice(discountPercent: Double): Double = price * (1 - discountPercent / 100)
}

// Abstract base class for all products
abstract class Product(val name: String, val basePrice: Double) extends Sellable {
  def description: String
}

// Concrete product classes
class Book(name: String, author: String, basePrice: Double) extends Product(name, basePrice) {
  override def price: Double = basePrice
  override def description: String = s"$name by $author"
}

class Electronics(name: String, brand: String, basePrice: Double, markupPercent: Double) extends Product(name, basePrice) {
  override def price: Double = basePrice * (1 + markupPercent / 100)
  override def description: String = s"$brand $name"
}

// Companion object for creating products
object Product {
  def apply(productType: String, name: String, price: Double, extra: String, markup: Double = 0): Product = {
    productType match {
      case "book" => new Book(name, extra, price)
      case "electronics" => new Electronics(name, extra, price, markup)
      case _ => throw new IllegalArgumentException("Unknown product type")
    }
  }
}

// Shopping cart to hold and manage products
class ShoppingCart {
  private var items: List[Product] = List.empty

  def addItem(item: Product): Unit = {
    items = item :: items
  }

  def totalPrice: Double = items.map(_.price).sum

  def applyDiscount(discountPercent: Double): Double = {
    items.map(_.discountedPrice(discountPercent)).sum
  }

  def showItems(): Unit = {
    items.foreach(item => println(s"${item.description} - $${item.price}"))
  }
}

// Usage
val cart = new ShoppingCart()
cart.addItem(Product("book", "Scala for Beginners", 39.99, "John Doe"))
cart.addItem(Product("electronics", "Laptop", 999.99, "TechBrand", 20))

println("Items in cart:")
cart.showItems()
println(s"Total price: $${cart.totalPrice}")
println(s"Price with 10% discount: $${cart.applyDiscount(10)}")
```

This example demonstrates:
- Traits (`Sellable`)
- Abstract classes (`Product`)
- Concrete classes (`Book`, `Electronics`)
- Inheritance and method overriding
- Companion objects with `apply` methods
- Polymorphism in the `ShoppingCart` class

## Conclusion

Object-oriented programming in Scala provides powerful tools for structuring and organizing code. By combining classes, objects, traits, and inheritance, you can create flexible and reusable code structures. Scala's approach to OOP, which includes features like companion objects and traits, offers unique advantages over traditional OOP languages.

As you continue to work with Scala, you'll find that effectively combining object-oriented and functional programming paradigms can lead to elegant and efficient solutions to complex problems. In the next lesson, we'll explore Scala's type system, which adds another layer of power and flexibility to your Scala programs.