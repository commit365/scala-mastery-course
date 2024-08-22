# Advanced Spark Programming

In this lesson, we will explore advanced features of Apache Spark, including custom partitioners, optimizations, the Catalyst optimizer, and machine learning with Spark MLlib and Spark Streaming.

## Custom Partitioner

By default, Spark uses a hash partitioner to distribute data across partitions. However, you can create a custom partitioner to control how data is distributed, which can improve performance for certain workloads.

### Creating a Custom Partitioner

To create a custom partitioner, extend the `Partitioner` class and override the `numPartitions` and `getPartition` methods.

```scala
import org.apache.spark.Partitioner

class CustomPartitioner(partitions: Int) extends Partitioner {
  require(partitions >= 0, s"Number of partitions ($partitions) cannot be negative.")

  def numPartitions: Int = partitions

  def getPartition(key: Any): Int = {
    key match {
      case k: String => k.length % numPartitions // Partition by string length
      case _ => 0 // Default partition for unknown types
    }
  }
}

// Example usage
val rdd = sc.parallelize(Seq(("apple", 1), ("banana", 2), ("kiwi", 3), ("pear", 4)))
val partitionedRDD = rdd.partitionBy(new CustomPartitioner(3))
```

### Benefits of Custom Partitioners

Using custom partitioners can help:

- Reduce data shuffling during joins.
- Optimize data locality for specific workloads.
- Improve performance by grouping related data together.

## Optimizations and the Catalyst Optimizer

Spark SQL uses the Catalyst optimizer to optimize query execution plans. It applies various optimization techniques to improve performance.

### Catalyst Optimizer Features

1. **Logical Plan Optimization**: Transforms the logical query plan into an optimized logical plan by applying rules like predicate pushdown and constant folding.
  
2. **Physical Plan Generation**: Generates one or more physical execution plans from the optimized logical plan and selects the most efficient one.

3. **Cost-Based Optimization**: Uses statistics about data distribution to choose the best execution plan.

### Example of Catalyst Optimization

When you run a DataFrame operation, Spark uses the Catalyst optimizer behind the scenes to optimize the execution plan.

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Catalyst Optimization Example")
  .master("local[*]")
  .getOrCreate()

import spark.implicits._

// Create a DataFrame
val df = Seq(
  ("Alice", 30),
  ("Bob", 25),
  ("Charlie", 35)
).toDF("name", "age")

// Perform a query
val resultDF = df.filter($"age" > 30).select("name")

// Show the execution plan
resultDF.explain(true) // Displays the physical and logical plans
```

## Machine Learning with Spark MLlib

Spark MLlib is a scalable machine learning library that provides various algorithms and utilities for building machine learning models.

### Basic Example of MLlib

Here’s a simple example of using MLlib to perform linear regression.

```scala
import org.apache.spark.ml.regression.LinearRegression
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Linear Regression Example")
  .master("local[*]")
  .getOrCreate()

// Load training data
val trainingData = spark.read.format("libsvm")
  .load("data/mllib/sample_linear_regression_data.txt")

// Create a Linear Regression model
val lr = new LinearRegression()
  .setMaxIter(10)
  .setRegParam(0.3)
  .setElasticNetParam(0.8)

// Fit the model
val lrModel = lr.fit(trainingData)

// Print the coefficients and intercept
println(s"Coefficients: ${lrModel.coefficients}")
println(s"Intercept: ${lrModel.intercept}")
```

### Evaluating the Model

You can evaluate the performance of your model using various metrics.

```scala
import org.apache.spark.ml.evaluation.RegressionEvaluator

val predictions = lrModel.transform(trainingData)

val evaluator = new RegressionEvaluator()
  .setLabelCol("label")
  .setPredictionCol("prediction")
  .setMetricName("rmse")

val rmse = evaluator.evaluate(predictions)
println(s"Root Mean Squared Error (RMSE): $rmse")
```

## Spark Streaming

Spark Streaming enables processing real-time data streams. It allows you to build applications that can process data in micro-batches.

### Basic Example of Spark Streaming

Here’s a simple example of using Spark Streaming to process text data from a socket.

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

val conf = new SparkConf().setMaster("local[*]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))

// Create a DStream that connects to a socket
val lines = ssc.socketTextStream("localhost", 9999)

// Split the lines into words
val words = lines.flatMap(_.split(" "))

// Count the words
val wordCounts = words.map(word => (word, 1)).reduceByKey(_ + _)

// Print the counts
wordCounts.print()

// Start the streaming context
ssc.start()
ssc.awaitTermination()
```

### Running the Streaming Application

To run the Spark Streaming application, you need to set up a socket server that sends data. You can use `netcat` for this purpose:

```bash
nc -lk 9999
```

You can then type words into the terminal, and the Spark Streaming application will process and count the words in real-time.

## Practice Exercises

1. Create a custom partitioner that partitions data based on the first letter of a string key.

2. Implement a simple Spark application that reads a CSV file, performs data cleaning, and writes the cleaned data to a new file.

3. Use Spark MLlib to implement a classification model (e.g., logistic regression) and evaluate its performance.

4. Set up a Spark Streaming application that reads data from a Kafka topic and processes it.

## Advanced Example: Combining Spark Features

Let’s create a more complex example that combines RDDs, DataFrames, and machine learning:

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.ml.feature.{VectorAssembler, StringIndexer}
import org.apache.spark.ml.classification.LogisticRegression

val spark = SparkSession.builder()
  .appName("Advanced Spark Example")
  .master("local[*]")
  .getOrCreate()

// Load data into a DataFrame
val data = spark.read.option("header", "true").csv("data/iris.csv")

// Convert the label to an index
val indexer = new StringIndexer()
  .setInputCol("species")
  .setOutputCol("label")

val indexedData = indexer.fit(data).transform(data)

// Assemble feature columns into a feature vector
val assembler = new VectorAssembler()
  .setInputCols(Array("sepal_length", "sepal_width", "petal_length", "petal_width"))
  .setOutputCol("features")

val finalData = assembler.transform(indexedData)

// Split the data into training and test sets
val Array(trainingData, testData) = finalData.randomSplit(Array(0.8, 0.2))

// Create and train a Logistic Regression model
val lr = new LogisticRegression()
val model = lr.fit(trainingData)

// Make predictions on the test set
val predictions = model.transform(testData)

// Show the predictions
predictions.select("features", "label", "prediction").show()

// Stop the Spark session
spark.stop()
```

This example demonstrates:
- Loading data from a CSV file into a DataFrame.
- Using `StringIndexer` to convert categorical labels into numerical indices.
- Assembling feature columns into a single feature vector.
- Training a Logistic Regression model and making predictions.

## Conclusion

Advanced Spark programming involves leveraging custom partitioners, understanding optimizations through the Catalyst optimizer, and utilizing Spark MLlib for machine learning tasks. Additionally, Spark Streaming allows you to process real-time data effectively.

By mastering these advanced features, you can build powerful data processing pipelines and machine learning applications that can handle large-scale data efficiently. In the next lesson, we will explore testing in Scala, focusing on techniques and tools for writing effective tests for your applications.