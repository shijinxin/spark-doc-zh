---
layout: global
title: Structured Streaming + Kafka Integration Guide (Kafka broker version 0.10.0 or higher)
---

为Kafka 0.10提供结构化的流集成，以读取数据，并将数据写入Kafka。

## 链接
对于使用SBT / Maven项目定义的Scala / Java应用程序，将您的应用程序链接到以下工件:

    groupId = org.apache.spark
    artifactId = spark-sql-kafka-0-10_{{site.SCALA_BINARY_VERSION}}
    version = {{site.SPARK_VERSION_SHORT}}

对于Python应用程序，您需要在部署应用程序时添加上述库及其依赖项。请参阅下面的[部署](#deploying)小节。

## 从卡夫卡读取数据

### 创建一个用于流查询的Kafka源

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}

// Subscribe to 1 topic
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

// Subscribe to multiple topics
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1,topic2")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

// Subscribe to a pattern
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribePattern", "topic.*")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

{% endhighlight %}
</div>
<div data-lang="java" markdown="1">
{% highlight java %}

// Subscribe to 1 topic
DataFrame<Row> df = spark
  .readStream()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

// Subscribe to multiple topics
DataFrame<Row> df = spark
  .readStream()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1,topic2")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

// Subscribe to a pattern
DataFrame<Row> df = spark
  .readStream()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribePattern", "topic.*")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

{% endhighlight %}
</div>
<div data-lang="python" markdown="1">
{% highlight python %}

# Subscribe to 1 topic
df = spark \
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribe", "topic1") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

# Subscribe to multiple topics
df = spark \
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribe", "topic1,topic2") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

# Subscribe to a pattern
df = spark \
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribePattern", "topic.*") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

{% endhighlight %}
</div>
</div>

### 为批量查询创建Kafka源：
如果您有一个更适合于批处理的用例，您可以创建一个Dataset/DataFrame来定义范围的偏移量。

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}

// Subscribe to 1 topic defaults to the earliest and latest offsets
val df = spark
  .read
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

// Subscribe to multiple topics, specifying explicit Kafka offsets
val df = spark
  .read
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1,topic2")
  .option("startingOffsets", """{"topic1":{"0":23,"1":-2},"topic2":{"0":-2}}""")
  .option("endingOffsets", """{"topic1":{"0":50,"1":-1},"topic2":{"0":-1}}""")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

// Subscribe to a pattern, at the earliest and latest offsets
val df = spark
  .read
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribePattern", "topic.*")
  .option("startingOffsets", "earliest")
  .option("endingOffsets", "latest")
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .as[(String, String)]

{% endhighlight %}
</div>
<div data-lang="java" markdown="1">
{% highlight java %}

// Subscribe to 1 topic defaults to the earliest and latest offsets
DataFrame<Row> df = spark
  .read()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load();
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)");

// Subscribe to multiple topics, specifying explicit Kafka offsets
DataFrame<Row> df = spark
  .read()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1,topic2")
  .option("startingOffsets", "{\"topic1\":{\"0\":23,\"1\":-2},\"topic2\":{\"0\":-2}}")
  .option("endingOffsets", "{\"topic1\":{\"0\":50,\"1\":-1},\"topic2\":{\"0\":-1}}")
  .load();
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)");

// Subscribe to a pattern, at the earliest and latest offsets
DataFrame<Row> df = spark
  .read()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribePattern", "topic.*")
  .option("startingOffsets", "earliest")
  .option("endingOffsets", "latest")
  .load();
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)");

{% endhighlight %}
</div>
<div data-lang="python" markdown="1">
{% highlight python %}

# Subscribe to 1 topic defaults to the earliest and latest offsets
df = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribe", "topic1") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

# Subscribe to multiple topics, specifying explicit Kafka offsets
df = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribe", "topic1,topic2") \
  .option("startingOffsets", """{"topic1":{"0":23,"1":-2},"topic2":{"0":-2}}""") \
  .option("endingOffsets", """{"topic1":{"0":50,"1":-1},"topic2":{"0":-1}}""") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

# Subscribe to a pattern, at the earliest and latest offsets
df = spark \
  .read \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribePattern", "topic.*") \
  .option("startingOffsets", "earliest") \
  .option("endingOffsets", "latest") \
  .load()
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
{% endhighlight %}
</div>
</div>

源中的每一行都有以下模式:
<table class="table">
<tr><th>Column</th><th>Type</th></tr>
<tr>
  <td>key</td>
  <td>binary</td>
</tr>
<tr>
  <td>value</td>
  <td>binary</td>
</tr>
<tr>
  <td>topic</td>
  <td>string</td>
</tr>
<tr>
  <td>partition</td>
  <td>int</td>
</tr>
<tr>
  <td>offset</td>
  <td>long</td>
</tr>
<tr>
  <td>timestamp</td>
  <td>long</td>
</tr>
<tr>
  <td>timestampType</td>
  <td>int</td>
</tr>
</table>

对于批处理和流查询，必须为Kafka源设置以下选项。

<table class="table">
<tr><th>Option</th><th>value</th><th>meaning</th></tr>
<tr>
  <td>assign</td>
  <td>json string {"topicA":[0,1],"topicB":[2,4]}</td>
  <td>具体TopicPartitions消费。"assign", "subscribe" or "subscribePattern" 选项，只可以有一个指定给kafka 源。</td>
</tr>
<tr>
  <td>subscribe</td>
  <td>A comma-separated list of topics</td>
  <td>订阅的主题列表。"assign", "subscribe" or "subscribePattern" 选项，只可以有一一个指定给kafka 源。</td>
</tr>
<tr>
  <td>subscribePattern</td>
  <td>Java regex string</td>
  <td>subscribePattern	Java regex string	用于订阅主题的模式。"assign", "subscribe" or "subscribePattern" 选项，只可以有一一个指定给kafka 源。</td>
</tr>
<tr>
  <td>kafka.bootstrap.servers</td>
  <td>A comma-separated list of host:port</td>
  <td>Kafka的 "bootstrap.servers" 配置。</td>
</tr>
</table>

以下配置是可选的:

<table class="table">
<tr><th>Option</th><th>value</th><th>default</th><th>query type</th><th>meaning</th></tr>
<tr>
  <td>startingOffsets</td>
  <td>"earliest", "latest" (streaming only), or json string
  """ {"topicA":{"0":23,"1":-1},"topicB":{"0":-2}} """
  </td>
  <td>"latest" for streaming, "earliest" for batch</td>
  <td>streaming and batch</td>
  <td>当一个查询开始的时候, 或者从最早的偏移量："earliest",或者从最新的偏移量："latest",或JSON字符串指定为每个topicpartition起始偏移。在json中，-2作为偏移量可以用来表示最早的，-1到最新的。注意:对于批处理查询，不允许使用最新的查询(隐式或在json中使用-1)。对于流查询，这只适用于启动一个新查询时，并且恢复总是从查询的位置开始，在查询期间新发现的分区将会尽早开始。</td>
</tr>
<tr>
  <td>endingOffsets</td>
  <td>latest or json string
  {"topicA":{"0":23,"1":-1},"topicB":{"0":-1}}
  </td>
  <td>latest</td>
  <td>batch query</td>
  <td>当一个批处理查询结束时，或者从最新的偏移量："latest", 或者为每个topic分区指定一个结束偏移的json字符串。在json中，-1作为偏移量可以用于引用最新的，而-2(最早)是不允许的偏移量。</td>
</tr>
<tr>
  <td>failOnDataLoss</td>
  <td>true or false</td>
  <td>true</td>
  <td>streaming query</td>
  <td>当数据丢失的时候，这是一个失败的查询。(如：主题被删除，或偏移量超出范围。)这可能是一个错误的警报。当它不像你预期的那样工作时，你可以禁用它。如果由于数据丢失而不能从提供的偏移量中读取任何数据，批处理查询总是会失败。</td>
</tr>
<tr>
  <td>kafkaConsumer.pollTimeoutMs</td>
  <td>long</td>
  <td>512</td>
  <td>streaming and batch</td>
  <td>在执行器中从卡夫卡轮询执行数据，以毫秒为超时间隔单位。</td>
</tr>
<tr>
  <td>fetchOffset.numRetries</td>
  <td>int</td>
  <td>3</td>
  <td>streaming and batch</td>
  <td>放弃获取卡夫卡偏移值之前重试的次数。</td>
</tr>
<tr>
  <td>fetchOffset.retryIntervalMs</td>
  <td>long</td>
  <td>10</td>
  <td>streaming and batch</td>
  <td>在重新尝试取回Kafka偏移量之前等待毫秒值。</td>
</tr>
<tr>
  <td>maxOffsetsPerTrigger</td>
  <td>long</td>
  <td>none</td>
  <td>streaming and batch</td>
  <td>对每个触发器间隔处理的偏移量的最大数量的速率限制。偏移量的指定总数将按比例在不同卷的topic分区上进行分割。</td>
</tr>
</table>

## 写数据到卡夫卡

在这里，我们描述了对Apache Kafka编写流查询和批量查询的支持。注意，Apache Kafka只支持至少一次写语义。因此，当对卡夫卡进行流媒体查询或批量查询时，一些记录可能会被复制;例如，如果Kafka需要重试一个未被代理确认的消息，即使该代理收到并编写了消息记录。由于这些卡夫卡的写语义，结构化流不能防止这种重复发生。但是，如果写入查询是成功的，那么您可以假设查询输出至少编写一次。当阅读书面数据时，可能的解决方法是引入一个主键(唯一的)键，用于在读取时执行去复制。

Dataframe写入Kafka应该在模式中有以下列：
<table class="table">
<tr><th>Column</th><th>Type</th></tr>
<tr>
  <td>key (optional)</td>
  <td>string or binary</td>
</tr>
<tr>
  <td>value (required)</td>
  <td>string or binary</td>
</tr>
<tr>
  <td>topic (*optional)</td>
  <td>string</td>
</tr>
</table>
\* 如果没有指定“topic”配置选项，则需要主题列。<br>

value列是惟一需要的选项。如果没有指定键列，则会自动添加一个空值键列(参见Kafka语义，以处理如何处理null值的键值)。如果主题列存在，那么当将给定的行写入Kafka时，它的值就被用作主题，除非“topic”配置选项设置为。，“topic”配置选项覆盖主题栏。

对于批处理和流查询，必须为Kafka接收器设置以下选项。

<table class="table">
<tr><th>Option</th><th>value</th><th>meaning</th></tr>
<tr>
  <td>kafka.bootstrap.servers</td>
  <td>A comma-separated list of host:port</td>
  <td> Kafka的集群("bootstrap.servers")配置。</td>
</tr>
</table>

以下配置是可选的:

<table class="table">
<tr><th>Option</th><th>value</th><th>default</th><th>query type</th><th>meaning</th></tr>
<tr>
  <td>topic</td>
  <td>string</td>
  <td>none</td>
  <td>streaming and batch</td>
  <td>设置所有行写入卡夫卡的主题。此选项覆盖数据中可能存在的任何主题列。</td>
</tr>
</table>

### 为流查询创建Kafka接收器：

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}

// Write key-value data from a DataFrame to a specific Kafka topic specified in an option
val ds = df
  .selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .start()

// Write key-value data from a DataFrame to Kafka using a topic specified in the data
val ds = df
  .selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .start()

{% endhighlight %}
</div>
<div data-lang="java" markdown="1">
{% highlight java %}

// Write key-value data from a DataFrame to a specific Kafka topic specified in an option
StreamingQuery ds = df
  .selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .start()

// Write key-value data from a DataFrame to Kafka using a topic specified in the data
StreamingQuery ds = df
  .selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .start()

{% endhighlight %}
</div>
<div data-lang="python" markdown="1">
{% highlight python %}

# Write key-value data from a DataFrame to a specific Kafka topic specified in an option
ds = df \
  .selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") \
  .writeStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("topic", "topic1") \
  .start()

# Write key-value data from a DataFrame to Kafka using a topic specified in the data
ds = df \
  .selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)") \
  .writeStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .start()

{% endhighlight %}
</div>
</div>

### Writing the output of Batch Queries to Kafka

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}

// Write key-value data from a DataFrame to a specific Kafka topic specified in an option
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .write
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .save()

// Write key-value data from a DataFrame to Kafka using a topic specified in the data
df.selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")
  .write
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .save()

{% endhighlight %}
</div>
<div data-lang="java" markdown="1">
{% highlight java %}

// Write key-value data from a DataFrame to a specific Kafka topic specified in an option
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .write()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .save()

// Write key-value data from a DataFrame to Kafka using a topic specified in the data
df.selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)")
  .write()
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .save()

{% endhighlight %}
</div>
<div data-lang="python" markdown="1">
{% highlight python %}

# Write key-value data from a DataFrame to a specific Kafka topic specified in an option
df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)") \
  .write \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("topic", "topic1") \
  .save()

# Write key-value data from a DataFrame to Kafka using a topic specified in the data
df.selectExpr("topic", "CAST(key AS STRING)", "CAST(value AS STRING)") \
  .write \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .save()
  
{% endhighlight %}
</div>
</div>


## 卡夫卡的具体配置：

卡夫卡自己的配置可以通过`datastreamreader.option`中为`kafka.`的前缀设置,例如, 
`stream.option("kafka.bootstrap.servers", "host:port")`. 对于合理的卡夫卡参数，请参阅[kafka的消费者配置文档](http://kafka.apache.org/documentation.html#newconsumerconfigs)，
了解与读取数据相关的参数，以及[kafka生产者配置文档](http://kafka.apache.org/documentation/#producerconfigs)，以获得与写入数据相关的参数。

注意，以下卡夫卡的参数不能被设置，卡夫卡source或sink会抛出一个例外:

- **group.id**: Kafka源将自动为每个查询创建一个唯一的组id。
- **auto.offset.reset**:设置源选项`startingoffset`来指定从哪里开始。结构化流管理可以在内部消耗抵消，而不是依赖于kafka的消费者来完成。这将确保在动态订阅新主题/分区时不会遗漏任何数据。注意，`startingoffset`只适用于启动一个新的流查询时，并且恢复将总是从查询停止的地方开始。
- **key.deserializer**: 键总是被反序列化为字节数组和`ByteArrayDeserializer`。使用`DataFrame`操作显式地反序列化键。
- **value.deserializer**:值总是以字节数组和`ByteArrayDeserializer`进行反序列化。使用`DataFrame`操作显式地反序列化值。
- **key.serializer**: 键总是用`ByteArraySerializer`或`StringSerializer`序列化。使用`DataFrame`操作显式地将键序列化为字符串或字节数组。
- **value.serializer**: 值总是用`ByteArraySerializer`或`StringSerializer`序列化。使用`DataFrame oeprations`将值显式序列化到字符串或字节数组中。
- **enable.auto.commit**: 卡夫卡源不能提交任何偏移量。
- **interceptor.classes**: Kafka源总是将键和值读取为字节数组。使用`ConsumerInterceptor`是不安全的，因为它可能会破坏查询。
## 部署

与任何Spark应用程序一样，`spark-submit`用于启动应用程序。`spark-sql-kafka-0- 10_2.11`及其依赖关系可以直接添加到`spark-submit`中，例如，

    ./bin/spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_{{site.SCALA_BINARY_VERSION}}:{{site.SPARK_VERSION_SHORT}} ...
有关提交与外部依赖关系的应用程序的详细信息，请参阅应用[程序提交指南](submitting-applications.html)。
