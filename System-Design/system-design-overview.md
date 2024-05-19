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
  - Video to understand it all better [Link](https://www.youtube.com/watch?v=cODCpXtPHbQ)
  - GeoSpatial Data? -- Which Database is better for this ? [Information related to location]
    - Latitude, Longitude


# Database Details 
  - Redis
    - Probablistic Datastructure
    - Ttl for entries
  - Elastic search -- Works on Apache Lucene
    - Fuzzy search 
    - Inverted Index
    -

# Properties on DB's
  - Partitioning
  - Sharding
  - Indexing
