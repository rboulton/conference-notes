# Fast decompression Lucene codec

"Low level fun"

http://berlinbuzzwords.de/session/fast-decompression-lucene-codec

Lucene compresses data to minimize IO, memory size, and be cache friendly.
 - Showed some numbers, similar to http://highscalability.com/numbers-everyone-should-know

Codec API determines how and where data is stored; separation between indexng/searching code and filesystem.

Codec API is "4 dimensional".

 - First find field in FieldsEnum
 - Then term in TermsEnum
 - Then doc in DocsEnum
 - Then position in PositionsEnum

Posting lists are encoded in lucene 5.0 (default) using "FOR" delta.

 - uses the delta
 - blocks of 128 values
 - bit pack each block
 - for remaining docs, encode with vint encoding
 - [see https://lucene.apache.org/core/4_1_0/core/org/apache/lucene/codecs/lucene41/Lucene41PostingsFormat.html]

Gave an example of encoding 10 numbers with block size of 4.

 - great compression and decoding speed
 - *can be vectorized*
 - But: no random access within the block (because using deltas), cost is determined by the largest delta in a block

## Vectorisation

Gave an example of adding two vectors
 - JIT => 32 machine instricutions
 - gcc => 24 machine scalar instructions
 - or, with some hints, gcc => 4 machine instructions with SSE2 (load vector1, load vector2, sum them, store result in memory)

This works 4 times faster [me - this is surely an over simplification, but should be faster]

## In hotspot

 - auto-vectorization
   - pattern recognition, only currently supports array initialization and array copy
   - cannot have if-statements (can do some bitmasking stuff, though)
 - explicit vectorization: JVM does not provide interfaces for this

 - workaround: HotSpot team recommend writing critical ("kernel") code in C/C++, and call via JNI
   - but cost of the JNI call can be significant, so lose the boost
     - JNI slow because of lots of checking, locking, parameter marshalling

 - Critical native
   - Looks like JNI method, but JVM does not check:
     - static and not synchronized
     - no exceptions can be thrown
     - does not use warppers
     - works with primitives
   - See https://bugs.openjdk.java.net/browse/JDK-7013347

 - Native FOR library: https://github.com/lemire/simdcomp
   - Supports SSE2, SSE4.1, AVX
   - C99 syntax
   - uses SIMD intrinsics

   - In SIMD can do about 10 instructions per clock cycle

# Benchmark

 - Java code:
   - java_vint: classic vint
   - java_FOR - classic FOR
 - JNI + native FOR
   - normal_JNI - usual JNI call
   - critical_JNI - critical native call

 - Environment; pretty much latest everything: gcc 4.9.2 on, JRE 1.8.0_40, fedora 21 (kernel 3.17.4), i5-4300M CPU @2.60GHZ

 - Decode blocks with fixes size
 - Every block contains random elements with fixed density - fixed density reduces jitter between results [me - isn't this going to make results a bit biassed]

 - Plotted decoding latency in ns/op  against  block size
   - normal JNI same as javaFOR for 128 entry block size (about 200ns)
     - but critical JNI around 90ns
   - critical JNI looks to be slow down proportional to block size
   - normal JNI performance gets close only when block size gets huge [me - basically, looks like a large constant overhead, as would be expected]

 - Still in progress, so it does not support lots of things
 - code at: https://github.com/griddynamics/solr-fork/tree/SIMDCodec
 - Benchmark searches top 10k most common terms in 49s, compared to Lucene50 codec which takes 60s.
 - Index is binary compatible between coded produced by Lucene50 codec.

## Future work

 - apply to more stuff
   - compression and intersection: https://github.com/lemire/SIMDCompressionAndIntersection
   - vbyte: https://github.com/lemire/MaskedVByte

## Questions

 - To get the speedup, does the native component need to be compiled specifically for the right processor
   - GCC provides, if -native flag is provided, compiles for specific platform
   - But is possible to compile for lots of platforms, and use CPU dispatching (see eg, http://www.agner.org/optimize/blog/read.php?i=121#121)
   - Can also fall back to JVM native

 - Currently, with Java 9, intel is sending patches regularly to detect more vectorisation opportunities.  Did you test with Java 9?
   - Have a plan to introduce some kind of pattern to JVM to detect these kind of patterns
   - But JVM is a huge platform, hard to support, lots of feature requests related to this.
     - Oracle drops these requests as WONTFIX because so hard to support
     - Starting from Java 10, some projects (like "grail") to support native calls from Java, so these intrinsics could be introduced as little extensions in Java.

 - Why can't you just use large block sizes
   - Compression rates start dropping
 - What's a reasonable block size
   - 128 items is good.  Numbers are just to demonstrate how normal JNI call compares.

 - What does FOR stand for?
   - Frame of Reference; set up a fram (the bits per value) and represent stuff in that frame
