# Advanced Functional Programming in Scala

This lesson covers advanced functional programming concepts and their practical applications in Scala.

## Functors

A functor is a type class that represents the ability to map over a structure. In Scala, any type that defines a `map` method can be considered a functor.

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Example: List as a Functor
implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

// Usage
val numbers = List(1, 2, 3, 4, 5)
val doubled = listFunctor.map(numbers)(_ * 2)
println(doubled) // List(2, 4, 6, 8, 10)
```

## Applicatives

An applicative is a more powerful version of a functor that allows you to apply functions wrapped in the context to values wrapped in the context.

```scala
trait Applicative[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
}

// Example: Option as an Applicative
implicit val optionApplicative: Applicative[Option] = new Applicative[Option] {
  def pure[A](a: A): Option[A] = Some(a)
  def ap[A, B](ff: Option[A => B])(fa: Option[A]): Option[B] = 
    ff.flatMap(f => fa.map(f))
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa.map(f)
}

// Usage
val maybeNumber: Option[Int] = Some(5)
val maybeDouble: Option[Int => Int] = Some(x => x * 2)
val result = optionApplicative.ap(maybeDouble)(maybeNumber)
println(result) // Some(10)
```

## Monads

A monad is a type class that allows you to chain operations while handling effects (like optionality or failure).

```scala
trait Monad[F[_]] extends Applicative[F] {
  def flatMap[A, B](fa: F[A])(f: A => F[B]): F[B]
}

// Example: List as a Monad
implicit val listMonad: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a)
  def ap[A, B](ff: List[A => B])(fa: List[A]): List[B] = 
    for {
      f <- ff
      a <- fa
    } yield f(a)
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
  def flatMap[A, B](fa: List[A])(f: A => List[B]): List[B] = fa.flatMap(f)
}

// Usage
val numbers = List(1, 2, 3)
val result = listMonad.flatMap(numbers)(x => List(x, x * 2))
println(result) // List(1, 2, 2, 4, 3, 6)
```

## Functional Error Handling

Scala provides several types for functional error handling: `Option`, `Either`, and `Try`.

### Option

`Option` represents a value that may or may not be present.

```scala
def divide(a: Int, b: Int): Option[Int] = 
  if (b != 0) Some(a / b) else None

val result1 = divide(10, 2)
val result2 = divide(10, 0)

println(result1) // Some(5)
println(result2) // None

// Using map and flatMap
val computation = for {
  x <- divide(10, 2)
  y <- divide(x, 2)
} yield y

println(computation) // Some(2)
```

### Either

`Either` represents a value of one of two possible types (usually a success value or an error value).

```scala
def divide(a: Int, b: Int): Either[String, Int] = 
  if (b != 0) Right(a / b) else Left("Division by zero")

val result1 = divide(10, 2)
val result2 = divide(10, 0)

println(result1) // Right(5)
println(result2) // Left(Division by zero)

// Using map and flatMap
val computation = for {
  x <- divide(10, 2)
  y <- divide(x, 2)
} yield y

println(computation) // Right(2)
```

### Try

`Try` represents a computation that may result in a value or an exception.

```scala
import scala.util.{Try, Success, Failure}

def divide(a: Int, b: Int): Try[Int] = Try(a / b)

val result1 = divide(10, 2)
val result2 = divide(10, 0)

println(result1) // Success(5)
println(result2) // Failure(java.lang.ArithmeticException: / by zero)

// Using map and flatMap
val computation = for {
  x <- divide(10, 2)
  y <- divide(x, 2)
} yield y

println(computation) // Success(2)
```

## Practice Exercises

1. Implement a `Functor` instance for `Option`.

2. Create an `Applicative` instance for `Either[E, _]`.

3. Write a function that uses `Option` to find the first element in a `List` that satisfies a predicate.

4. Implement a safe division function that returns an `Either[String, Double]`, handling both division by zero and invalid input.

## Advanced Example: Combining Concepts

Let's create a more complex example that combines these advanced functional programming concepts:

```scala
import scala.util.{Try, Success, Failure}

// Define a simple User class
case class User(id: Int, name: String, email: String)

// Define a UserRepository trait with operations that might fail
trait UserRepository {
  def findById(id: Int): Try[User]
  def updateEmail(user: User, newEmail: String): Try[User]
}

// Implement a simple in-memory UserRepository
class InMemoryUserRepository extends UserRepository {
  private var users = Map(
    1 -> User(1, "Alice", "alice@example.com"),
    2 -> User(2, "Bob", "bob@example.com")
  )

  def findById(id: Int): Try[User] = Try(users(id))

  def updateEmail(user: User, newEmail: String): Try[User] = Try {
    val updatedUser = user.copy(email = newEmail)
    users = users + (user.id -> updatedUser)
    updatedUser
  }
}

// Define a UserService that uses the UserRepository
class UserService(repo: UserRepository) {
  def changeUserEmail(userId: Int, newEmail: String): Either[String, User] = {
    (for {
      user <- repo.findById(userId)
      updatedUser <- repo.updateEmail(user, newEmail)
    } yield updatedUser).toEither.left.map(_.getMessage)
  }
}

// Usage
val repo = new InMemoryUserRepository()
val service = new UserService(repo)

val result1 = service.changeUserEmail(1, "alice.new@example.com")
val result2 = service.changeUserEmail(3, "nonexistent@example.com")

println(result1) // Right(User(1,Alice,alice.new@example.com))
println(result2) // Left(java.util.NoSuchElementException: key not found: 3)

// Functor, Applicative, and Monad for Either
implicit def eitherFunctor[E]: Functor[Either[E, *]] = new Functor[Either[E, *]] {
  def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
}

implicit def eitherApplicative[E]: Applicative[Either[E, *]] = new Applicative[Either[E, *]] {
  def pure[A](a: A): Either[E, A] = Right(a)
  def ap[A, B](ff: Either[E, A => B])(fa: Either[E, A]): Either[E, B] = 
    (ff, fa) match {
      case (Right(f), Right(a)) => Right(f(a))
      case (Left(e), _) => Left(e)
      case (_, Left(e)) => Left(e)
    }
  def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
}

implicit def eitherMonad[E]: Monad[Either[E, *]] = new Monad[Either[E, *]] {
  def pure[A](a: A): Either[E, A] = Right(a)
  def ap[A, B](ff: Either[E, A => B])(fa: Either[E, A]): Either[E, B] = 
    eitherApplicative.ap(ff)(fa)
  def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
  def flatMap[A, B](fa: Either[E, A])(f: A => Either[E, B]): Either[E, B] = fa.flatMap(f)
}

// Using the Monad instance
val computationResult = for {
  user1 <- service.changeUserEmail(1, "alice.new2@example.com")
  user2 <- service.changeUserEmail(2, "bob.new@example.com")
} yield (user1, user2)

println(computationResult) // Right((User(1,Alice,alice.new2@example.com),User(2,Bob,bob.new@example.com)))
```

This example demonstrates:
- Use of `Try` for operations that might fail
- Conversion from `Try` to `Either` for better error handling
- Implementation of a simple repository and service layer
- Use of for-comprehensions with `Try` and `Either`
- Implementation of `Functor`, `Applicative`, and `Monad` instances for `Either`

## Conclusion

Advanced functional programming concepts like functors, applicatives, and monads provide powerful abstractions for working with effects and composing computations. Scala's `Option`, `Either`, and `Try` types offer elegant solutions for handling errors and uncertain values in a functional way.

By mastering these concepts and techniques, you can write more expressive, composable, and robust code. These patterns are widely used in Scala libraries and are fundamental to functional programming in Scala.

In the next lesson, we'll explore implicits and type classes, which build upon these functional programming concepts to provide even more powerful abstractions and capabilities in Scala.