# Scala and Functional Reactive Programming

Functional Reactive Programming (FRP) is a programming paradigm that combines functional programming with reactive programming. It allows developers to work with asynchronous data streams and events in a declarative way. In Scala, libraries such as Akka and Monix provide powerful tools for building reactive systems. This lesson covers key concepts like Observables, Subjects, and event streams, as well as how to build reactive systems using Akka and Monix.

## Observables, Subjects, and Event Streams

### Observables

An Observable is a core concept in reactive programming that represents a stream of data that can be observed. Observables can emit multiple values over time, allowing subscribers to react to changes.

#### Creating an Observable

You can create an Observable using Monix or Akka Streams. Here’s an example using Monix:

```scala
import monix.reactive.Observable

// Create an Observable that emits numbers
val numbers: Observable[Int] = Observable.range(1, 5)

// Subscribe to the Observable
numbers.subscribe(num => println(s"Received: $num"))
```

### Subjects

A Subject is a special type of Observable that can both emit and subscribe to values. It acts as both an observer and an observable.

#### Creating a Subject

You can create a Subject in Monix as follows:

```scala
import monix.reactive.Subject

// Create a Subject
val subject: Subject[Int] = Subject[Int]()

// Subscribe to the Subject
subject.subscribe(num => println(s"Received from subject: $num"))

// Emit values to the Subject
subject.onNext(1)
subject.onNext(2)
```

### Event Streams

Event streams are sequences of events that can be observed and reacted to. They are often used in applications to handle user interactions or external data sources.

#### Example of an Event Stream

```scala
import monix.reactive.Observable
import scala.concurrent.duration._

val eventStream: Observable[String] = Observable.interval(1.second).map(_ => "New Event")

// Subscribe to the event stream
eventStream.subscribe(event => println(event))
```

## Building Reactive Systems with Akka

Akka is a toolkit for building concurrent and distributed applications using the actor model. It provides a powerful framework for building reactive systems.

### Key Concepts of Akka

1. **Actors**: Actors are lightweight, isolated entities that communicate through messages.

2. **Supervision**: Akka provides a supervision strategy to handle failures in a structured way.

3. **Streams**: Akka Streams allows you to process data streams in a non-blocking way.

### Creating an Akka Actor

Here’s an example of creating a simple Akka actor:

```scala
import akka.actor.{Actor, ActorSystem, Props}

class HelloActor extends Actor {
  def receive: Receive = {
    case "hello" => println("Hello, World!")
    case _ => println("Unknown message")
  }
}

object AkkaExample extends App {
  val system: ActorSystem = ActorSystem("HelloSystem")
  val helloActor = system.actorOf(Props[HelloActor], "helloActor")

  // Send a message to the actor
  helloActor ! "hello"
  helloActor ! "goodbye"

  // Shutdown the actor system
  system.terminate()
}
```

### Using Akka Streams

Akka Streams provides a way to build data processing pipelines using the actor model.

#### Example of Akka Streams

```scala
import akka.actor.ActorSystem
import akka.stream.scaladsl.{Sink, Source}
import akka.stream.ActorMaterializer

object AkkaStreamsExample extends App {
  implicit val system: ActorSystem = ActorSystem("MyActorSystem")
  implicit val materializer: ActorMaterializer = ActorMaterializer()

  // Create a source of numbers
  val source = Source(1 to 10)

  // Create a flow that squares each number
  val flow = source.map(x => x * x)

  // Create a sink that prints each number
  val sink = Sink.foreach[Int](println)

  // Run the stream
  flow.runWith(sink) // Outputs: 1, 4, 9, 16, 25, 36, 49, 64, 81, 100
}
```

## Building Reactive Systems with Monix

Monix is a library for asynchronous programming and reactive streams in Scala. It provides a powerful way to work with observables and manage concurrency.

### Key Features of Monix

1. **Observables**: Monix provides `Observable`, which allows you to work with streams of data in a functional way.

2. **Task**: Monix's `Task` represents a computation that can be executed asynchronously.

3. **Composability**: Monix allows you to compose operations on observables and tasks easily.

### Example of Using Monix

```scala
import monix.eval.Task
import monix.reactive.Observable
import monix.execution.Scheduler.Implicits.global

object MonixExample extends App {
  // Create an observable of numbers
  val observable = Observable.range(1, 11)

  // Process the observable
  val task: Task[Unit] = observable
    .map(x => x * x) // Square each number
    .foreach(x => println(x)) // Print each squared number

  // Run the task
  task.runSyncUnsafe() // Outputs: 1, 4, 9, 16, 25, 36, 49, 64, 81, 100
}
```

### Combining Akka and Monix

You can combine Akka and Monix to build reactive systems that leverage both libraries' strengths.

```scala
import akka.actor.ActorSystem
import akka.stream.ActorMaterializer
import akka.stream.scaladsl.{Sink, Source}
import monix.reactive.Observable
import monix.execution.Scheduler.Implicits.global

object CombinedExample extends App {
  implicit val system: ActorSystem = ActorSystem("MyActorSystem")
  implicit val materializer: ActorMaterializer = ActorMaterializer()

  // Create an Akka Stream source
  val akkaSource = Source(1 to 10)

  // Convert Akka Stream to Monix Observable
  val observable: Observable[Int] = Observable.from(akkaSource.runWith(Sink.seq).map(_.toList))

  // Process the observable
  observable
    .map(x => x * x) // Square each number
    .foreach(x => println(x)) // Print each squared number
}
```

## Practice Exercises

1. Create an Akka HTTP service that processes incoming requests and streams a response back to the client.

2. Write a Monix application that fetches data from a public API and processes the response asynchronously.

3. Implement a simple event-driven system using Monix Observables to handle user interactions.

4. Create a reactive stream using Akka Streams that reads data from a file and processes it line by line.

## Advanced Example: Building a Reactive Application

Let’s create a more complex example that combines Akka, Monix, and reactive programming concepts to build a simple chat application.

```scala
import akka.actor.{Actor, ActorRef, ActorSystem, Props}
import akka.stream.scaladsl.{Sink, Source}
import akka.stream.ActorMaterializer
import monix.reactive.Observable
import scala.concurrent.duration._

case class Message(sender: String, content: String)

class ChatRoom extends Actor {
  private var clients: List[ActorRef] = List()

  def receive: Receive = {
    case msg: Message =>
      clients.foreach(_ ! msg) // Broadcast message to all clients
    case client: ActorRef =>
      clients = client :: clients // Add new client
  }
}

object ChatApp extends App {
  implicit val system: ActorSystem = ActorSystem("ChatSystem")
  implicit val materializer: ActorMaterializer = ActorMaterializer()

  val chatRoom = system.actorOf(Props[ChatRoom], "chatRoom")

  // Simulate clients sending messages
  val client1 = system.actorOf(Props(new Actor {
    def receive: Receive = {
      case msg: Message => println(s"Client 1 received: ${msg.sender}: ${msg.content}")
    }
  }))

  val client2 = system.actorOf(Props(new Actor {
    def receive: Receive = {
      case msg: Message => println(s"Client 2 received: ${msg.sender}: ${msg.content}")
    }
  }))

  chatRoom ! client1
  chatRoom ! client2

  // Create a source of messages
  val messageSource = Source.tick(1.second, 1.second, "tick").zipWithIndex.map {
    case (_, idx) => Message(s"Client ${idx % 2 + 1}", s"Hello from client ${idx % 2 + 1}!")
  }

  // Stream messages to the chat room
  messageSource.runForeach(msg => chatRoom ! msg)

  // Keep the application running
  Thread.sleep(10000)
  system.terminate()
}
```

In this example:
- We create a `ChatRoom` actor that manages clients and broadcasts messages.
- We simulate two clients that receive messages from the chat room.
- We use an Akka Stream to generate messages periodically and send them to the chat room.

## Conclusion

Scala provides robust tools for building reactive applications through Akka and Monix. By leveraging Observables, Subjects, and event streams, you can create responsive and efficient systems that handle asynchronous data flows.

Understanding and applying these advanced concurrency patterns will enhance your ability to build scalable, maintainable, and high-performance applications. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.