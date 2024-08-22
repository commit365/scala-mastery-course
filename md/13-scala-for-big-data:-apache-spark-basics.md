# Scala for Big Data: Apache Spark Basics

Apache Spark is a powerful, open-source distributed computing system that provides an interface for programming entire clusters with implicit data parallelism and fault tolerance. This lesson covers the basics of Spark, focusing on RDDs, DataFrames, Datasets, and basic operations.

## RDDs (Resilient Distributed Datasets)

RDDs are the fundamental data structure in Spark. They represent an immutable, distributed collection of objects that can be processed in parallel.

### Creating RDDs

You can create RDDs from existing data in memory or by loading data from external storage systems.

```scala
import org.apache.spark.{SparkConf, SparkContext}

// Create a Spark configuration and context
val conf = new SparkConf().setAppName("RDD Example").setMaster("local[*]")
val sc = new SparkContext(conf)

// Creating an RDD from a collection
val numbersRDD = sc.parallelize(Seq(1, 2, 3, 4, 5))

// Creating an RDD from a text file
val textRDD = sc.textFile("path/to/textfile.txt")
```

### RDD Operations

RDDs support two types of operations: transformations and actions.

#### Transformations

Transformations create a new RDD from an existing one. They are lazy, meaning they are not executed until an action is called.

```scala
// Map transformation: multiply each element by 2
val doubledRDD = numbersRDD.map(_ * 2)

// Filter transformation: keep only even numbers
val evenRDD = numbersRDD.filter(_ % 2 == 0)
```

#### Actions

Actions trigger the execution of transformations and return results to the driver program.

```scala
// Collect action: return all elements as an array
val collected = doubledRDD.collect()
println(collected.mkString(", ")) // Outputs: 2, 4, 6, 8, 10

// Count action: return the number of elements
val count = numbersRDD.count()
println(count) // Outputs: 5

// Save action: save the RDD to a text file
doubledRDD.saveAsTextFile("path/to/output.txt")
```

## DataFrames

DataFrames are distributed collections of data organized into named columns. They provide a higher-level abstraction than RDDs and allow for more optimized execution.

### Creating DataFrames

You can create DataFrames from existing RDDs, structured data files, or external databases.

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("DataFrame Example")
  .master("local[*]")
  .getOrCreate()

// Creating a DataFrame from an existing RDD
import spark.implicits._

val df = numbersRDD.toDF("numbers")
df.show() // Displays the DataFrame
```

### DataFrame Operations

DataFrames support a wide range of operations, including filtering, grouping, and aggregating.

```scala
// Filter operation
val filteredDF = df.filter($"numbers" > 2)
filteredDF.show() // Displays numbers greater than 2

// GroupBy and aggregation
val countDF = df.groupBy("numbers").count()
countDF.show() // Displays the count of each number
```

## Datasets

Datasets are a distributed collection of data that provides the benefits of both RDDs and DataFrames. They are strongly typed and allow for compile-time type safety.

### Creating Datasets

You can create Datasets from DataFrames or existing collections.

```scala
case class Person(name: String, age: Int)

val peopleDS = Seq(Person("Alice", 30), Person("Bob", 25)).toDS()
peopleDS.show() // Displays the Dataset
```

### Dataset Operations

Datasets support the same operations as DataFrames, with the added benefit of compile-time type safety.

```scala
// Filter operation
val adultsDS = peopleDS.filter(_.age >= 18)
adultsDS.show() // Displays only adults

// Map operation
val namesDS = peopleDS.map(_.name)
namesDS.show() // Displays names of all people
```

## Basic Spark Operations

### Transformations

Transformations create a new dataset from an existing one. Common transformations include:

- `map`: Applies a function to each element.
- `filter`: Keeps elements that satisfy a predicate.
- `flatMap`: Similar to map, but flattens the results.
- `reduceByKey`: Combines values with the same key.

Example of using `reduceByKey` with RDDs:

```scala
val pairsRDD = sc.parallelize(Seq(("a", 1), ("b", 1), ("a", 1)))
val countsRDD = pairsRDD.reduceByKey(_ + _)
countsRDD.collect().foreach(println) // Outputs: (a,2), (b,1)
```

### Actions

Actions trigger the execution of transformations and return results. Common actions include:

- `collect`: Returns all elements as an array.
- `count`: Returns the number of elements.
- `first`: Returns the first element.
- `take(n)`: Returns the first n elements.

Example of using `collect` with DataFrames:

```scala
val collectedData = df.collect()
collectedData.foreach(row => println(row.getInt(0))) // Outputs each number in the DataFrame
```

## Practice Exercises

1. Create an RDD from a list of strings and use transformations to filter out strings shorter than 5 characters.

2. Load a CSV file into a DataFrame and perform a group-by operation on a specific column.

3. Create a Dataset of case class instances and write a function to calculate the average age of the people in the Dataset.

4. Implement a Spark application that reads a text file, counts the occurrences of each word, and saves the results to an output file.

## Advanced Example: Combining RDDs, DataFrames, and Datasets

Let's create a more complex example that combines RDDs, DataFrames, and Datasets:

```scala
// Load data from a text file into an RDD
val textRDD = sc.textFile("path/to/textfile.txt")

// Transform RDD into a DataFrame by splitting lines into words
val wordsDF = textRDD.flatMap(line => line.split(" ")).toDF("word")

// Count occurrences of each word using DataFrame operations
val wordCountsDF = wordsDF.groupBy("word").count()

// Show the results
wordCountsDF.show()

// Convert DataFrame to Dataset
import spark.implicits._

val wordCountsDS = wordCountsDF.as[(String, Long)]

// Calculate the total number of words
val totalWords = wordCountsDS.map(_._2).reduce(_ + _)
println(s"Total words: $totalWords")
```

This example demonstrates:
- Loading data into RDDs and transforming it into DataFrames
- Using DataFrame operations to perform aggregations
- Converting DataFrames to Datasets for type-safe operations

## Conclusion

Apache Spark provides powerful abstractions for big data processing through RDDs, DataFrames, and Datasets. Understanding these concepts and their operations is crucial for effectively leveraging Spark's capabilities.

By mastering these basics, you can efficiently process large datasets and build scalable applications. In the next lesson, we'll explore more advanced Spark features, including machine learning and graph processing capabilities.
