# Advanced Akka Features

Akka is a powerful toolkit for building concurrent, distributed, and resilient applications on the JVM. This lesson covers advanced features of Akka, including Akka Clustering, Akka Persistence, distributed data, and eventual consistency.

## Akka Clustering

Akka Clustering allows you to build a distributed system by grouping multiple Akka actors into a cluster. This enables your application to scale horizontally and handle failures gracefully.

### Setting Up Akka Clustering

1. **Add the Akka Clustering Dependency**: Add the following dependencies to your `build.sbt` file:

```sbt
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-cluster" % "2.6.16",
  "com.typesafe.akka" %% "akka-cluster-tools" % "2.6.16"
)
```

2. **Configure Akka Clustering**: Update your `application.conf` to set up clustering:

```hocon
akka {
  actor {
    provider = "cluster"
  }
  remote {
    artery {
      canonical.hostname = "127.0.0.1"
      canonical.port = 2551
    }
  }
  cluster {
    seed-nodes = [
      "akka://MyCluster@127.0.0.1:2551",
      "akka://MyCluster@127.0.0.1:2552"
    ]
  }
}
```

3. **Creating a Cluster Node**:

```scala
import akka.actor.{Actor, ActorSystem, Props}
import akka.cluster.Cluster
import akka.cluster.ClusterEvent._

class ClusterListener extends Actor {
  val cluster = Cluster(context.system)

  override def preStart(): Unit = {
    cluster.subscribe(self, classOf[MemberUp])
  }

  override def postStop(): Unit = {
    cluster.unsubscribe(self)
  }

  def receive: Receive = {
    case MemberUp(member) =>
      println(s"Member is Up: ${member.address}")
  }
}

object ClusterApp extends App {
  val system = ActorSystem("MyCluster")
  system.actorOf(Props[ClusterListener], name = "clusterListener")
}
```

### Running the Cluster

To run the cluster, start multiple instances of your application on different ports:

```bash
sbt "run -Dakka.remote.artery.canonical.port=2551"
sbt "run -Dakka.remote.artery.canonical.port=2552"
```

You should see the cluster nodes joining and the listener printing the member addresses.

## Akka Persistence

Akka Persistence allows you to persist the state of actors so that they can recover from failures. This is particularly useful for building stateful applications.

### Setting Up Akka Persistence

1. **Add the Akka Persistence Dependency**: Include the following dependencies in your `build.sbt`:

```sbt
libraryDependencies += "com.typesafe.akka" %% "akka-persistence" % "2.6.16"
libraryDependencies += "com.typesafe.akka" %% "akka-persistence-jdbc" % "5.0.0"
```

2. **Configure Akka Persistence**: Update your `application.conf` to configure persistence:

```hocon
akka {
  persistence {
    journal.plugin = "akka.persistence.jdbc.journal"
    snapshot.plugin = "akka.persistence.jdbc.snapshot"
  }
}
```

3. **Implementing a Persistent Actor**:

```scala
import akka.actor.{Actor, Props}
import akka.persistence._

case class UpdateData(data: String)

class PersistentDataActor extends PersistentActor {
  override def persistenceId: String = "persistent-data-actor"

  private var state: String = ""

  def updateState(data: String): Unit = {
    state = data
  }

  override def receiveRecover: Receive = {
    case UpdateData(data) => updateState(data)
  }

  override def receiveCommand: Receive = {
    case UpdateData(data) =>
      persist(UpdateData(data)) { event =>
        updateState(event.data)
        println(s"Updated state: $state")
      }
  }
}

object PersistenceApp extends App {
  val system = ActorSystem("PersistenceSystem")
  val persistentActor = system.actorOf(Props[PersistentDataActor], "persistentDataActor")

  persistentActor ! UpdateData("First update")
  persistentActor ! UpdateData("Second update")
}
```

### Recovering State

When the `PersistentDataActor` is restarted, it will recover its state from the journal:

```scala
val persistentActor = system.actorOf(Props[PersistentDataActor], "persistentDataActor")
// The actor will recover its state from the journal
```

## Distributed Data and Eventual Consistency

Akka provides a distributed data module that allows you to manage shared state across multiple nodes in a cluster. This module uses CRDTs (Conflict-Free Replicated Data Types) to achieve eventual consistency.

### Using Distributed Data

1. **Add the Akka Distributed Data Dependency**:

```sbt
libraryDependencies += "com.typesafe.akka" %% "akka-distributed-data" % "2.6.16"
```

2. **Configuring Distributed Data**: Update your `application.conf`:

```hocon
akka {
  distributed-data {
    # Enable the distributed data module
    enable = on
    # Configure the data replication settings
    gossip-interval = 1s
  }
}
```

3. **Using Distributed Data**:

```scala
import akka.actor.ActorSystem
import akka.cluster.Cluster
import akka.cluster.ddata._

object DistributedDataExample extends App {
  implicit val system: ActorSystem = ActorSystem("DistributedDataSystem")
  val cluster = Cluster(system)
  val distributedData = DistributedData(system)

  // Create a distributed data key
  val Key = ORSetKey[String]("myKey")

  // Update the distributed data
  distributedData.update(Key, ORSet("value1")) // Add a value

  // Retrieve the distributed data
  val data = distributedData.get(Key).getOrElse(ORSet.empty)
  println(s"Current values: ${data.elements}") // Outputs: Current values: Set(value1)
}
```

### Eventual Consistency

With distributed data, you can achieve eventual consistency, meaning that all replicas will converge to the same state over time, even in the presence of network partitions or failures.

## Practice Exercises

1. Create an Akka cluster application that uses clustering features to communicate between nodes.

2. Implement a persistent actor that maintains a counter and persists its state.

3. Build a distributed data application that uses Akka's distributed data module to manage shared state across nodes.

4. Write a simple application that demonstrates eventual consistency using Akka's distributed data features.

## Advanced Example: Combining Akka Clustering and Persistence

Let’s create a more complex example that combines Akka Clustering and Persistence to build a distributed counter application:

```scala
import akka.actor.{Actor, ActorSystem, Props}
import akka.cluster.Cluster
import akka.cluster.ClusterEvent._
import akka.persistence._

case class Increment()
case class GetCount(replyTo: ActorRef)

class CounterActor extends PersistentActor {
  override def persistenceId: String = "counter-actor"

  private var count: Int = 0

  def updateCount(increment: Int): Unit = {
    count += increment
  }

  override def receiveRecover: Receive = {
    case Increment() => updateCount(1)
  }

  override def receiveCommand: Receive = {
    case Increment() =>
      persist(Increment()) { _ =>
        updateCount(1)
        println(s"Current count: $count")
      }
    case GetCount(replyTo) =>
      replyTo ! count
  }
}

object DistributedCounterApp extends App {
  val system = ActorSystem("CounterSystem")
  val counterActor = system.actorOf(Props[CounterActor], "counterActor")

  // Simulate increments
  counterActor ! Increment()
  counterActor ! Increment()

  // Get the current count
  counterActor ! GetCount(counterActor)
}
```

In this example:
- We create a `CounterActor` that increments a count and persists its state.
- The actor can respond to commands to increment the count and retrieve the current count.

## Conclusion

Akka provides powerful tools for building advanced distributed systems through features like clustering, persistence, and distributed data. By understanding these concepts, you can create resilient and scalable applications that can handle complex workloads and maintain consistency across distributed nodes.

Mastering these advanced Akka features will enhance your ability to build cloud-native applications and systems that leverage the power of concurrency and distributed computing. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.