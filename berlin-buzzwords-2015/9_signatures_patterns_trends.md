# Signatures, patterns and trends: Timeseries data mining at Etsy - Andrew Clegg

http://berlinbuzzwords.de/session/signatures-patterns-and-trends-timeseries-data-mining-etsy

 - Etsy: data science team
   - helps people find items they'll love
     - recommendation
     - data-driven ranking
     - navigation
     - text mining; supporting auto-suggest, related items

   - also internal tool building
     - helping with security / growth / operations
     - helping other team to do their jobs
     - eg, image similarity for finding stolen images
     - spam detections
     - modelling how long customers are likely to stay with etsy
     - detecting anomalies

## Kale

 - Version 1:
   - skyline for anomaly detection
   - oculous for searching through timelines for similar ones
   - constraints:
     - 250k metrics coming in in parallel; mostly every 10 seconds
     - need live updating
     - nice user interface, fit into the workflow
   - more difficult than expected, but still doing it
     - 1m metrics now
     - Twitter, yahoo, netflix have all release tools in this area recently
 
  - What worked well?
    - timeseries similarity search
      - shape description alphabet, for making fingerprints, first stage method, search with sloppy phrase searching
      - then use a 2nd stage: use fast dynamic timewarping (see http://en.wikipedia.org/wiki/Dynamic_time_warping#Fast_computation)
    - graphical user interface

  - What proved hard?
    - Complicated architecture, hard to install, hard for people to understand when it goes wrong
    - Anomaly detection
      - Data from machines isn't normal, has power laws, mixtures of distributions, roughness spikes, periodic cyclic behaviour
      - 250k metrics; if threshold of significance p-value is 10^-6, still see false positives all the time
        - majority voting ensembles and auto-silencing repeated alerts helped a bit
	- but false alarms tended to be correlated
      - not every anomaly is a point outlier
        - periodic oscillations sometimes appear out of nowhere
          - could be daily cycles, garbage collection, etc
          - change in oscillations often significant
        - distributions can change unpredictably
	- trends change, baselines can suddenly shift
	- mostly it's not even a problem; even if it's a genuine anomaly; a change in a metric doesn't usually affect users.

# Kale 2.0

 - Phase 1: Thyme
   - library of algorithms and composable processing steps; platform agnostic
   - supporting flexible experimentation and prototyping
   - stream driven functional API; built on ReactiveX
 
 - Components:
   - signal processing; wavelet decomposition, filtering, FFT, DCT, smoothing
   - statistical tests; generalized ESD, kolmogorov-Smirnov, Mann-Whitney
   - fingerprinting and search; similar to before
   - interactive demo app

 - Why Java?
   - interop with existing platforms: kafka, samza, spark, ELK, Hadoop
   - opentsdb, datastax, etc, plugins should be easy
   - lots of maths stats resources to build on

 - Why ReactiveX?
   - Functional Reactive Programming, good for data flows

# Live demo

 - reads data from the microphone, take volume reading every few ms
 - graphs average volume levels, with low frequency component removed
 - shows P-value of anomaly
 - detects silence clearly, low value for clapping compared to baseline of talking

# Lessons

 - Anomaly detection is more than outlier detection
 - Not all anomalies should result in alerts
   - use in diagnostics
 - Good UI and workflow vital for production
 - Ensemble methods and auto-calibration a good idea
 - Timeseries search is feasible, by using fingerprinting

# Questions

 - Did you deal with non-consistent sampling or rare events
   - rare might work well with poisson based model
   - inconsistent sampling interesting; current assumption of regular sampling
     - should be possible to put quantization / interpolation into the pipeline to tidy that up before further analysis

 - feature extraction; any tips on what to do there?
   - have avoided computationally expensive feature extraction methods, want fast / close to real time
   - can get a lot out of wavelet decompositions; features here aren't quite the same thing as features in machine learning
   - use techniques to remove noise, not don't robust analysis of things yet
   - open to suggestions, but trying to keep usable for streaming
   - netflix has a hadoop-based thing doing detection based on SVD

 - Lots of servers; would like to detect a couple of instances misbehaving given the context of the cluster
   - Phase 2; look for collections of metrics which are abnormal compared to rest of cluster, or rest of machine; at ideas stage, happy to bounce ideas around
