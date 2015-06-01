# Adrien Grand - from Elastic

Amusing algorithms and data-structures that power Lucene and Elasticsearch

 - conjunctions
 - regexp queries
 - number doc values compression
 - cardinality aggregation

## Conjunctions

 - first explained inverted index (structure, and operations: "next" and "advance")
   - advance on term dictionary uses a tiny in-memory terms index.
 - described implementing conjunctions by sorting posting lists by length, and then leap-frogging

## Regexp queries

 - Challenge: find matching terms and merge posting lists
 - Naive: iterate over terms, check each against regexp => very slow
 - Instead, lucene has logic to intersect the term dictionary with an automaton (from a compiled regexp)
   - uses same leap-frogging logic, to iterate through the intersection of the term dictionary and the automaton
 - Can also do this for fuzzy queries; since they can be represented as automatons.

## Numeric document value compression

 - document values; lists of values for documents; column-oriented field storage.
 - compressing:
   - default strategy: delta encoding + bit packing
     - compute min and max; tells us how many bits each value needs
     - encode the minimum value
     - encode all the deltas from this minimum value
   - if there are less than 256 unique values:
     - dedup and sort values
     - encode the table index for every doc, using as few bits as needed (depends on number of unique values)
   - for timestamps without full precision
     - compute minimum value
     - compute deltas
     - compute GCD of the deltas
     - encode min value and GCD
     - encode values divided by GCD

## Cardinality aggregation
"bit-pattern observable magic"

 - For computing things like `COUNT(DISTINCT(*))`
 - Counting large numbers (eg, 10^8) of unique values would require more than 1Gb of RAM if using a set to put them in.
   - worse in distributed environment, because whole set of values from each shard would need to be transferred to central place
 - Using `HyperLogLog++`  (http://stefanheule.com/papers/edbt13-hyperloglog.pdf from Google)
   - Approximates cardinality using a few Kb of memory; particularly good for distributed case
   - Hash each value. Count number of zero bytes at end of the hash
   - As we iterate, track the maximum seen number of zeros
   - Sensitive to occasional "bad values" which have lots of zeros
     - So use multiple diffenert hashes, instead; keep multiple counters
     - Use harmonic mean times number of register to get an estimate => gives less weight to outliers.
     - Needs very little memory; registers are small
     - Unions of the hashes are lossless.
   - HyperLogLog++ is an improvement to correct bias of this
 - t-digest is another algorithm similar, but for computing percentiles
