# Computing recommendations at extreme scale with Apache Flink

http://berlinbuzzwords.de/session/computing-recommendations-extreme-scale-apache-flink

 - Recommender systems omnipresent (selling stuff)

## Basics

 - overview of collaborative filtering
   - recommending items based on users with similar preferences
   - latent factor models capture underlying characteristics of items and user preferences
   - matrix factorization; calculate low rank approximation which gives the latent factors
     - machine learning to find this approximation; needs a loss function, use RMS error

 - but, we have two variables, so this makes it hard to minimze loss function
   - alternating least squares
     - fix one variable gives quadratic problem
     - minimise, then swap
     - repeat

## Apache Flink

https://flink.apache.org/

 - for really large scale, need distributed min-squares.
 - Flink does streaming dataflow algorithms
   - gave overview of APIs (Dataset, DataStream)

 - Why is Flink good for ALS:

   - API: abstraction for distributed data.  Define computations as a sequence
     of lazily evaluated transformations; essentially, define the dataflow.
     example in Scala:

         case class Word(word: String, frequency: Int)
         val lines: DataSet[String] = env.readTextFile(...)
         lines.flatMap(line => line.split(" ").map(word => Word(word, 1)).groupBy("word").sum("frequency").print()

   - Pipelined stream processor:
     - avoids materialization of intermediate results

   - Iterate in the Dataflow:
     - Can specify iteration explicitly; Flink can then optimise this - eg, by caching reused information

   - Memory management:
     - Objects in JVM are put on the heap; when it gets full, they need to be shunted to disk.
       - Flink avoids this by working mainly on serialised data.
         - Data is serialised, put into buffers in managed memory, when full some buffers are put on disk.  Easy to get back, because serialised.

 - Naive implementation of ALS
   - Joining item vectors; needs to send redundant information over the network

 - Better; do a "blocked" ALS implementation
   - Create user and item rating blocks; duplicate rating matrix; once partitioned by user, once by item
   - Cache them on worker nodes
   - Send all vectors needed by user rating block bundled

 - 40 node GCE cluster (highmem-8 machines)
   - 10 iterations for a rating matrix with 28 billion non-zero entries, took 14 hours before, with optimisation 5.5 hours

 - Don't want to implement yourself:
   - FlinkML contains implementation of blocked ALS
     - http://flink.apache.org/features.html#machine-learning-library
   - and many other tasks - clustering, regression, classification
 - scikit-learn like pipeline support

# Questions

 - does library support implicit feedback
   - you need to give it a rating matrix, so that's up to you
 - what does the "predict" function give you
   - dataset of userid, itemid, predicted rating value.

 - What is Flink more suitable for than Spark / Storm
   - Flink and Spark share concepts from API point of view
   - Underlying technology of both systems is similar
   - Flink will do "Real stream processing" - window functions, etc, 
   - From machine learning point of view, Spark's library is more mature, but Flink is catching up quickly, and are adding new features every few weeks.
   - Difference is mainly the runtime; how it can deal with different workloads
   - Streaming machine learning library will probably be a distinguishing feature.
