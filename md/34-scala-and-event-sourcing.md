# Scala and Event Sourcing

Event sourcing is a powerful architectural pattern used in software development to model state changes as a sequence of events. Instead of storing the current state of an application, event sourcing stores all changes to the application state as a series of events. This lesson covers how to implement event sourcing patterns in Scala and how to use CQRS (Command Query Responsibility Segregation) with Akka Persistence.

## Implementing Event Sourcing Patterns in Scala

### What is Event Sourcing?

In event sourcing, every change to the application state is captured as an event. This allows you to reconstruct the current state by replaying the events. Event sourcing provides benefits such as:

- **Auditability**: You can track all changes made to the state.
- **Temporal Queries**: You can query the state at any point in time by replaying events.
- **Decoupling**: It decouples the write and read models, which can lead to better scalability.

### Basic Structure of Event Sourcing

1. **Event**: A class representing a state change.
2. **Aggregate**: A class that encapsulates the state and behavior of a specific entity.
3. **Event Store**: A storage mechanism for persisting events.

### Example of Event Sourcing

```scala
// Define events
sealed trait Event
case class UserRegistered(username: String) extends Event
case class UserUpdated(username: String, newName: String) extends Event

// Define the aggregate
class UserAggregate {
  private var state: Option[String] = None

  def applyEvent(event: Event): Unit = event match {
    case UserRegistered(username) => state = Some(username)
    case UserUpdated(_, newName) => state = state.map(_ => newName)
  }

  def getState: Option[String] = state
}

// Usage
val userAggregate = new UserAggregate()
userAggregate.applyEvent(UserRegistered("Alice"))
println(userAggregate.getState) // Outputs: Some(Alice)

userAggregate.applyEvent(UserUpdated("Alice", "Alice Smith"))
println(userAggregate.getState) // Outputs: Some(Alice Smith)
```

## CQRS with Akka Persistence

CQRS is an architectural pattern that separates the read and write sides of an application. In a CQRS architecture, commands modify the state, while queries retrieve the state. Akka Persistence can be used to implement CQRS with event sourcing.

### Setting Up Akka Persistence

1. **Add Dependencies**: Include the necessary dependencies for Akka Persistence in your `build.sbt`:

```sbt
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-persistence" % "2.6.16",
  "com.typesafe.akka" %% "akka-persistence-jdbc" % "5.0.0"
)
```

2. **Configure Akka Persistence**: Update your `application.conf`:

```hocon
akka {
  persistence {
    journal.plugin = "akka.persistence.jdbc.journal"
    snapshot.plugin = "akka.persistence.jdbc.snapshot"
  }
}
```

### Implementing CQRS with Akka Persistence

1. **Define Commands and Events**:

```scala
sealed trait Command
case class RegisterUser(username: String) extends Command
case class UpdateUser(username: String, newName: String) extends Command

sealed trait Event
case class UserRegistered(username: String) extends Event
case class UserUpdated(username: String, newName: String) extends Event
```

2. **Create a Persistent Actor for the Aggregate**:

```scala
import akka.persistence._

class UserActor extends PersistentActor {
  override def persistenceId: String = "user-actor"

  private var state: Option[String] = None

  def updateState(event: Event): Unit = event match {
    case UserRegistered(username) => state = Some(username)
    case UserUpdated(_, newName) => state = state.map(_ => newName)
  }

  override def receiveRecover: Receive = {
    case event: Event => updateState(event)
  }

  override def receiveCommand: Receive = {
    case RegisterUser(username) =>
      persist(UserRegistered(username)) { event =>
        updateState(event)
        println(s"User registered: $username")
      }
    case UpdateUser(username, newName) =>
      persist(UserUpdated(username, newName)) { event =>
        updateState(event)
        println(s"User updated: $newName")
      }
  }
}
```

3. **Using the Actor**:

```scala
import akka.actor.ActorSystem
import akka.actor.Props

object CQRSApp extends App {
  val system = ActorSystem("CQRSSystem")
  val userActor = system.actorOf(Props[UserActor], "userActor")

  // Send commands to the actor
  userActor ! RegisterUser("Alice")
  userActor ! UpdateUser("Alice", "Alice Smith")

  // Shutdown the actor system
  system.terminate()
}
```

## Event Store and Query Side

In a CQRS architecture, the event store is used to persist events, while the query side retrieves the current state from the event store.

### Implementing the Event Store

You can implement an event store using Akka Persistence:

```scala
import akka.persistence._

trait EventStore {
  def saveEvent(event: Event): Unit
}

class InMemoryEventStore extends EventStore {
  private var events: List[Event] = List()

  def saveEvent(event: Event): Unit = {
    events = event :: events
  }

  def getEvents: List[Event] = events.reverse
}
```

### Querying the State

You can query the state by replaying events from the event store:

```scala
def getUserState(username: String, eventStore: EventStore): Option[String] = {
  val events = eventStore.getEvents
  val userAggregate = new UserAggregate()

  events.foreach {
    case UserRegistered(user) if user == username => userAggregate.applyEvent(UserRegistered(user))
    case UserUpdated(user, newName) if user == username => userAggregate.applyEvent(UserUpdated(user, newName))
    case _ =>
  }

  userAggregate.getState
}

// Usage
val eventStore = new InMemoryEventStore()
eventStore.saveEvent(UserRegistered("Alice"))
eventStore.saveEvent(UserUpdated("Alice", "Alice Smith"))

val userState = getUserState("Alice", eventStore)
println(userState) // Outputs: Some(Alice Smith)
```

## Performance Considerations

1. **Event Size**: Keep events small to reduce the overhead of storing and processing them.

2. **Snapshotting**: Use snapshots to optimize recovery time by periodically saving the state of the aggregate.

3. **Concurrency Control**: Implement appropriate concurrency control mechanisms to handle simultaneous updates.

4. **Monitoring**: Monitor the performance of your event sourcing system to identify bottlenecks and optimize accordingly.

## Practice Exercises

1. Implement a simple event sourcing system for managing a shopping cart, including commands for adding and removing items.

2. Create a CQRS application that tracks user registrations and allows querying of user data.

3. Write tests for your event sourcing implementation to ensure that events are correctly applied to the state.

4. Explore the use of snapshots in Akka Persistence to optimize the recovery of state in your application.

## Advanced Example: Building a Complete Event Sourcing System

Let’s create a more complex example that combines event sourcing and CQRS for a simple banking application:

```scala
import akka.actor.{Actor, ActorSystem, Props}
import akka.persistence._

sealed trait BankingCommand
case class Deposit(amount: Double) extends BankingCommand
case class Withdraw(amount: Double) extends BankingCommand
case class GetBalance(replyTo: ActorRef) extends BankingCommand

sealed trait BankingEvent
case class Deposited(amount: Double) extends BankingEvent
case class Withdrawn(amount: Double) extends BankingEvent

class BankAccountActor extends PersistentActor {
  override def persistenceId: String = "bank-account-actor"

  private var balance: Double = 0.0

  def updateBalance(event: BankingEvent): Unit = event match {
    case Deposited(amount) => balance += amount
    case Withdrawn(amount) => balance -= amount
  }

  override def receiveRecover: Receive = {
    case event: BankingEvent => updateBalance(event)
  }

  override def receiveCommand: Receive = {
    case Deposit(amount) =>
      persist(Deposited(amount)) { event =>
        updateBalance(event)
        println(s"Deposited: $amount, New Balance: $balance")
      }
    case Withdraw(amount) =>
      persist(Withdrawn(amount)) { event =>
        updateBalance(event)
        println(s"Withdrawn: $amount, New Balance: $balance")
      }
    case GetBalance(replyTo) =>
      replyTo ! balance
  }
}

object BankingApp extends App {
  val system = ActorSystem("BankingSystem")
  val bankAccountActor = system.actorOf(Props[BankAccountActor], "bankAccountActor")

  // Send commands to the actor
  bankAccountActor ! Deposit(100.0)
  bankAccountActor ! Withdraw(50.0)

  // Get the current balance
  bankAccountActor ! GetBalance(bankAccountActor)

  // Shutdown the actor system
  system.terminate()
}
```

In this example:
- We define a `BankAccountActor` that manages deposits and withdrawals.
- The actor persists events representing the changes to the account balance.
- We can query the current balance by sending a `GetBalance` command.

## Conclusion

Event sourcing is a powerful pattern for managing state in distributed systems, and Scala provides robust tools for implementing this pattern with Akka Persistence. By understanding how to implement event sourcing and CQRS, you can build resilient and scalable applications that maintain a clear audit trail of state changes.

Mastering these concepts will enhance your ability to design effective data management solutions in your applications. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.