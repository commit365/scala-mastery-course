# Concurrent Programming in Scala

Concurrent programming is essential for building responsive and efficient applications. Scala provides powerful abstractions for concurrency, including Futures, Promises, and the Akka actors framework. This lesson covers these concepts and their practical applications.

## Futures

A `Future` represents a computation that may not have completed yet. It allows you to write non-blocking code by executing tasks asynchronously.

### Creating a Future

You can create a Future using the `Future` companion object.

```scala
import scala.concurrent.{Future, Await}
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global

val futureResult: Future[Int] = Future {
  // Simulate a long-running computation
  Thread.sleep(1000)
  42
}

// Use Await to block and get the result (not recommended in production)
val result = Await.result(futureResult, 2.seconds)
println(result) // Outputs: 42
```

### Handling Future Results

You can handle the result of a Future using `onComplete`, `map`, and `flatMap`.

```scala
val future = Future {
  // Simulate computation
  Thread.sleep(500)
  10
}

future.onComplete {
  case Success(value) => println(s"Result: $value")
  case Failure(exception) => println(s"Failed with: $exception")
}

// Using map and flatMap
val doubledFuture: Future[Int] = future.map(_ * 2)

doubledFuture.foreach(result => println(s"Doubled Result: $result")) // Outputs: Doubled Result: 20
```

### Composing Futures

You can compose multiple Futures using for-comprehensions or combinators like `zip`.

```scala
val future1 = Future {
  Thread.sleep(300)
  10
}

val future2 = Future {
  Thread.sleep(200)
  20
}

// Using for-comprehension
val combinedFuture: Future[Int] = for {
  a <- future1
  b <- future2
} yield a + b

combinedFuture.foreach(result => println(s"Combined Result: $result")) // Outputs: Combined Result: 30
```

## Promises

A `Promise` is a writable, single-assignment container that completes a Future. You can use Promises to create Futures that can be completed later.

### Creating a Promise

```scala
import scala.concurrent.Promise

val promise = Promise[Int]()
val futureFromPromise: Future[Int] = promise.future

// Complete the promise
promise.success(100)

// Handling the future
futureFromPromise.foreach(result => println(s"Promise Result: $result")) // Outputs: Promise Result: 100
```

### Failure Handling with Promises

You can also fail a Promise.

```scala
val failingPromise = Promise[Int]()
val failingFuture: Future[Int] = failingPromise.future

// Complete with failure
failingPromise.failure(new RuntimeException("Something went wrong!"))

failingFuture.onComplete {
  case Success(value) => println(s"Result: $value")
  case Failure(exception) => println(s"Failed with: $exception") // Outputs: Failed with: java.lang.RuntimeException: Something went wrong!
}
```

## ExecutionContext

An `ExecutionContext` is a context in which a Future runs. It defines the thread pool and can be customized.

### Using the Global ExecutionContext

```scala
import scala.concurrent.ExecutionContext.Implicits.global

val future = Future {
  // This runs on the global execution context
  Thread.sleep(100)
  "Hello, World!"
}

future.foreach(println) // Outputs: Hello, World!
```

### Creating a Custom ExecutionContext

You can create a custom ExecutionContext using a thread pool.

```scala
import java.util.concurrent.Executors
import scala.concurrent.ExecutionContext

val customThreadPool = Executors.newFixedThreadPool(4)
implicit val customExecutionContext: ExecutionContext = ExecutionContext.fromExecutor(customThreadPool)

val future = Future {
  // This runs on the custom execution context
  Thread.sleep(100)
  "Hello from custom context!"
}

future.foreach(println) // Outputs: Hello from custom context!

// Shutdown the custom thread pool
customThreadPool.shutdown()
```

## Akka Actors

Akka is a powerful toolkit for building concurrent and distributed applications. It uses the actor model, where actors are lightweight, isolated entities that communicate through messages.

### Creating an Actor

To create an actor, you need to define a class that extends `Actor` and implement the `receive` method.

```scala
import akka.actor.{Actor, ActorSystem, Props}

// Define an actor
class HelloActor extends Actor {
  def receive: Receive = {
    case "hello" => println("Hello, World!")
    case _ => println("Unknown message")
  }
}

// Create an actor system
val system = ActorSystem("HelloSystem")
val helloActor = system.actorOf(Props[HelloActor], "helloActor")

// Send a message to the actor
helloActor ! "hello" // Outputs: Hello, World!
helloActor ! "goodbye" // Outputs: Unknown message

// Shutdown the actor system
system.terminate()
```

### Actor Communication

Actors communicate by sending messages. Messages can be any serializable object.

```scala
case class Greet(name: String)

class GreetingActor extends Actor {
  def receive: Receive = {
    case Greet(name) => println(s"Hello, $name!")
  }
}

val greetingActor = system.actorOf(Props[GreetingActor], "greetingActor")
greetingActor ! Greet("Alice") // Outputs: Hello, Alice!
```

### Actor Lifecycle

Actors have a lifecycle managed by the Akka framework. You can override methods like `preStart` and `postStop` to handle initialization and cleanup.

```scala
class LifecycleActor extends Actor {
  override def preStart(): Unit = println("Actor is starting...")
  override def postStop(): Unit = println("Actor is stopping...")

  def receive: Receive = {
    case msg => println(s"Received: $msg")
  }
}

val lifecycleActor = system.actorOf(Props[LifecycleActor], "lifecycleActor")
lifecycleActor ! "Test Message" // Outputs: Actor is starting... Received: Test Message
system.stop(lifecycleActor) // Outputs: Actor is stopping...
```

## Practice Exercises

1. Create a Future that simulates a long-running computation and returns a result after a delay.

2. Implement a Promise that completes after a specified duration, and handle both success and failure cases.

3. Create an actor that counts the number of messages it receives and prints the count when it receives a "print" message.

4. Build a simple Akka application that sends messages between multiple actors.

## Advanced Example: Combining Futures and Actors

Let's create a more complex example that combines Futures and Akka actors to demonstrate their interaction:

```scala
import akka.actor.{Actor, ActorSystem, Props}
import scala.concurrent.{Future, Promise}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

// Define a message for the actor
case class ComputeSquare(number: Int)

// Define an actor that computes squares
class SquareActor extends Actor {
  def receive: Receive = {
    case ComputeSquare(number) =>
      // Simulate a long-running computation
      Thread.sleep(1000)
      sender() ! (number * number) // Send the result back to the sender
  }
}

// Create an actor system
val system = ActorSystem("SquareSystem")
val squareActor = system.actorOf(Props[SquareActor], "squareActor")

// Create a Future to compute the square
def computeSquareAsync(number: Int): Future[Int] = {
  val promise = Promise[Int]()
  squareActor.tell(ComputeSquare(number), ActorRef.noSender) // Send the message to the actor
  promise.future
}

// Usage
val futureResult = computeSquareAsync(5)

futureResult.onComplete {
  case Success(result) => println(s"The square is: $result") // Outputs: The square is: 25
  case Failure(exception) => println(s"Failed with: $exception")
}

// Shutdown the actor system after a delay to allow the Future to complete
Thread.sleep(2000)
system.terminate()
```

This example demonstrates:
- Interaction between Futures and Akka actors
- Sending messages to actors and handling responses
- Using Promises to create a Future that is completed by an actor

## Conclusion

Concurrent programming in Scala is facilitated by powerful abstractions like Futures, Promises, and the Akka actors framework. Futures and Promises provide a way to write non-blocking code, while Akka actors enable building scalable and resilient applications using the actor model.

By mastering these concepts, you can write responsive applications that efficiently utilize system resources and handle concurrent tasks. In the next lesson, we'll explore testing in Scala, focusing on techniques and tools for writing effective tests for your applications.