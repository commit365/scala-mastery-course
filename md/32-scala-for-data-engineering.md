# Scala for Data Engineering

Scala, combined with Apache Spark, is a powerful tool for data engineering tasks, particularly in building ETL (Extract, Transform, Load) processes. This lesson covers how to implement ETL processes using Scala and Spark, as well as data validation and schema evolution techniques.

## ETL Processes with Scala and Spark

ETL processes are essential for moving and transforming data between systems. Apache Spark provides a distributed computing framework that can efficiently handle large datasets.

### Setting Up Spark with Scala

1. **Create a new sbt project**:

   Create a new directory for your project and navigate into it:

   ```bash
   mkdir my-spark-etl
   cd my-spark-etl
   ```

2. **Create a `build.sbt` file**:

   Add the following content to your `build.sbt` file:

   ```sbt
   name := "MySparkETL"

   version := "0.1.0"

   scalaVersion := "2.13.6"

   libraryDependencies += "org.apache.spark" %% "spark-core" % "3.1.2"
   libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.1.2"
   ```

3. **Create an ETL Application**:

   Create a file named `ETLApp.scala` in `src/main/scala` with the following content:

```scala
import org.apache.spark.sql.{SparkSession, DataFrame}
import org.apache.spark.sql.functions._

object ETLApp extends App {
  // Initialize Spark session
  val spark: SparkSession = SparkSession.builder()
    .appName("ETL Example")
    .master("local[*]")
    .getOrCreate()

  // Step 1: Extract data
  val rawData: DataFrame = spark.read.option("header", "true").csv("data/input.csv")

  // Step 2: Transform data
  val transformedData: DataFrame = rawData
    .withColumn("age", col("age").cast("int")) // Cast age to integer
    .filter(col("age").isNotNull) // Filter out rows with null age

  // Step 3: Load data
  transformedData.write.mode("overwrite").parquet("data/output.parquet")

  // Stop the Spark session
  spark.stop()
}
```

### Running the ETL Application

To run your ETL application, use sbt:

```bash
sbt run
```

Make sure you have an input CSV file located at `data/input.csv` with appropriate data to test the ETL process.

## Data Validation

Data validation is crucial in ETL processes to ensure that the data meets specific criteria before further processing. You can implement data validation using Spark's DataFrame API.

### Example of Data Validation

You can validate data by checking for null values, data types, and other constraints:

```scala
val validatedData: DataFrame = transformedData
  .filter(col("name").isNotNull) // Ensure name is not null
  .filter(col("age") > 0) // Ensure age is positive
```

### Handling Invalid Data

You can handle invalid data by either filtering it out or logging it for further analysis:

```scala
val (validData, invalidData) = transformedData
  .filter(col("age").isNotNull)
  .partitionBy(col("age").gt(0)) // Partition valid and invalid data

// Write invalid data to a separate location for review
invalidData.write.mode("overwrite").csv("data/invalid_data.csv")
```

## Schema Evolution

Schema evolution refers to the ability to change the schema of your data over time without breaking existing processes. This is particularly important in data engineering as data sources and requirements change.

### Handling Schema Evolution in Spark

Spark supports schema evolution when writing data in formats like Parquet and Delta Lake. You can specify options to allow schema merging.

#### Example of Schema Evolution

1. **Write Data with Schema Evolution**:

```scala
transformedData.write
  .mode("append")
  .option("mergeSchema", "true") // Allow schema merging
  .parquet("data/output.parquet")
```

2. **Reading Data with Evolved Schema**:

When reading data, Spark will automatically handle the evolved schema:

```scala
val newData: DataFrame = spark.read.parquet("data/output.parquet")
```

### Using Delta Lake for Schema Evolution

Delta Lake is an open-source storage layer that brings ACID transactions to Apache Spark and big data workloads. It provides better support for schema evolution.

1. **Add Delta Lake Dependency** to your `build.sbt`:

```sbt
libraryDependencies += "io.delta" %% "delta-core" % "1.0.0"
```

2. **Using Delta Lake**:

```scala
import io.delta.tables._

val deltaTable = DeltaTable.forPath(spark, "data/output.delta")

// Update the schema
deltaTable.toDF
  .withColumn("newColumn", lit("defaultValue"))
  .write
  .format("delta")
  .mode("overwrite")
  .save("data/output.delta")
```

## Practice Exercises

1. Implement an ETL process that reads data from a JSON file, transforms it, and writes it to a Parquet file.

2. Create a data validation step that checks for duplicate entries in the dataset.

3. Use Delta Lake to handle schema evolution in a sample dataset, demonstrating adding new columns.

4. Write a Spark application that reads data from a database, performs transformations, and loads it into another database.

## Advanced Example: Building a Complete ETL Pipeline

Let’s create a more complex example that combines all the concepts discussed:

```scala
import org.apache.spark.sql.{SparkSession, DataFrame}
import org.apache.spark.sql.functions._
import io.delta.tables._

object CompleteETLPipeline extends App {
  // Initialize Spark session
  val spark: SparkSession = SparkSession.builder()
    .appName("Complete ETL Pipeline")
    .master("local[*]")
    .getOrCreate()

  // Step 1: Extract data
  val rawData: DataFrame = spark.read.option("header", "true").csv("data/input.csv")

  // Step 2: Validate and transform data
  val transformedData: DataFrame = rawData
    .filter(col("name").isNotNull && col("age").isNotNull)
    .withColumn("age", col("age").cast("int"))
    .filter(col("age") > 0)

  // Step 3: Load data into Delta Lake with schema evolution
  transformedData.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .save("data/delta_output")

  // Step 4: Read and update the Delta table
  val deltaTable = DeltaTable.forPath(spark, "data/delta_output")

  // Add a new column to the existing schema
  deltaTable.toDF
    .withColumn("newColumn", lit("defaultValue"))
    .write
    .format("delta")
    .mode("overwrite")
    .save("data/delta_output")

  // Stop the Spark session
  spark.stop()
}
```

In this example:
- We extract data from a CSV file, validate it, and transform it.
- We load the data into a Delta Lake table, allowing for schema evolution.
- We demonstrate how to update the schema by adding a new column.

## Conclusion

Scala, in conjunction with Apache Spark, provides powerful tools for data engineering tasks, particularly for building ETL processes. By understanding how to implement data validation, handle schema evolution, and leverage Delta Lake, you can create robust and scalable data pipelines.

Mastering these concepts will enhance your ability to work with large datasets and build effective data engineering solutions. In the next lesson, we will explore additional advanced topics in Scala, focusing on best practices and design patterns for building effective software solutions.