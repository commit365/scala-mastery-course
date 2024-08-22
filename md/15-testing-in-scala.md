# Testing in Scala

Testing is a crucial part of software development that ensures your code behaves as expected. Scala provides several powerful libraries for writing tests, including ScalaTest, specs2, and ScalaCheck. This lesson covers unit testing, property-based testing, and integration testing in Scala.

## Unit Testing with ScalaTest

ScalaTest is a popular testing framework in Scala that supports various styles of testing, including flat spec, fun spec, and word spec.

### Setting Up ScalaTest

Add the following dependency to your `build.sbt` file:

```sbt
libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.10" % Test
```

### Writing a Unit Test

Here’s an example of a simple unit test using ScalaTest:

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

class Calculator {
  def add(a: Int, b: Int): Int = a + b
  def subtract(a: Int, b: Int): Int = a - b
}

class CalculatorSpec extends AnyFlatSpec with Matchers {

  "A Calculator" should "correctly add two numbers" in {
    val calculator = new Calculator
    calculator.add(3, 4) shouldEqual 7
  }

  it should "correctly subtract two numbers" in {
    val calculator = new Calculator
    calculator.subtract(10, 5) shouldEqual 5
  }
}
```

### Running the Tests

You can run the tests using your IDE or by using the following command in the terminal:

```bash
sbt test
```

## Unit Testing with specs2

specs2 is another powerful testing framework for Scala that emphasizes behavior-driven development (BDD).

### Setting Up specs2

Add the following dependency to your `build.sbt` file:

```sbt
libraryDependencies += "org.specs2" %% "specs2-core" % "4.12.3" % Test
```

### Writing a Unit Test

Here’s an example of a simple unit test using specs2:

```scala
import org.specs2.mutable.Specification

class Calculator {
  def add(a: Int, b: Int): Int = a + b
  def subtract(a: Int, b: Int): Int = a - b
}

class CalculatorSpec extends Specification {

  "A Calculator" should {
    "correctly add two numbers" in {
      val calculator = new Calculator
      calculator.add(3, 4) mustEqual 7
    }

    "correctly subtract two numbers" in {
      val calculator = new Calculator
      calculator.subtract(10, 5) mustEqual 5
    }
  }
}
```

### Running the Tests

You can run the tests using your IDE or by using the same command as before:

```bash
sbt test
```

## Property-Based Testing with ScalaCheck

Property-based testing is a technique where you define properties that should hold for a range of inputs, and the testing framework generates test cases to verify those properties.

### Setting Up ScalaCheck

Add the following dependency to your `build.sbt` file:

```sbt
libraryDependencies += "org.scalacheck" %% "scalacheck" % "1.15.4" % Test
```

### Writing Property-Based Tests

Here’s an example of property-based testing using ScalaCheck:

```scala
import org.scalacheck.Properties
import org.scalacheck.Prop.forAll

object CalculatorProperties extends Properties("Calculator") {

  property("addition is commutative") = forAll { (a: Int, b: Int) =>
    val calculator = new Calculator
    calculator.add(a, b) == calculator.add(b, a)
  }

  property("addition is associative") = forAll { (a: Int, b: Int, c: Int) =>
    val calculator = new Calculator
    calculator.add(a, calculator.add(b, c)) == calculator.add(calculator.add(a, b), c)
  }
}
```

### Running Property-Based Tests

You can run property-based tests using the same command:

```bash
sbt test
```

## Integration Testing

Integration testing verifies that different components of your application work together as expected. You can use ScalaTest or specs2 for integration testing, often in conjunction with a test framework like `Mockito` for mocking dependencies.

### Setting Up Mockito

Add the following dependencies to your `build.sbt` file:

```sbt
libraryDependencies += "org.mockito" %% "mockito-scala" % "1.16.46" % Test
```

### Writing an Integration Test

Here’s an example of an integration test using ScalaTest and Mockito:

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers
import org.mockito.Mockito._
import org.mockito.ArgumentMatchers._

class UserService(userRepository: UserRepository) {
  def getUserName(id: Int): String = {
    userRepository.findById(id) match {
      case Some(user) => user.name
      case None => "Unknown"
    }
  }
}

trait UserRepository {
  def findById(id: Int): Option[User]
}

case class User(id: Int, name: String)

class UserServiceSpec extends AnyFlatSpec with Matchers {

  "UserService" should "return user name for existing user" in {
    val mockRepo = mock[UserRepository]
    when(mockRepo.findById(1)).thenReturn(Some(User(1, "Alice")))

    val userService = new UserService(mockRepo)
    userService.getUserName(1) shouldEqual "Alice"
  }

  it should "return 'Unknown' for non-existing user" in {
    val mockRepo = mock[UserRepository]
    when(mockRepo.findById(2)).thenReturn(None)

    val userService = new UserService(mockRepo)
    userService.getUserName(2) shouldEqual "Unknown"
  }
}
```

### Running Integration Tests

You can run integration tests using the same command:

```bash
sbt test
```

## Practice Exercises

1. Write unit tests for a simple `Stack` class using ScalaTest and specs2.

2. Implement property-based tests for a function that checks if a string is a palindrome.

3. Create an integration test for a service that interacts with a database, using a mocking framework to simulate the database behavior.

4. Write a property-based test that verifies the behavior of a sorting algorithm.

## Advanced Example: Combining Testing Techniques

Let’s create a more complex example that combines unit testing, property-based testing, and integration testing:

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers
import org.scalacheck.Properties
import org.scalacheck.Prop.forAll
import org.mockito.Mockito._
import org.mockito.ArgumentMatchers._

// Define a simple service
class UserService(userRepository: UserRepository) {
  def getUserName(id: Int): String = {
    userRepository.findById(id) match {
      case Some(user) => user.name
      case None => "Unknown"
    }
  }
}

// Define the UserRepository trait
trait UserRepository {
  def findById(id: Int): Option[User]
}

// Define the User case class
case class User(id: Int, name: String)

// Unit tests for UserService
class UserServiceSpec extends AnyFlatSpec with Matchers {

  "UserService" should "return user name for existing user" in {
    val mockRepo = mock[UserRepository]
    when(mockRepo.findById(1)).thenReturn(Some(User(1, "Alice")))

    val userService = new UserService(mockRepo)
    userService.getUserName(1) shouldEqual "Alice"
  }

  it should "return 'Unknown' for non-existing user" in {
    val mockRepo = mock[UserRepository]
    when(mockRepo.findById(2)).thenReturn(None)

    val userService = new UserService(mockRepo)
    userService.getUserName(2) shouldEqual "Unknown"
  }
}

// Property-based tests for UserService
object UserServiceProperties extends Properties("UserService") {

  property("user name is consistent") = forAll { (id: Int, name: String) =>
    val user = User(id, name)
    val mockRepo = mock[UserRepository]
    when(mockRepo.findById(id)).thenReturn(Some(user))

    val userService = new UserService(mockRepo)
    userService.getUserName(id) == name
  }
}

// Integration test for UserService
class UserServiceIntegrationSpec extends AnyFlatSpec with Matchers {

  "UserService" should "interact correctly with a real UserRepository" in {
    // Assume we have a real UserRepository implementation
    val realRepo = new RealUserRepository() // Hypothetical implementation
    val userService = new UserService(realRepo)

    // Test with known data
    userService.getUserName(1) shouldEqual "Alice" // Assuming Alice is in the repository
  }
}

// Note: RealUserRepository needs to be implemented for the integration test to work

```

This example demonstrates:
- Unit testing with mocks using ScalaTest.
- Property-based testing using ScalaCheck.
- Integration testing with a hypothetical real repository.

## Conclusion

Testing is a vital part of software development that ensures your code behaves as expected. Scala provides powerful libraries like ScalaTest, specs2, and ScalaCheck for unit testing, property-based testing, and integration testing. By mastering these testing techniques, you can write robust and maintainable code that meets your application's requirements.

In the next lesson, we will explore functional programming concepts in more depth, building on the foundation laid by these testing techniques to create more expressive and efficient code.