# Scala and Cloud Computing

Scala is a powerful language for building cloud-native applications, leveraging its functional programming capabilities and strong type system. This lesson covers building serverless applications with Scala using AWS Lambda and Google Cloud Functions, as well as deploying Scala applications in a cloud-native environment with containerization techniques.

## Serverless Scala with AWS Lambda

AWS Lambda is a serverless compute service that allows you to run code in response to events without provisioning or managing servers. You can write your functions in Scala and deploy them to AWS Lambda.

### Setting Up AWS Lambda for Scala

1. **Create a new Scala project**:

   Use sbt to create a new Scala project:

   ```bash
   mkdir my-lambda-scala
   cd my-lambda-scala
   ```

2. **Create a `build.sbt` file**:

   Add the following content to your `build.sbt` file:

   ```sbt
   name := "MyLambdaScala"

   version := "0.1.0"

   scalaVersion := "2.13.6"

   libraryDependencies += "com.amazonaws" % "aws-lambda-java-core" % "1.2.1"
   libraryDependencies += "com.amazonaws" % "aws-lambda-java-events" % "3.2.1"
   ```

3. **Implement a Lambda Function**:

   Create a file named `Handler.scala` in `src/main/scala` with the following content:

```scala
import com.amazonaws.services.lambda.runtime.{Context, RequestHandler}

class Handler extends RequestHandler[String, String] {
  override def handleRequest(input: String, context: Context): String = {
    s"Hello, $input!"
  }
}
```

4. **Package the Application**:

   Use sbt to create a JAR file for your Lambda function:

   ```bash
   sbt assembly
   ```

5. **Deploy to AWS Lambda**:

   You can deploy your JAR file to AWS Lambda using the AWS Management Console or AWS CLI. 

   Example using AWS CLI:

   ```bash
   aws lambda create-function --function-name MyLambdaFunction \
     --zip-file fileb://target/scala-2.13/my-lambda-scala-assembly-0.1.0.jar \
     --handler Handler::handleRequest \
     --runtime java11 \
     --role arn:aws:iam::your-account-id:role/your-lambda-role
   ```

### Testing Your Lambda Function

You can test your Lambda function using the AWS Management Console or AWS CLI. 

Example using AWS CLI:

```bash
aws lambda invoke --function-name MyLambdaFunction --payload '"World"' output.txt
cat output.txt  # Outputs: Hello, World!
```

## Serverless Scala with Google Cloud Functions

Google Cloud Functions is another serverless compute service that allows you to run code in response to events. You can also use Scala for building functions in Google Cloud.

### Setting Up Google Cloud Functions for Scala

1. **Create a new Scala project** (similar to the AWS setup).

2. **Implement a Cloud Function**:

   Create a file named `Function.scala` in `src/main/scala`:

```scala
import com.google.cloud.functions.{HttpFunction, HttpRequest, HttpResponse}

class Function extends HttpFunction {
  override def service(request: HttpRequest, response: HttpResponse): Unit = {
    val name = request.getFirstQueryParameter("name").orElse("World")
    response.getWriter.write(s"Hello, $name!")
  }
}
```

3. **Deploy to Google Cloud Functions**:

   Use the Google Cloud SDK to deploy your function:

   ```bash
   gcloud functions deploy HelloFunction \
     --entry-point Function \
     --runtime java11 \
     --trigger-http \
     --allow-unauthenticated
   ```

### Testing Your Google Cloud Function

You can test your deployed function using the URL provided by Google Cloud:

```bash
curl "https://REGION-PROJECT_ID.cloudfunctions.net/HelloFunction?name=Scala"
```

## Scala for Cloud-Native Applications

Scala is well-suited for building cloud-native applications, which are designed to take advantage of cloud computing environments. This involves using microservices, containerization, and orchestration.

### Containerization with Docker

Containerization allows you to package your applications and their dependencies into a single container image, ensuring consistency across environments.

1. **Create a Dockerfile**:

   In your project root, create a `Dockerfile`:

```dockerfile
FROM openjdk:11-jre-slim
COPY target/scala-2.13/my-lambda-scala-assembly-0.1.0.jar /app/my-lambda-scala.jar
CMD ["java", "-jar", "/app/my-lambda-scala.jar"]
```

2. **Build the Docker Image**:

   Run the following command to build your Docker image:

```bash
docker build -t my-lambda-scala .
```

3. **Run the Docker Container**:

   You can run your container locally to test it:

```bash
docker run -p 8080:8080 my-lambda-scala
```

### Orchestrating with Kubernetes

Kubernetes is a powerful platform for managing containerized applications. You can deploy your Scala applications in a Kubernetes cluster for scalability and resilience.

1. **Create a Kubernetes Deployment**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-lambda-scala
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-lambda-scala
  template:
    metadata:
      labels:
        app: my-lambda-scala
    spec:
      containers:
      - name: my-lambda-scala
        image: my-lambda-scala:latest
        ports:
        - containerPort: 8080
```

2. **Create a Service to Expose Your Application**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lambda-scala
spec:
  selector:
    app: my-lambda-scala
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: LoadBalancer
```

3. **Deploy to Kubernetes**:

Use `kubectl` to deploy your application:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Practice Exercises

1. Create a serverless function using AWS Lambda that processes incoming data and returns a response.

2. Implement a Google Cloud Function that interacts with a database and returns data based on a query parameter.

3. Containerize a Scala application using Docker and deploy it to a local Kubernetes cluster.

4. Build a cloud-native microservice using Akka HTTP and deploy it on Kubernetes.

## Advanced Example: Building a Cloud-Native Microservice

Let’s create a more complex example that combines AWS Lambda with a cloud-native architecture:

```scala
import com.amazonaws.services.lambda.runtime.{Context, RequestHandler}
import scala.collection.JavaConverters._

class MyLambdaFunction extends RequestHandler[Map[String, String], String] {
  override def handleRequest(input: Map[String, String], context: Context): String = {
    val name = input.getOrElse("name", "World")
    s"Hello, $name!"
  }
}

// Deploy the function to AWS Lambda as described earlier
```

In this example:
- We define a simple AWS Lambda function that takes a map as input and returns a greeting.
- This function can be deployed to AWS Lambda and invoked with different parameters.

## Conclusion

Scala is a powerful language for building cloud-native applications and serverless functions. By leveraging frameworks like AWS Lambda and Google Cloud Functions, you can create scalable and efficient applications without managing infrastructure. Additionally, containerization with Docker and orchestration with Kubernetes provide flexibility and resilience for deploying Scala applications in the cloud.

Mastering these concepts will enhance your ability to build modern applications that take full advantage of cloud computing. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.
