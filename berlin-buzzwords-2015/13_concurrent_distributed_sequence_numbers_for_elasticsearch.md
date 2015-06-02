# Designing Concurrent Distributed Sequence Numbers for Elasticsearch - Boaz Leskes

http://berlinbuzzwords.de/session/designing-concurrent-distributed-sequence-numbers-elasticsearch

Slides: https://speakerdeck.com/bleskes/designing-concurrent-distributed-sequence-numbers-for-elasticsearch

## Document level versioning

 - useful
 - but when there are multiple document updates, no way to track which happens first
 - this is what sequence numbers give you

## Why ordering?

 - can build other things on it
   - replication
   - replaying
   - building sub-views

 - primary replica sync:
   - sometimes a copy goes offline
     - currently do a file-based sync
     - see which ones are different, copy over the ones which are different
     - but "crude and rough"; often not much in common, takes ages
     - but, if could reliably order the operations, just need to copy the changes

## Indexing

 - elasticsearch has shards:
   - can index to any node in the cluster
   - node will route to the primary of the relevant shard
   - primary will route to replica
   - then finally acknowledge

 - Very concurrent; Lucene will make most use of server resources
   - means that changes can be applied in different orders to different segments

## Requirement

 - Correct
 - Fault tolerant
 - Support concurrency

# Could use; consensus algorithm, eg Raft

 - understandable
 - leader based
 - modular
 - used for replication by Facebook's HBase port, etc

 - how does it work?
   - raft.appendEntries: with each entry, it includes the new sequence number, and also the old one.
   - once quorum of nodes acknowledge, do commit (second communication from primary to replicas)
   - so, raft is single-threaded change control; can't do concurrently
   - also, read visibility happens on some nodes after others have committed
   - if there's a failure of a primary, can correctly pick new master

 - So:
   - quorum means lagging shards don't slow down indexing: good
   - But:
     - read visibility issues
     - tolerates up to `quorum - 1` failures
     - needs at least 3 copies for correctness
     - Challenges with concurrency

 - So, do we need quorum really?

# Master Backup replication

 - Leader based
 - Write to all copies before ack-ing (not just quorum)
 - Used by elasticsearch, kafka, etc

 - No ordering of writes
   - means can't work out what to rollback if the primary goes away
   - rollbacks on failure are more frequent

# No clear commit point

 - primary knows a commit point; knows the minimum change that is on all shards
   - replicas have a lagging "safe" point; no bigger than the "true" one on the primary

 - So design goal is to have a "safe" point on each shard.


# Question

 - How do you make sure sequence numbers are unique
   - Tricky part is if primary is lost and another primary takes over

 - How will this influence the behaviour of asking for an acknowledgments?
   - This isn't the same as the write consistency setting
   - Needs good documentation, and renaming some settings

 - Testing, how's that being done?
   - Have refactored replication code so it can be tested more easily (independent of rest of code, makes faster random testing, etc)

 - Thoughts about replicating segments instead of records?
   - Have added "shadow replicas" - more friendly to shared filesystems (but has some durability issues)
