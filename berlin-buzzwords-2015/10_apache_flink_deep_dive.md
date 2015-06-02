# Apache Flink deep dive - Stephan Ewen - @StephanEwen

http://berlinbuzzwords.de/session/apache-flink-deep-dive

 - Used to be called stratosphere
 - Now company in Berlin @dataArtisans: http://data-artisans.com/
 - Stratosphere adopted by Apache in April 2014
 - Lots of community adoption in last year

# What is Flink

 - streaming dataflow engine
 - bit like a storm topology, but nodes can be much richer, more complex data flow
 - can be presented either as a streaming or a batch processor (batch as a special case of streaming)
 - batch API is "DataSet"; streaming is "DataStream"
 - Can run locally, remotely, using Yarn, Apache Tez (scheduler for things like hive)

# Usecases

 - Streaming topologies; optimising flow, window functions
 - long batch pipelines
 - ML at scale
 - graph analysis

 - Supporting these natively
   - eg, avoid iterating by running lots of jobs and passing the output from one to the input of the next
   - eg, avoid doing streaming by breaking it up into lots of small batch jobs

 - Engine:
   - Execute everything as streams
   - Allow some iterative (cyclic) dataflows; allows a way of feeding data back, so system can take advantage of this, fewer barriers
   - Handle mutable state; while preserving fault tolerance
   - Operate on managed memory; avoid garbage collection where possible

 - Use case: data streaming analysis

   - Would expect the infrastructure to be something like:
     - gathering data (logs, sensors, transation logs)
     - broker (eg kafka), making reliable, consumable streams
     - analysis (flink)

   - Shuffle operations can be streamed; there's some buffering (buffers get flushed if full for too long, to preserve low-latency)

   - streaming API
     - lets you do windowing (length, and separation; overlapping usually)
     - can do things like groupBy(word), sum(frequency) within windows
     - dataset API looks like datastream API without windowing (for the example given)

   - Fault tolerance
     - push checkpoint barriers through the dataflow; Chandy-Lamport Algorithm for consistent async distributed snapshots (http://en.wikipedia.org/wiki/Snapshot_algorithm)
     - barriers chop stream into checkpoints, flush stream and push elements before it. Then move to end; one all barriers reach end, that part is persistent.

# Long batch pipelines

 - special case of streaming; finite stream
 - one window spanning everything
 - for batch programs, some operators block (eg, sorts, hash tables)
   - will only bring processors up when they're first getting data, and tear them down at the end
 - but even things like sort can be started before all the data's in (just not finished).
 - managed memory is really important to avoid things running on same node killing each other by using all the memory
   - lots of work in serialised form
   - degrades gracefully as runs out of memory, by writing blocks to disk.

 - caching internal state, etc, allows iterative algorithms to do less work for latest steps (assuming consistency)

# Roadmap

 - Lots for 2015
 - Conference FlinkForward in Oct 2015

# Questions

 No time - Lunch!
