# Good Sites to get overview of System design
  - Hello Interview [Link](https://www.hellointerview.com/learn/system-design/in-a-hurry/introduction)
    - Scroll to view the common problems | Also has good Youtube videos for explanation


# DataBases and types [Overview]
  - Relational DB - ACID Properties -> SQL, Oracle SQL, MySQL
  - Caching -> Redis
  - Blob  [Images , Videos] -> BlogStorage(S3), CDN
  - Text Search (Search Optimized Database) [Fuzzy Search, Tag search] -> Elastic Search, Solr
  - Time Series [Logs, Monotoring, Metrics] -> Influx DB, OpenTSDB (Open Time Series DB)
      - specialised to store increment / append mode data
      - specialised to do queries based on time eg: last 5 mins , last hour etc (Chunck of Data)
  - Analytics [Columnar] (specific attributes to be queried in very large DB) -> Casandra, Hbase
  - Video to understand it all better [YouTube Link](https://www.youtube.com/watch?v=cODCpXtPHbQ)
  - GeoSpatial Data? -- Which Database is better for this ? [Information related to location]
    - Latitude, Longitude


# Database Details 
  - Redis
    - Popular usecases  
      - Probablistic Datastructure
      - Ttl for entries
      - Leaderboard
      - Rate Limiting
      - Proximity Search
    - Details for redis [HelloInterview-Redis](https://www.hellointerview.com/learn/system-design/deep-dives/redis)
  - Elastic search -- Works on Apache Lucene
    - Fuzzy search 
    - Inverted Index
    - Why is ElasticSearch Fast? [Link](https://www.encora.com/insights/elasticsearch-demystified-part-1)
  - NoSQL Generic 
    - Why is NoSQl fast?
      - LSM tree and SS table [Log Structure Merge Tree and Sorted String table] [Medium Blog](https://medium.com/@dwivedi.ankit21/lsm-trees-the-go-to-data-structure-for-databases-search-engines-and-more-c3a48fa469d2)
      - 

# Properties on DB's
  - Partitioning
  - Sharding
    - Highlevel overview of sharding [AWS Link](https://aws.amazon.com/what-is/database-sharding/)
    - Types of sharding
      - Range Based Sharding (Based on some column's range)
      - Hashed Sharding (Hashing one of the column's data)
      - Directory Sharding (Having a external directory like phonebook to know which data goes in which shard)
      - Geo Sharding (Based on location)
      - Entity-/relationship-based sharding [Read Sharding architecture and types](https://www.mongodb.com/resources/products/capabilities/database-sharding-explained)
    - Challanges with Sharding
      - Data Hotspots (To overcome shard key must be good)
      - Disbalanced shard (To overcome shard key must be good)
  - Indexing

# CAP Theorem
  - Consistency, Availability, Partition Tolerance.
  - We can usually have CP or PA but never CA
  - Also when designing a system think that CAP is not required for all components you can divide the system into multiple parts and then decide on CP or PA for each individual parts 
  - Good video to recap along with example [ByteByteGo](https://www.youtube.com/watch?v=BHqjEjzAicA&list=PLCRMIe5FDPsd0gVs500xeOewfySTsmEjf&index=13)


# Queues / Event Streams / Pub-Sub
  - Kafka
      - why is kafka fast? [Byte](https://www.youtube.com/watch?v=UNUz1-msbOM&list=PLCRMIe5FDPsd0gVs500xeOewfySTsmEjf&index=21)
          - 2 main pointers among many others
            - Sequential reads (about 10-100 times faster then Random reads)
            - Direct Memory Access [DMA]
