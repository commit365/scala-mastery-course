# Capstone Project: Designing and Implementing a Full-Scale, Production-Ready Scala Application

In this final lesson, we will bring together all the concepts learned throughout the course to design and implement a full-scale, production-ready Scala application. We will cover the application architecture, implementation details, code review practices, optimization techniques, and deployment strategies.

## Project Overview

For this capstone project, we will build a simple e-commerce application that allows users to browse products, place orders, and manage their accounts. The application will consist of the following components:

1. **Backend API**: Built using Akka HTTP for handling requests.
2. **Database**: Using an embedded database like H2 or a more robust solution like PostgreSQL.
3. **Frontend**: A simple web interface using Scala.js or a REST client for testing the API.
4. **Deployment**: Containerized using Docker and deployed on a cloud platform like AWS or Google Cloud.

### Application Architecture

The architecture of the application will follow a microservices approach, with the following components:

- **Product Service**: Manages product listings and details.
- **Order Service**: Handles order placement and tracking.
- **User Service**: Manages user accounts and authentication.
- **Database**: Stores product, order, and user information.

## Step 1: Designing the API

### Define the API Endpoints

Using Akka HTTP, we will define the following endpoints:

- **Product Service**:
  - `GET /products`: Retrieve a list of products.
  - `GET /products/{id}`: Retrieve details of a specific product.
  - `POST /products`: Add a new product.

- **Order Service**:
  - `POST /orders`: Place a new order.
  - `GET /orders/{id}`: Retrieve order details.

- **User Service**:
  - `POST /users`: Create a new user account.
  - `GET /users/{id}`: Retrieve user details.

### Example of Product Service Implementation

```scala
import akka.http.scaladsl.server.Directives._
import akka.http.scaladsl.model.StatusCodes
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport._
import spray.json.DefaultJsonProtocol._

case class Product(id: Int, name: String, price: Double)

object ProductJsonProtocol {
  implicit val productFormat = jsonFormat3(Product)
}

class ProductService {
  import ProductJsonProtocol._

  private var products = List(
    Product(1, "Laptop", 999.99),
    Product(2, "Smartphone", 499.99)
  )

  val route =
    pathPrefix("products") {
      concat(
        pathEndOrSingleSlash {
          get {
            complete(products)
          } ~
          post {
            entity(as[Product]) { product =>
              products = products :+ product
              complete(StatusCodes.Created, product)
            }
          }
        },
        path(IntNumber) { id =>
          get {
            products.find(_.id == id) match {
              case Some(product) => complete(product)
              case None => complete(StatusCodes.NotFound)
            }
          }
        }
      )
    }
}
```

## Step 2: Implementing the Application

### Setting Up the Project Structure

1. **Create the project directory**:

```bash
mkdir e-commerce-app
cd e-commerce-app
```

2. **Create the `build.sbt` file**:

```sbt
name := "ECommerceApp"

version := "0.1.0"

scalaVersion := "2.13.6"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-http" % "10.6.3",
  "com.typesafe.akka" %% "akka-stream" % "2.6.16",
  "com.typesafe.akka" %% "akka-http-spray-json" % "10.6.3",
  "com.h2database" % "h2" % "1.4.200",
  "org.postgresql" % "postgresql" % "42.2.20"
)
```

3. **Implement the Services**: Create separate files for each service (ProductService, OrderService, UserService) and implement their routes similarly to the ProductService example.

### Connecting to the Database

You can use JDBC for connecting to H2 or PostgreSQL. Here’s a simple example of connecting to a database:

```scala
import java.sql.{Connection, DriverManager}

object Database {
  def getConnection: Connection = {
    DriverManager.getConnection("jdbc:h2:mem:test;DB_CLOSE_DELAY=-1", "sa", "")
  }
}
```

### Running the Application

To run the application, create a main entry point:

```scala
import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.stream.ActorMaterializer

object Main extends App {
  implicit val system: ActorSystem = ActorSystem("ECommerceSystem")
  implicit val materializer: ActorMaterializer = ActorMaterializer()

  val productService = new ProductService()
  val orderService = new OrderService() // Implement similarly
  val userService = new UserService() // Implement similarly

  val routes = productService.route ~ orderService.route ~ userService.route

  Http().bindAndHandle(routes, "localhost", 8080)
  println("Server online at http://localhost:8080/")
}
```

## Step 3: Code Review and Optimization Techniques

### Code Review Practices

1. **Peer Review**: Have team members review each other’s code for readability, maintainability, and adherence to coding standards.

2. **Automated Code Analysis**: Use tools like Scalastyle or Scalafix to enforce coding standards.

3. **Testing**: Ensure that unit tests and integration tests are written and pass successfully.

### Optimization Techniques

1. **Profiling**: Use profiling tools like VisualVM or YourKit to identify bottlenecks in your application.

2. **Database Optimization**: Optimize database queries by indexing frequently accessed columns and using efficient joins.

3. **Caching**: Implement caching strategies for frequently accessed data to reduce database load.

4. **Concurrency**: Utilize Akka actors to handle concurrent requests efficiently.

## Step 4: Deployment Strategies

### Containerization with Docker

1. **Create a Dockerfile**:

```dockerfile
FROM openjdk:11-jre-slim
COPY target/scala-2.13/e-commerce-app-assembly-0.1.0.jar /app/e-commerce-app.jar
CMD ["java", "-jar", "/app/e-commerce-app.jar"]
```

2. **Build the Docker Image**:

```bash
sbt assembly
docker build -t e-commerce-app .
```

3. **Run the Docker Container**:

```bash
docker run -p 8080:8080 e-commerce-app
```

### Deployment on Cloud Platforms

You can deploy your application on cloud platforms such as AWS, Google Cloud, or Azure. Use services like AWS ECS, Google Kubernetes Engine, or Azure App Service to manage your containers.

1. **Deploying to AWS ECS**:
   - Push your Docker image to Amazon ECR (Elastic Container Registry).
   - Create an ECS task definition and service to run your container.

2. **Deploying to Google Kubernetes Engine**:
   - Push your Docker image to Google Container Registry.
   - Create a Kubernetes deployment and service to manage your application.

## Practice Exercises

1. Expand the e-commerce application by adding an order service that handles order placement and retrieval.

2. Implement user authentication and authorization for the e-commerce application.

3. Write unit tests for your services and ensure they are covered by automated tests.

4. Create a Docker Compose file to run your application with a database.

## Advanced Example: Complete E-Commerce Application

Let’s create a more complex example that combines all the components discussed:

```scala
import akka.http.scaladsl.server.Directives._
import akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport._
import spray.json.DefaultJsonProtocol._
import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.stream.ActorMaterializer

case class Product(id: Int, name: String, price: Double)
case class Order(id: Int, productId: Int, quantity: Int)
case class User(id: Int, username: String)

object JsonFormats {
  implicit val productFormat = jsonFormat3(Product)
  implicit val orderFormat = jsonFormat3(Order)
  implicit val userFormat = jsonFormat2(User)
}

class ProductService {
  import JsonFormats._

  private var products = List(Product(1, "Laptop", 999.99), Product(2, "Smartphone", 499.99))

  val route =
    pathPrefix("products") {
      concat(
        pathEndOrSingleSlash {
          get {
            complete(products)
          } ~
          post {
            entity(as[Product]) { product =>
              products = products :+ product
              complete(StatusCodes.Created, product)
            }
          }
        },
        path(IntNumber) { id =>
          get {
            products.find(_.id == id) match {
              case Some(product) => complete(product)
              case None => complete(StatusCodes.NotFound)
            }
          }
        }
      )
    }
}

class OrderService {
  import JsonFormats._

  private var orders = List[Order]()

  val route =
    pathPrefix("orders") {
      post {
        entity(as[Order]) { order =>
          orders = orders :+ order
          complete(StatusCodes.Created, order)
        }
      }
    }
}

object ECommerceApp extends App {
  implicit val system: ActorSystem = ActorSystem("ECommerceSystem")
  implicit val materializer: ActorMaterializer = ActorMaterializer()

  val productService = new ProductService()
  val orderService = new OrderService()

  val routes = productService.route ~ orderService.route

  Http().bindAndHandle(routes, "localhost", 8080)
  println("Server online at http://localhost:8080/")
}
```

In this example:
- We define a complete e-commerce application with product and order services.
- We implement JSON serialization for the data models using Spray JSON.

## Conclusion

In this capstone project, we have designed and implemented a full-scale, production-ready Scala application. We covered the architecture, implementation details, code review practices, optimization techniques, and deployment strategies. By integrating various components and best practices, you now have a solid foundation for building robust applications in Scala.

Mastering these concepts will empower you to develop effective software solutions that are scalable, maintainable, and ready for production. In the next steps, consider exploring additional advanced topics in Scala and continuously improving your skills in software development.