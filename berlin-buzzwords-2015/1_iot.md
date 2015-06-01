Analytics in the age of Internet of Things (Keynote)

 - Lots of connected devices with sensors.
 - est by 2020, 150billion connected devices.

 - eg, aircraft; sensor data being analyzed in real time; detect anomalies, prevent accidents
 - insurance; use data from a connected car key; monitor location, drive safety, driving score; cheaper insurance for safe drivers
 - Connected devices can affect our lives

 - Time series data; signal processing, stock market, sensor data, ...
   - timestamps and values
 - Protocols for talking to sensors
   - DDS: device to device - realtime
   - MQTT: device to server
   - XMPP
   - AMQP
 - Sensors have low CPU and memory and need a low energy communication network

 - Storing data
   - Large scale
     - flat file
     - database
     - things like cassandra

 - MQTT/REST => Kafka => storage & processing in storm/spark streaming => realtime and offline analysis

 - Example:
   - predict the physical activity that a user is performing
   - eg, walking, running, sitting, standing, going down or up stairs
   - classification: eg, decision tree, random forest, etc.
   - Collecting data
    - Smartphone data: timestamp, XYZ accelerometer, every 50ms.
    - Labelled CSV files containing this (from WISDM lab data)
   - Put into cassandra using the timestamp as the cassandra timestamp
   - each row has id, label, timestamp, 3 accelerometer values
   - code on https://github.com/nivdul/actitracker-cassandra-spark
   - Using spark, and MLlib
     - spark-cassandra-connector
   - Identify features that would be useful to use:
     - repetitive activities vs static activities
     - repetitive has waves;
       - for running Y-axis peaks space about 0.2 seconds
       - for walking, same byt about 0.5 seconds separated
       => average acceleration, variance in acceleration, average time between peaks, ...
    - break data into small windows (eg, 5 seconds) and compute features for each. Need small windows to get enough data.
    - Computing faetures:
      - mean; easy
      - computing time between peaks
        - find peaks; values which are greater than 0.9 max, filter out these, then use "map" function to get pairs and take time between them
    - Classifying:
      - decision trees; train the classifier, using MlLib

    - Have an android app to record the data
    - Use the trained models to predict.

    - applications
      - adapt music to your speed
      - detect lack of activity
      - smarter pacemakers
      - smarter oxygen therapy

