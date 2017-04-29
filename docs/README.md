# Spark Streaming
Spark version: master branch (2016/11/28)

**Chapter 0** will teach you to set up the environment of reading spark source code, **Chapter 1** uses a real streaming job as a example, and explains what happen inside the job. **Chapter 2 ~ 5** will dig into all parts of source code, and explain why they are designed like that. **Chapter 6** will update some useful resources.

## Chapter 0 - IDE Setup for reading source code

### Environment 
* Intellij IDEA 15.0.4
* Spark source code (master branch)

### Steps
* `git clone git://github.com/apache/spark.git
`
* `./dev/make-distribution.sh --name spark-2.0.2 --tgz --mvn mvn -Phadoop-2.7 -Pyarn`
* open Intellij IDEA, `open` -> `./pom.xml`, it all detect all modules in spark. And you can choose needed profiled, it will resolve the dependencies.  This may cost a long time.

## Chapter 1 - Streaming Job Example, Module Introduction

### Overview
First of all, let's talk about what is spark streaming. What is the difference between spark streaming programing and batch spark programming.

As the name suggests, spark streaming is for streaming job, e.g. you read continuous data from a Kafka topic, and manipulate them almost realtime, in this way, sending data and processing data can be doing in the same time, we don't need to wait all data are sent, and then process all of them.

Let's compare two example of spark streaming and batch spark job.


```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

// batch spark application
object BatchApp {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("Basic Spark Application")
    val sc = new SparkContext(conf)
    val logFile = "/cluster/user/foo/1"
    val logData = sc.textFile(logFile, 2)
    val result = logData.map { x => x + 1 }
    			.filter { x => x < 10 }
    			.collect()
    result.foreach{ x => println(x) }
    sc.stop()
  }
}
```

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._ 

// streaming applicaiton
object StreamingApp {
	val conf = new SparkConf().setAppName("Streaming Application")
	val ssc = new StreamingContext(conf, Seconds(1))
	val lines = ssc.socketTextStream("localhost", 9999)
	val result = lines.map { x => x + 1 }
							.filter { x => x < 10 }
							.print()
	ssc.start()
	ssc.awaitTermination()
}
```

You can see there are several difference.
(1) Entry of program. Batch spark application use `SparkContext` as program entry, and streaming job use `StreamingContext` as entry. Although the main functionality of them are both based on RDD, but the job schedule is quite different, streaming job need to split the input into several small batch, and process each batch as a batch spark job. So `StreamingContext` can be seen as `SparkContext` + (some streaming features). And you can see when construct a `StreamingContext`, you need to provide not only `SparkConf`(`SparkContext` only need it) but also a duration which is used to schedule splitting the input streaming data.

(2) The data source. You can use file as batch application's data source, and streaming job usually need a continuous data source, like kafka, TCP socket. Besides, it can be also used to monitor HDFS, and deal with the new moved files.

(3) Starting. When you implement the workflow in batch spark application, like transformation and action, it will build a DAG. Once it meet a action, it will execute. But in Spark streaming, when you write the workflow code, it does not start to process the data, but until you start the `StreamingContext`. So you can see `ssc.start()` and `ssc.awaitTermination()`.

### Basic Idea
The basic idea of spark streaming is splitting input data streaming into small batches (as following figure shows), and one job process one small batch each time. So the process job is just the same as general spark job which process rdd.

![](img/streaming-flow.png)
ref: [Spark Streaming](http://spark.apache.org/docs/latest/streaming-programming-guide.html)

Thus we need to have a function to do the splitting, and store the meta information of each batch job. So basically spark streaming contain some particular modules:

* Receiver (use to receive streaming data)
* Job scheduler (after receive data, schedule the job for each batch data)
* Receiver
* Discretized Stream Operation (wrap the streaming data as rdd)

### Architecture
![]()

* Main entry point: `StreamingContext`

`StreamingContext` is use to create `Dstream`, and the streaming computation is started or stoped by `StreamingContext.start()` and `StreamingContext.stop()`.

You can create `StreamingContext` from a `SparkConf` or from an existing `SparkContext` or restore from a checkpoint. And when creating `StreamingContext`, also need to set the batch duration information, this is used to control how much data one job will process. 

* 

### Problem
1. spark.master should be set as local[n], n > 1 in local mode if you have receivers" +   " to get data, otherwise Spark jobs will not get resources to process the received data."

## Chapter 2 - DStream, RDD
## Chapter 3 - Job Schedule
## Chapter 4 - Receiver
## Chapter 5 - Fault Tolerance
## Chapter 6 - Reference Resources
### Books

### Blogs

### Videos

### Projects


