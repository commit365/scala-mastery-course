# Domain-Specific Languages (DSLs) in Scala

Domain-Specific Languages (DSLs) are specialized languages tailored to a specific problem domain. Scala's expressive syntax and advanced features make it an excellent choice for creating both internal and external DSLs. This lesson covers how to create internal DSLs using method chaining, implicit classes, and type classes, as well as how to implement external DSLs using parser combinators and FastParse.

## Internal DSLs

Internal DSLs are built using the host language's syntax and features. In Scala, you can create internal DSLs using method chaining, implicit classes, and type classes.

### Method Chaining

Method chaining allows you to call multiple methods on the same object in a single expression, making the code more readable.

```scala
class QueryBuilder {
  private var query: String = ""

  def select(columns: String): this.type = {
    query += s"SELECT $columns "
    this
  }

  def from(table: String): this.type = {
    query += s"FROM $table "
    this
  }

  def where(condition: String): this.type = {
    query += s"WHERE $condition "
    this
  }

  def build(): String = query.trim + ";"
}

// Usage
val query = new QueryBuilder()
  .select("name, age")
  .from("users")
  .where("age > 18")
  .build()

println(query) // Outputs: SELECT name, age FROM users WHERE age > 18;
```

### Implicit Classes

Implicit classes allow you to add methods to existing types, enhancing the readability of your DSL.

```scala
object QueryDSL {
  implicit class QueryOps(val queryBuilder: QueryBuilder) extends AnyVal {
    def and(condition: String): QueryBuilder = {
      queryBuilder.where(condition)
    }
  }
}

// Usage
import QueryDSL._

val queryWithAnd = new QueryBuilder()
  .select("name, age")
  .from("users")
  .where("age > 18")
  .and("name LIKE 'A%'")
  .build()

println(queryWithAnd) // Outputs: SELECT name, age FROM users WHERE age > 18 AND name LIKE 'A%';
```

### Type Classes

Type classes can be used to define behavior for different types, allowing for more flexible and extensible DSLs.

```scala
trait Jsonable[A] {
  def toJson(value: A): String
}

object Jsonable {
  implicit val stringJsonable: Jsonable[String] = (value: String) => s""""$value""""
  implicit val intJsonable: Jsonable[Int] = (value: Int) => value.toString

  def toJson[A](value: A)(implicit jsonable: Jsonable[A]): String = jsonable.toJson(value)
}

// Usage
val jsonString = Jsonable.toJson("Hello, World!")
val jsonInt = Jsonable.toJson(42)

println(jsonString) // Outputs: "Hello, World!"
println(jsonInt)    // Outputs: 42
```

## External DSLs

External DSLs are standalone languages designed for specific tasks. Scala provides tools like parser combinators and FastParse to create external DSLs.

### Parser Combinators

Parser combinators allow you to build parsers for your DSL using a compositional approach.

```scala
import scala.util.parsing.combinator._

object SimpleDSL extends RegexParsers {
  def expr: Parser[Int] = term ~ rep("+" ~> term) ^^ {
    case t ~ ts => ts.foldLeft(t) { (acc, next) => acc + next }
  }

  def term: Parser[Int] = wholeNumber ^^ { _.toInt }

  def parseExpression(input: String): Int = {
    parseAll(expr, input) match {
      case Success(result, _) => result
      case Failure(msg, _) => throw new IllegalArgumentException(s"Parsing failed: $msg")
    }
  }
}

// Usage
val result = SimpleDSL.parseExpression("3 + 4 + 5")
println(result) // Outputs: 12
```

### FastParse

FastParse is a high-performance library for parsing text. It provides a more efficient way to define parsers compared to traditional parser combinators.

1. **Add the FastParse dependency** to your `build.sbt`:

   ```sbt
   libraryDependencies += "com.lihaoyi" %% "fastparse" % "2.3.3"
   ```

2. **Define a parser using FastParse**:

```scala
import fastparse._
import NoWhitespace._

object FastParseDSL {
  def expr[_: P]: P[Int] = term ~ P("+".? ~ term).rep() map {
    case (t, ts) => ts.foldLeft(t) { case (acc, (_, t2)) => acc + t2 }
  }

  def term[_: P]: P[Int] = P(CharsWhileIn("0-9").!.map(_.toInt))

  def parseExpression(input: String): Int = {
    parse(input, expr(_)) match {
      case Parsed.Success(value, _) => value
      case Parsed.Failure(label, index, extra) => throw new IllegalArgumentException("Parsing failed")
    }
  }
}

// Usage
val result = FastParseDSL.parseExpression("3 + 4 + 5")
println(result) // Outputs: 12
```

## Practice Exercises

1. Create an internal DSL for building SQL queries that supports `JOIN` operations.

2. Implement an implicit class that adds a method to convert a string to uppercase in a more readable way.

3. Write a parser combinator for a simple arithmetic expression language that supports addition and multiplication.

4. Use FastParse to create a parser for a configuration file format (e.g., key-value pairs).

## Advanced Example: Combining Internal and External DSLs

Let’s create an example that combines both internal and external DSLs to build a simple configuration system:

```scala
import scala.util.parsing.combinator._
import scala.language.implicitConversions

// Internal DSL for configuration
class ConfigBuilder {
  private var config: Map[String, String] = Map()

  def set(key: String, value: String): this.type = {
    config += (key -> value)
    this
  }

  def get(key: String): Option[String] = config.get(key)

  def build(): Map[String, String] = config
}

// Parser combinators for external DSL
object ConfigParser extends RegexParsers {
  def entry: Parser[(String, String)] = ident ~ "=" ~ ident ^^ {
    case key ~ "=" ~ value => (key, value)
  }

  def config: Parser[List[(String, String)]] = rep(entry)

  def parseConfig(input: String): List[(String, String)] = {
    parseAll(config, input) match {
      case Success(result, _) => result
      case Failure(msg, _) => throw new IllegalArgumentException(s"Parsing failed: $msg")
    }
  }
}

// Usage
val configInput = """key1=value1
                    |key2=value2""".stripMargin

val parsedConfig = ConfigParser.parseConfig(configInput)

val configBuilder = new ConfigBuilder()
parsedConfig.foreach { case (key, value) => configBuilder.set(key, value) }

val config = configBuilder.build()
println(config) // Outputs: Map(key1 -> value1, key2 -> value2)
```

In this example:
- We create an internal DSL for building a configuration using method chaining.
- We define an external DSL for parsing configuration entries using parser combinators.

## Conclusion

Scala's powerful language features enable the creation of expressive and flexible Domain-Specific Languages (DSLs). By using internal DSLs with method chaining, implicit classes, and type classes, you can enhance the readability and usability of your APIs. External DSLs can be implemented using parser combinators and FastParse, allowing for the definition of custom languages tailored to specific tasks.

Mastering these DSL techniques can significantly improve the expressiveness of your code and make it easier to work with complex domains. In the next lesson, we will explore best practices for building scalable applications in Scala, focusing on design patterns and architectural principles.