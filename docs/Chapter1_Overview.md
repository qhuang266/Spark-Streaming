# Chapater 1 Introdution
## Overview
First of all, let's talk about what is spark streaming. What is the difference between spark streaming programing and basic spark programming.

As the name suggests, spark streaming is for streaming job, e.g. you read continuous data from a Kafka topic, and manipulate them almost realtime, thus data sending and data processing can be doing in the same time. We don't need to wait until all data are sent, and then process all of them.

Let's compare two example of spark streaming and common spark job.


```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

// normal spark application
object CommonApp {
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
(1) Entry of program. Normal spark application use `SparkContext` as program entry, and streaming job use `StreamingContext` as entry. Although the core parts of computation of them are both based on RDD. But the job schedule is quite different. So the entries are different.
(2) The data source. You can use file as normal application's data source, but streaming job usually need a continuous data source, like kafka, TCP socket.
(3) Starting. When you write logical code in normal spark application, like transformation and action, it will build a DAG. Once it meet a action, it will execute. But in Spark streaming, when you write the logical part, it does not start, until you start the `SparkContext`. So you can see `ssc.start()` and `ssc.awaitTermination()`.

## Basic Idea
The basic idea of spark streaming is splitting input data streaming into small batches, and each time one job process one batch.

![](https://raw.githubusercontent.com/qhuang266/Spark-Streaming/master/docs/img/streaming-flow.png)
ref: [Spark Streaming](http://spark.apache.org/docs/latest/streaming-programming-guide.html)

If we want to split the streaming, of course we need to set a parameter to do the splitting, and store the meta information of each batch job. So basically spark streaming contain following modules:

* Job scheduler
* Receiver
* Discretized Stream Operation

## Architecture
![]()

* Main entry point: `StreamingContext`

`StreamingContext` is use to create `Dstream`, and the streaming computation is started or stoped by `StreamingContext.start()` and `StreamingContext.stop()`.

You can create `StreamingContext` from a `SparkConf` or from an existing `SparkContext` or restore from a checkpoint. And when creating `StreamingContext`, also need to set the batch duration information, this is used to control how much data one job will process. 

* 

## Problem
1. spark.master should be set as local[n], n > 1 in local mode if you have receivers" +   " to get data, otherwise Spark jobs will not get resources to process the received data."


