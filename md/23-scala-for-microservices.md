# Scala for Microservices

Scala is an excellent choice for building microservices due to its expressive syntax, powerful concurrency features, and robust ecosystem. This lesson covers how to build microservices using Akka HTTP and Lagom, as well as service discovery, deployment, and containerization techniques.

## Building Microservices with Akka HTTP

Akka HTTP is a toolkit for building HTTP-based services using the Akka actor model. It provides a flexible and powerful way to handle HTTP requests and responses.

### Setting Up Akka HTTP

1. **Create a new sbt project**:

   Create a new directory for your project and navigate into it:

   ```bash
   mkdir my-akka-http-microservice
   cd my-akka-http-microservice
   ```

2. **Create a `build.sbt` file**:

   Add the following content to your `build.sbt` file:

   ```scala
   name := "MyAkkaHttpMicroservice"

   version := "0.1.0"

   scalaVersion := "2.13.6"

   libraryDependencies += "com.typesafe.akka" %% "akka-http" % "10.6.3"
   libraryDependencies += "com.typesafe.akka" %% "akka-stream" % "2.6.16"
   ```

3. **Create a simple Akka HTTP service**:

   Create a file named `Main.scala` in `src/main/scala` with the following content:

   ```scala
   import akka.actor.ActorSystem
   import akka.http.scaladsl.Http
   import akka.http.scaladsl.model._
   import akka.http.scaladsl.server.Directives._
   import akka.stream.ActorMaterializer
   import scala.concurrent.ExecutionContextExecutor

   object Main extends App {
     implicit val system: ActorSystem = ActorSystem("my-system")
     implicit val materializer: ActorMaterializer = ActorMaterializer()
     implicit val executionContext: ExecutionContextExecutor = system.dispatcher

     val route =
       path("hello") {
         get {
           complete(HttpResponse(entity = "Hello, World!"))
         }
       }

     Http().bindAndHandle(route, "localhost", 8080)
     println("Server online at http://localhost:8080/")
   }
   ```

4. **Run the service**:

   Use sbt to run your service:

   ```bash
   sbt run
   ```

   You can test the service by navigating to `http://localhost:8080/hello` in your web browser.

### Handling JSON

To handle JSON in Akka HTTP, you can use the `akka-http-spray-json` library for marshalling and unmarshalling JSON data.

1. **Add the dependency** to your `build.sbt`:

   ```scala
   libraryDependencies += "com.typesafe.akka" %% "akka-http-spray-json" % "10.6.3"
   ```

2. **Define case classes and JSON formats**:

```scala
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport._
import spray.json.DefaultJsonProtocol._

case class User(name: String, age: Int)

implicit val userFormat = jsonFormat2(User)
```

3. **Update your route to handle JSON**:

```scala
val route =
  path("user") {
    post {
      entity(as[User]) { user =>
        complete(s"Received user: ${user.name}, age: ${user.age}")
      }
    }
  }
```

## Building Microservices with Lagom

Lagom is a framework specifically designed for building microservices in Scala. It provides built-in support for service discovery, messaging, and persistence.

### Setting Up Lagom

1. **Create a new Lagom project**:

   Use the Lagom template to create a new project:

   ```bash
   sbt new lagom/lagom-scala.g8
   ```

2. **Follow the prompts** to set up your project.

### Creating a Service

1. **Define a service interface**:

   In the `api` module, create a service interface:

```scala
package com.example.hello.api

import akka.Done
import play.api.libs.json.{Format, Json}
import lagom.api.ScalaService
import lagom.api.ServiceCall

trait HelloService extends ScalaService {
  def hello(name: String): ServiceCall[NotUsed, String]
}

object HelloService {
  case class Greeting(message: String)

  implicit val format: Format[Greeting] = Json.format[Greeting]
}
```

2. **Implement the service**:

   In the `impl` module, implement the service:

```scala
package com.example.hello.impl

import com.example.hello.api.HelloService
import akka.NotUsed
import com.lightbend.lagom.scaladsl.api.ServiceCall
import javax.inject.Inject
import scala.concurrent.Future

class HelloServiceImpl @Inject()() extends HelloService {
  override def hello(name: String): ServiceCall[NotUsed, String] = ServiceCall { _ =>
    Future.successful(s"Hello, $name!")
  }
}
```

3. **Run the Lagom service**:

   Use sbt to run your Lagom service:

   ```bash
   sbt runAll
   ```

   You can access the service at `http://localhost:9000/hello/<name>`.

## Service Discovery

Both Akka HTTP and Lagom support service discovery, which allows microservices to locate and communicate with each other dynamically.

### Using Lagom's Built-in Service Locator

Lagom provides a built-in service locator that allows services to discover each other using their service names.

1. **Define services in `application.conf`**:

```hocon
lagom.service-locator {
  # Configuration for the service locator
}
```

2. **Use the service locator in your service implementation**:

```scala
import com.lightbend.lagom.scaladsl.api.ServiceLocator

class MyServiceImpl @Inject()(serviceLocator: ServiceLocator) extends MyService {
  // Use the service locator to discover other services
}
```

## Deployment and Containerization

Microservices are typically deployed in containers for scalability and isolation. Docker is the most common tool for containerization.

### Creating a Dockerfile

1. **Create a `Dockerfile`** in the root of your project:

```dockerfile
FROM openjdk:11-jre-slim
COPY target/universal/stage /opt/myapp
WORKDIR /opt/myapp
CMD ["bin/myapp"]
```

2. **Build the Docker image**:

```bash
docker build -t myapp .
```

3. **Run the Docker container**:

```bash
docker run -p 8080:8080 myapp
```

### Orchestrating with Kubernetes

For larger systems, you can use Kubernetes to manage your microservices.

1. **Create a Kubernetes deployment**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
```

2. **Create a service to expose your application**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

## Practice Exercises

1. Build a simple microservice using Akka HTTP that provides a REST API for managing a list of items.

2. Create a Lagom microservice that interacts with another Lagom service to fetch and display user information.

3. Implement service discovery for your Akka HTTP microservice using a service registry like Consul.

4. Containerize your Lagom microservice using Docker and deploy it to a local Kubernetes cluster.

## Advanced Example: Combining Microservices

Let’s create a more complex example that combines Akka HTTP and Lagom to build a microservice architecture:

```scala
// User Service API
package com.example.user.api

import akka.NotUsed
import lagom.api.ScalaService
import lagom.api.ServiceCall

trait UserService extends ScalaService {
  def getUser(id: String): ServiceCall[NotUsed, User]
}

// User case class
case class User(id: String, name: String)

// User Service Implementation
package com.example.user.impl

import com.example.user.api.UserService
import akka.NotUsed
import com.lightbend.lagom.scaladsl.api.ServiceCall
import javax.inject.Inject
import scala.concurrent.Future

class UserServiceImpl @Inject()() extends UserService {
  private val users = Map("1" -> User("1", "Alice"), "2" -> User("2", "Bob"))

  override def getUser(id: String): ServiceCall[NotUsed, User] = ServiceCall { _ =>
    Future.successful(users.get(id).getOrElse(User("unknown", "Unknown")))
  }
}

// Main application entry point
object Main extends App {
  // Start the Lagom application
}
```

In this example:
- We define a `UserService` API for fetching user information.
- We implement the service to return user data based on the provided ID.

## Conclusion

Scala provides powerful tools for building microservices through Akka HTTP and Lagom. With built-in support for service discovery, deployment, and containerization, you can create scalable and maintainable microservice architectures.

By mastering these concepts and technologies, you can leverage Scala's strengths to build robust applications that can efficiently handle complex business requirements. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.
