# t-digest - Ted Dunning - MapR technologies

Good for anomaly detection: O'reilly books, practical approaches

https://github.com/tdunning/t-digest

## Why online algorithms

 - showed an inconsistent dashboard, due to counts being delayed by different things for different amounts.
   - destroys trust

## Why quantiles

 - 100M users, 1K sites touches each day
   - 99.9% latency shows us what bad things are happening with low frequency
   - Want this for lots of different sets of users

 - Or want to see what's going on on 1000 machines
   - lots of unique RPC calls, with <100ns overhead per measurement, <10Mb overhead per node
   - No loges except for exceptionally slow cases

## What about accuracy?

 - online algorithms only approximate

 - eg
   - for median; 50% +- 0.5% is probably fine
   - for 99.99%-ile, +- 0.5% makes no sense

## Algorithm

 - T-digest has property of variable accuracy, and constant relative accuracy

 - does this by using clusters, where cluster size is large in the middle, and has to be small at the ends.
   - originally used a normal distribution, but now using something which grows faster at the ends, relatively flat in the middle
     - get errors which are quadratic in the size of the cluster.

 - As long as scaled size of a centroid is 

 - Walk through sorted data, collect them together
   - have short butter for new points
    - sort when full, merge with existing centroids
    - all buffers fixed size, statically allocated
    - use an in-place merge
    - really fast

 - Available as an aggregator in elasticsearch
 - In Apache Drill, Mahout, in Maven.

## Upshot

 - streaming approximations are important
 - accurate quantiles are important

 - Could detect maximums this way - samles which are larger than 99%th percentile within a minute

# Questions

 - Is there a linkable library?
   - There's a C version, but have been waiting to release until there's a completely static, merging digest version.
   - also 

 - Do you need to define the things you want to measure in advance?
   - Important property is that you can merge the digests without losing the accuracy guarantees.
   - So can compute for pieces, and merge them together.
   - Also lets you parallelize this lots

 - Can this be used as a time-window function?
   - Typically you keep a different aggregate for each window.  eg, running count you have lots of overlapping sliding windows
   - No easy way to subtract from a t-digest, so can't just keep a fifo.
   - So instead you need to keep separate t-digests for the window sub-components

 - How does HDR histogram compare?
   - HDR has different assumptions (High dynamic range histogram).
     - assumes that 0 is a special point.  So we can define special buckets from 0..10^-9, then 10^-9..10^-8, etc.
     - can work nicely if distribution is like this, but doesn't work for more general distributions
     - t-digest has weaker assumptions, but is more general
     - it's a tradeoff - HDR is interesting too
