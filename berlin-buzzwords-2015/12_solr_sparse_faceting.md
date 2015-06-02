# Solr Sparse faceting

http://berlinbuzzwords.de/session/solr-sparse-faceting

Danish state library

 - 500Tb+ web resources from Danish Net Archive (crawling and archiving danish web pages)
 - Estimated 50Tb Solr index data when finished
 - 3 machines of 16 CPU cores, 256Gb Ram, 25 * 900 GB SSD

# Problem

 - All fine, but researchers wanted to start faceting.
 - Response times too slow, high cardinality (number of distinct values).
 - Have 200 million different urls, want to facet on them.
 - Need a counter for each different url: 200 million counts

## Pipeline

 - how faceting works: make counters for each value, run through results, add things up
 - works, but high fluctuation in time
 - graph of heap size over time; lots of full GC; causes fluc

## reduce GC

 - don't allocate new counters for each facet calculation; reuse old ones (clearing them first)
   - makes things a lot more stable
   - still some GC spikes
   - plot of quartiles of response times makes this improvement more obvious; less spread, lower median

## Improving the minimum

 - lower bound on time is 500
   - because we iterate through all the counters (and have 200 million of these) to set them to 0
 - introduce a tracker layer to make the counters not need to be initialised, map to the first one needed
   - Adds an extra step, but wins in most cases
     - Gets rid of lower bound; makes things fast up to 100K hits in search result

# Solr cloud

"Fine counting" algorithm

 - phase 1:
   - Now get counts from each shard
   - merger calculates top-X terms
 - phase 2:
   - term counts are requested from the shards that did not return them in phase 1

 - improves things, but have "Pit of Pain" - spike in response time for searches which return an "intermediate" number of hits
   - because: small numbers of hits won't return many facet values
   - large numbers of hits will tend to return the same values from each shard
   - but middle numbers of hits will return different values from each, so have to transfer lots of data between shards

## improvment

 - Looking up counts in another way, that I didn't quite follow

## Going further

 - would like to be able to facet on links (places that pages link to)
   - 600 million different values
 - too many different counts
   - Use an int for each count, currently
     - Know the maximum value for each count (because we have the term frequency for the term)
     - So can use PackedInts, using only required number of bits
     - reduces memory use to 75%
 - instead, separate into separate planes:
   - one plane for first bit of counter
   - second plane for next 2 bits
   - third for next 3 bits

 - problem was that it didn't work due to tracking
   - so introuced another "1.5" bit layer.
     - confusing, but apparently works

 - gets memory usage down to much less space: 30% for domain, 13% for links, 8% for urls
   - there is an overhead for this for large result sets, but actually possible to run, so better

# Concurency

 - Ouch.  Worst case of 5-10 minutes
   - But can run with an 8Gb heap and 900 Gbshards

 - So, there are more tricks for getting better
   - No time to explain further
   - Nothing's impossible: "Try this at home!"

# Questions

 - why are there occasional outliers
   - garbage collection

 - will this be merged into Solr?
   - would like to, but this is complex, and getting it merged is a complex problem.
     - Yonik has done work to make the facetting API nicer, so could ideally be added one at a time.
