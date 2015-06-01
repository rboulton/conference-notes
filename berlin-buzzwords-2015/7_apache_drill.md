# What, Why and How: Apache Drill 1.0 - Ted Dunning

http://berlinbuzzwords.de/session/what-and-why-and-how-apache-drill-10

 - SQL for everything

## SQL like Mom made

 - all the standard ANSI SQL syntax
 - Supports standard PI tools, ODBC, standard Hadoop tools
 - Also extends standard
   - eg, Path based queries (eg, `select * from /my/logs`)
   - Modern data types (Any, Map, Array).
     - Any time is key to "latent typing" - runtime type checking
   - Complex Functions and operators (`FLATTEN`, `kvgen`, `convert_from`, `convert_to`, `repeated_count`, etc)

## Deployment is easy

 - Single daemon
 - With or without DFS
 - JSON everything

 - "Drillbits"

## Data Model

 - flexible: data model is JSON (representation isn't)
 - Query can be planned on any file, anywhere
 - Drill will operate at high speed with no metadata stored, but will use data when it gets it.
 - Even faster with more regular data.

## Example

 - Business dataset (from Yelp)
   - JSON files, no schema
 - Install (untar)
 - Launch (one command)
 - Run query directly:

     SELECT state, city, count(*) AS businesses
     FROM dfs.yelp.`business.json`
     GROUP BY state, city
     ORDER BY businesses

## Features

 - Supports views
   - logical views
     - can do things like online decryption, or line-by-line masking / filtering
   - also materialised views

 - Flatten -> queries on contents of arrays, as if they were already data which had been flattened
   - can be used in recursive queries

 - In NoSQL; data often goes into column names; the domain
   - eg, lists of checking counts: column name is range, value is a count.
   - Drill's `KVGEN` converts a map with lots of columns into a key-value pairs.
   - So then you can do queries on these.

 - Use the data which comes to you as it comes to you.

## Security

 - granular access control
   - can make views based on permissions
     - virtual copies of the data; based on filtering.
       - can have a chain of views, made by different people.
       - produce security which is based on filesystem permissions, then add more bits

## Goals

 - go fast when you don't know anything
 - go faster when you do

## Example - DAG engine
 - build a three-level DAG
  - fragments get parallelized
  - Uses Calcite query optimiser, and special rules which understand which bits can be parallel.
  - Data streams in in row batches
  - Supports partition pruning; lets bits which aren't relevant to a particular
    partition get thrown away; eg, if a column is renamed and query has support
    for handling the rename, only the branch required for each partition will
    actually be executed.

# Representation

 - In memory column representation
   - allows random access
   - Cache happiness
   - Does runtime compilation based on data types
     - if doesn't know, uses ANY type

 - Drill does vectorization
   - SIMD instructions

# Query explanation:
   - very colourful
     - produces "swim-lane" visualisation for the query, together with query tree
     - can see exactly which parts are parallizing well.

# Scale

 - Tested up to 150 nodes, target is to test 1000 nodes soon.
   - Drill adapts data transfer size, buffers, muxing, etc based on CPU.

# Questions

 - If you have Hive, do you need to have the Hive metadata
   - Drill can use it
     - Grevious performance hit from accessing hive API, though.

 - Have large stores based on deep Avro structures.  Can this be translated?
   - Yes; Avro is a nice data structure; doesn't columnise well, but the schema information can be used
     - nesting depth is immaterial
     - can flatten on the fly if wanted, too

 - Can drill create materialized views magically?
   - No, but can make them explicitly


- Drill is an open community.  Weekly hangout, etc.
