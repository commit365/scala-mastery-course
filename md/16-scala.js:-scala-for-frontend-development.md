# Scala.js: Scala for Frontend Development

Scala.js is a powerful tool that allows you to write front-end applications in Scala, which is then compiled to JavaScript. This lesson covers how to compile Scala to JavaScript, optimization techniques, and how to interact with JavaScript libraries and manipulate the DOM.

## Compiling Scala to JavaScript

To get started with Scala.js, you need to set up your project correctly. You can use sbt (Scala Build Tool) to manage your Scala.js project.

### Setting Up a Scala.js Project

1. **Create a new sbt project**:

   Create a new directory for your project and navigate into it:

   ```bash
   mkdir my-scalajs-project
   cd my-scalajs-project
   ```

2. **Create a `build.sbt` file**:

   Add the following content to your `build.sbt` file:

   ```scala
   enablePlugins(ScalaJSPlugin)

   name := "MyScalaJSProject"

   version := "0.1.0"

   scalaVersion := "2.13.6"

   libraryDependencies += "org.scala-js" %% "scalajs-dom" % "1.1.0"
   ```

3. **Create a `src/main/scala` directory**:

   Create the necessary directory structure for your Scala.js code:

   ```bash
   mkdir -p src/main/scala
   ```

4. **Write your Scala.js code**:

   Create a file named `Main.scala` in `src/main/scala` with the following content:

   ```scala
   import org.scalajs.dom
   import scala.scalajs.js
   import scala.scalajs.js.annotation.JSExportTopLevel

   object Main {
     @JSExportTopLevel("main")
     def main(): Unit = {
       dom.document.getElementById("app").textContent = "Hello, Scala.js!"
     }
   }
   ```

5. **Create an HTML file**:

   Create an `index.html` file in the root of your project with the following content:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Scala.js Example</title>
       <script type="text/javascript" src="target/scala-2.13/my-scalajs-project-fastopt.js"></script>
   </head>
   <body>
       <div id="app"></div>
       <script type="text/javascript">
           main(); // Call the main function defined in Scala
       </script>
   </body>
   </html>
   ```

6. **Compile and run your project**:

   Use sbt to compile your project and run it.

   ```bash
   sbt fastOptJS
   ```

   Open `index.html` in your web browser, and you should see "Hello, Scala.js!" displayed on the page.

## Optimization Techniques

When compiling Scala to JavaScript, there are several optimization techniques you can use to improve performance:

1. **Use `fullOptJS` for production builds**:

   The `fastOptJS` command is useful for development, but for production, you should use `fullOptJS`, which applies optimizations to reduce the size of the generated JavaScript.

   ```bash
   sbt fullOptJS
   ```

2. **Minimize the use of reflection**:

   Reflection can slow down your application and increase the size of the generated JavaScript. Try to use static types and avoid dynamic features when possible.

3. **Leverage Scala.js libraries**:

   Use libraries designed for Scala.js, such as `scalajs-dom` for DOM manipulation, which are optimized for performance.

4. **Code splitting**:

   If your application is large, consider using code splitting to load parts of your application on demand, reducing the initial load time.

## Interoperability with JavaScript Libraries

Scala.js allows you to seamlessly integrate with existing JavaScript libraries. You can call JavaScript functions and use JavaScript objects directly from your Scala code.

### Importing JavaScript Libraries

You can use the `@JSImport` annotation to import JavaScript libraries.

```scala
import scala.scalajs.js
import scala.scalajs.js.annotation.JSImport

@JSImport("lodash", JSImport.Namespace)
@js.native
object Lodash extends js.Object {
  def shuffle[T](array: js.Array[T]): js.Array[T] = js.native
}
```

### Using JavaScript Libraries

You can use the imported JavaScript libraries in your Scala code.

```scala
object Main {
  @JSExportTopLevel("main")
  def main(): Unit = {
    val nums = js.Array(1, 2, 3, 4, 5)
    val shuffled = Lodash.shuffle(nums)
    dom.console.log(shuffled.toString())
    dom.document.getElementById("app").textContent = "Shuffled numbers: " + shuffled.mkString(", ")
  }
}
```

### DOM Manipulation

Scala.js provides a convenient way to manipulate the DOM using the `scalajs-dom` library.

```scala
import org.scalajs.dom
import org.scalajs.dom.document

object Main {
  @JSExportTopLevel("main")
  def main(): Unit = {
    val button = document.createElement("button")
    button.textContent = "Click me!"
    button.onclick = (_: dom.MouseEvent) => {
      dom.window.alert("Button clicked!")
    }
    document.body.appendChild(button)
  }
}
```

## Practice Exercises

1. Create a Scala.js application that fetches data from a public API and displays it on the webpage.

2. Implement a simple to-do list application using Scala.js that allows users to add and remove tasks.

3. Use a JavaScript library (like jQuery or Lodash) in your Scala.js project to manipulate the DOM or perform data manipulation.

4. Optimize your Scala.js application by using `fullOptJS` and analyze the generated JavaScript file size.

## Advanced Example: Combining Scala.js Features

Let’s create a more complex example that combines various Scala.js features, including DOM manipulation and external library integration:

```scala
import org.scalajs.dom
import org.scalajs.dom.document
import scala.scalajs.js
import scala.scalajs.js.annotation.JSExportTopLevel
import scala.scalajs.js.annotation.JSImport

@JSImport("axios", JSImport.Namespace)
@js.native
object Axios extends js.Object {
  def get(url: String): js.Promise[js.Object] = js.native
}

object Main {
  @JSExportTopLevel("main")
  def main(): Unit = {
    val fetchButton = document.createElement("button")
    fetchButton.textContent = "Fetch Data"
    fetchButton.onclick = (_: dom.MouseEvent) => fetchData()
    document.body.appendChild(fetchButton)

    val resultDiv = document.createElement("div")
    resultDiv.id = "result"
    document.body.appendChild(resultDiv)
  }

  def fetchData(): Unit = {
    val url = "https://jsonplaceholder.typicode.com/posts/1"
    Axios.get(url).toFuture.onComplete {
      case Success(data) =>
        val resultDiv = document.getElementById("result")
        resultDiv.textContent = s"Fetched Data: ${js.JSON.stringify(data)}"
      case Failure(exception) =>
        dom.console.error(s"Failed to fetch data: $exception")
    }
  }
}
```

This example demonstrates:
- Using an external JavaScript library (`axios`) to fetch data from an API.
- Manipulating the DOM to create buttons and display results.
- Handling asynchronous operations with promises.

## Conclusion

Scala.js provides a powerful way to develop front-end applications using Scala. By compiling Scala to JavaScript, you can leverage the language’s features while integrating with existing JavaScript libraries and manipulating the DOM.

Mastering Scala.js allows you to build robust, type-safe web applications that can benefit from the rich ecosystem of Scala libraries. In the next lesson, we will explore additional advanced topics in Scala, including performance tuning and best practices for building scalable applications.