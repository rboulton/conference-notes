# Apache Lucene 5 - New Features and Improvements for Apache Solr and Elasticsearch - Uwe Schindler

http://berlinbuzzwords.de/session/apache-lucene-5-new-features-and-improvements-apache-solr-and-elasticsearch

# History

## Up to version 3.6

 - index format didn't change much: 10 years, a few new features from time to time
   - used the VINT format
   - was hard to add additional statistics for scoring to the index

 - Small changes to index format were often huge patches covering lots of files

## Lucene 4.0

 - added codec support and DocValues field
 - Added BM25, etc, relevancy models
 - FSAs / FST everywhere (term dictionaries, lots of queries, etc)
 - Lots of API changes, no backwards compatibility

## During Lucene 4.x series

 - APIs kept changing
 - Burden of maintaining old stuff: 
   - old index formats
   - especially support for Lucene 3.x indexes
   - complexity of maintaining non-codec ones and codec ones.

 - Disaster bugs
   - 4.2.0: deletes entire index if exception is thrown due to too many open files
   - 4.9.0: closing NRT reader after upgrading from 3.x => index corruption
   - 4.10.0: index version numbers caused CorruptIndexException

# Lucene 5

 - lots of new features
 - but not as many as you might expect for a major release

 - Feature 1: Removal of Lucene 3 index support
   - Can easily have old segments which haven't already been upgraded; upgrade happens on compaction of each index segment.
   - Index upgrader will ensure that all segments are in latest version
     - For elasticsearch, already implemented an automatic upgrader.
     - For solr, need to use upgrader from Lucene 5, or use "optimise" from a 4.10 version

 - Data safety:
   - Checksums in all index files
     - Validated on each merge
     - Can be validated during replication

 - Unique per-segment ID
   - better than just the filename
   - ensures that the reader definitely gets the right segment
     - prevents bugs caused by failures in replication

 - Java 7 support (already there since Lucene 4.8)
   - EOL of Java 6, but still bugs that affected Lucene
   - Java 8 released
   - Use Java 7 features for index safety
   - Huge speedup for dynamic instantiation of token Attributes, in Java 8
   - Have to use Java 7u55+ - "no serious bugs anymore"
     - G1 Garbage collector (http://www.oracle.com/technetwork/java/javase/tech/g1-intro-jsp-135488.html) still no good - might be okay in Java 8u40+

 - New index safety features
   - Using NIO.2 (JSR 203)
   - Full overhaul of Lucene IO APIs
     - Have a banned list of APIs on lucene: java.io.File* banned
   - No more segments.gen
   - Atomic renamed to publish commit
   - fsync() on directory metadata
     - Wasn't possible before Java 7, using undocumented API
     - Oracle broke it in Java 9 for a bit, but that's now reverted
     - And official APIs to do this are being added to Java 9

 - Exception handling cleaned up
   - NIO.2 APIs now throw useful exceptions; File.rename() / .delete() could previously do nothing at all
   - All file I/O is now channel based (or mmap)
     - if interrupted can throw ClosedByInterruptException, but this will break the IndexReader
       - So never use Future.cancel(true)!

 - Overhaul of Codec API
   - Pull APIs throughout
   - Norms are now simple docvalues

 - Index merging
   - On linux, detection if index is on SSD; use better merging settings (other operating systems assume SSDs)
   - Auto throttling
     - IO rates controlled based on indexing / merging rate; stalling under high load less likely.

 - Heap usage reduced
   - bitsets for query filters
   - better filter cache; tracking use frequency
   - merging uses less heap
   - classes implemtn "Accountable" interface, lets the heap usage be inspected

 - Add CustomAnalyzer

 - FieldCache is removed
   - Use DocValues instead

# Lucene 6

 - Filters being deprecated => become queries without scoring factors
   - removing lots of duplicate classes
   - compatibility by making Filter subclass Query

 - Splitting iterators into "cheap" and "expensive" parts
   - cheap part is the term matching
   - position lookup is replaced
   - span queries rewritten

# Questions

 - Retiring uninverted field?
   - this is something like fieldcache, but supports multiple fields
   - Still available in the "misc" module
