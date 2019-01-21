Resources
Designing Data Intensive Applications

G1: Need for Distributed Systems
Why do we need distributed systems? What issues are we solving distributed systems with? For each point you mention, provide an example of a system. (reading)
What of the system can be distributed? Provide two examples for each aspect.
https://medium.freecodecamp.org/a-thorough-introduction-to-distributed-systems-3b91562c9b3c
Data Store: Amazon S3, HDFS
Computing: Allows for computationally-intensive tasks to be completed quickly by a cluster of commodity computers, or for very difficult tasks to be completed in less time by a network of very powerful computers
 Distributed training of ML models on GPUs
Data ingestion tasks divided over a kafka cluster
Messaging: Provide a central place for storage and propagation of messages/events inside the overall system, allow to decouple the application logic from directly talking with other systems.
Rabbit MQ, Kafka
Application: application running its back-end code on a peer-to-peer network
Massively multiplayer online game: very large of the users share a virtual world financial trading: real time processing, web search: index the contents of the web.


https://www.uio.no/studier/emner/matnat/ifi/IN5020/h18/pensumliste/2018-08-22-introtods.pdf  (slide 6, 3 examples are provided)


What are the concerns of designing distributed systems? Provide an example.
Read Chapter 8 in Designing Data Intensive Applications.

List of problems mentioned in Chap.8 as I read:
networks (“Unreliable Networks” on page 277):  When a client makes a request to a server node, unreliable networks causes problems at various stages. Fig.8 gives a good illustration.  When designing a distributed system, nobody should assume the network is reliable. Here are more detailed breakdown of these issues: 
1) A faulty node, for instance a load balancer, or a lead node. Rapid and explicit feedback about a remote node being down is useful, but you can’t count on it.
2) Timeouts and Unbounded Delays. Timeout is the time it takes and the uncertainty of declaring a node is dead.  Asynchronous networks have unbounded delays (that is, they try to deliver packets as quickly as possible, but there is no upper limit on the time it may take for a packet to arrive), and most server implementations cannot guarantee that they can handle requests within some maximum time.
3) If several machines send network traffic to the same destination, its switch queue can fill up, causing delays. In a distributed system, Batch workloads such as MapReduce (see Chapter 10) can easily saturate network links, and you have no control over or insight into other customers’ usage of the shared resources.

clocks and timing issues (“Unreliable Clocks” on page 287)
Applications depend on clock to determine the duration and timestamp of events. Distributed systems make timing trickier because of delays in the network and it’s hard to determine which happens first when multiple machines involved.
Monotonic vs Time of Day Clocks
Monotonic clocks measure duration between actions, but actual clock values are arbitrary - useless to compare M clocks between nodes
Time of Day clocks tell us time since the Epoch according to Gregorian calendar - depend on hardware quartz clocks that may drift depending on temperature of hardware
Can be updated by GPS receiver or NTP server - NTP limited by network latency 
Limits how accurate you can expect your data to be 
“Google assumes a clock drift of 200 ppm (parts per million) for its servers [41], which is equivalent to 6 ms drift for a clock that is resynchronized with a server every 30 seconds, or 17 seconds drift for a clock that is resynchronized once a day. This drift limits the best possible accuracy you can achieve, even if everything is working correctly” (page 289, designing data intensive applications)
		
		
G2: Distributed Systems Architecture
Note: There are sections of this resource that go deep into the concepts. You can read those at your own time and interest. Please understand the basic architecture and relate it to different systems.
Centralized v Decentralized Architectures: http://cse.csusb.edu/tongyu/courses/cs660/notes/distarch.php (Section 2 and 3)
            Centralized：
                                                                                                                                                                                            Application Layering


Examples for each type of architecture: Classify them to each of the architecture, Spark, Cassandra, Cockroach DB, Kafka, Flink, provide a one-liner for each example, discuss the architecture used, avoid going into detail for each of them. Everyone can always go into them and discuss this further.
Centralized: Spark,Flink
Decentralized: Cassandra, Kafka
Additional content/examples: 
https://www.industronic-blog.com/decentralized-vs-centralized-system-architecture/ 

G3: Distributed Data: Replication and Partitioning
Chapter 5: What is replication of datasets, different ways of replicating data? 
Leaders and Followers approach
One leader, which accepts writes
Followers, which accept reads (sync - leader waits for “OK”, async - leader doesn’t wait for “OK”)
Semi-sync: one leader - one sync follower - async followers
Multi Leader replication
Several clusters, each of which contains a leader with followers (e.g., coordinated data centers in various locations). This setup provides low latency and fault tolerance to network and machine malfunctions. However, reconciliation across leaders need to take place.
Used topologies: circular (one-to-two), star, all-to-all
Leaderless replication
Each replica can process read and write requests.
Consistency is achieved by read-repair (update on read) or anti-entropy process (“crawling”)
Issues with replication (consistency issues)
Ensure that data is consistent across leader and followers, and across leaders.
Chapter 6: What are the different types of partitioning the data? How to decide on what to choose and when?
	Key range
Effective range queries, can lead to hotspots (i.e. encyclopedias on different shelves)
Hash partitioning
Key is hashed which allows for more distributed load of data but range queries are ineffective(timestamps are hashed and distributed over nodes but then concurrent times are no longer grouped sequentially 
		
What is a partition key?
	
What is Hash-partitioning?
	Hash function applied to each key, each partition owns a range of hashes
Issues with simple partitioning methods
Simple Key:Value partitioning can lead to hot spots, hash partitioning loses the ability to have effective range queries
Avoiding skewness by rebalancing.
	Rebalance your partitions when one node gets “overheated”
	Side note on secondary indexing:
		Secondary indexing allows you to have a quick reference for features of your data. In order to maintain the benefits of partitioning your secondary indexes must be partitioned as well. There are two ways to do this. 
Document - here the features of the data in each partition is catalogued for quick reference. The problem with this is it makes searching for multiple instances of a single attribute difficult (called scatter/gather searching)
Global  - here features of data across all partitions are catalogued on different singular partitions. (For example, the reference to all red cars are stored on partition 1, all black cars on partition 2) This makes searching by attributes simpler, but makes for messier and more costly writes

More things to read: RAID:
https://www.digitalocean.com/community/tutorials/an-introduction-to-raid-terminology-and-concepts 

G4: Distributed Data: Maintaining Consensus and Consistency
Read Chapter 9 in Designing Data Intensive Applications. Important topics:
Dealing with concurrent problems
Linearizability
when multiple leaders interfacing with the same dataset, the dataset achieves consensus and is updated following a strict time sequence of when each leader interfaces with the dataset. 
Eventual Consistency: most basic consistency guarantees, where all replicas will eventually converge. That is when writing to a database, the read requests will eventually return the latest updated value once the replicas have converged. 
However there is no indication as to when the replicas will converge. So till the replicas converge, the read request may return any value. 
Distributed Transactions in Databases: how we utilize distributed system to make replicas of the same database and allow multiple leaders to interface individually, and then establishes a consensus as the update for the database for a given time interval. It includes the Cassandra and Sparks models illustrated below:
Internode communication in Cassandra 
gossip/peer-to-peer model, ring system, each node only communicates with adjacent node, make data in cassandra nodes consistent with each other
nodes periodically exchange state information about themselves and about other nodes they know about

pro: if one node fails, other nodes can keep working
con: nodes need to communicate frequently to ensure data consistency
Internode communication in Spark
Spark uses RPC (Netty) to communicate between the executor processes (master-slave model)

pro: easy to design/data consistency
con: if the master node fails…?

Group 5: Distributed Processing: Mapreduce Design pattern for processing 
Note: This group will set the tone for understanding distributed processing. Please take your time to understand how the framework works.
Counting
The classic example of MapReduce is simply counting items (often words).  This will help you better understand the phases of MapReduce:

Input 
Map
Combine
Partition
Sort
Shuffle
Reduce
Output

You can see a concrete example here, and also in Chapter 1 of the MapReduce Design Pattern book.

Sorting
Given a very large set of comparable data (too big to fit into the main memory of a single computer), order the data so that the entire data set is ordered completely.  Page 92 of the MapReduce Design Pattern book explains it.

Group 6: Distributed Processing: Examples of Mapreduce frameworks
Filtering
Given a large set of data (too large for a single machine), filter out all the data that doesn’t match a certain condition.  Check out Page 44 of the MapReduce Design Pattern book.
Explore Bloom Filtering on page 49 of the same book.

Filtering 
Is a mapping step
Evaluates each record separately and decides, based on some condition, whether it should stay or go. Consider an evaluation function f that takes a record and returns a Boolean value of true or false. If this function returns true, keep the record; otherwise, toss it out

Bloom Filtering 
Is a mapping step
Filter such that we keep records that are member of some predefined set of values. The predetermined list of values will be called the set of hot values. For each record, extract a feature of that record. If that feature is a member of a set of values represented by a Bloom filter, keep it; otherwise toss it out (or the reverse).
Is an approximation and there would be false positives. It is not a problem if the output is a bit inaccurate, because we plan to do further checking. 

Inner Join (Reduce Side)
Take two very large tables of data (where each one is too big to fit into the main memory of a single computer) and create a combined table.  See page 105 of the MapReduce Design Patterns book for an example of an Inner Join, and details on the Reduce Side pattern.


Inner join 
Is typically the reduce step
Intent is to join large multiple data sets together by some foreign key. 
It can join as many data sets together at once as you need. 
A reduce side join will likely require a large amount of network bandwidth because the bulk of the data is sent to the reduce phase. 
Details 
The mapper prepares the join operation by taking each input record from each of the data sets and extracting the foreign key from the record. The foreign key is written as the output key, and the entire input record as the output value. 
This output value is flagged by some unique identifier for the data set, such as A or B if two data sets are used. 
A hash partitioner can be used, or a customized partitioner can be created to distribute the intermediate key/value pairs more evenly across the reducers.
The reducer performs the desired join operation by collecting the values of each input group into temporary lists. For example, all records flagged with A are stored in the ‘A’ list and all records flagged with B are stored in the ‘B’ list. These lists are then iterated over and the records from both sets are joined together.


If you have time, explore the Map Side Join on page 119, which works when one of the tables is small enough to fit into main memory, but the other is too large.

Summary of Map Side Join / Replicated Join / Join one large and many small tables / inner join or left (=large dataset) outer join
Map side operation, no reduce step => performance benefits, one of the fastest joins

Example: Given small set of user metadata and large set of comments, enrich comment data with user data
In map setup phase, read in the small datasets into memory, possibly as a HashMap. This is where OOM errors can occur, if JVM heap isn’t enough.
In the map step, we join each record with the stored small dataset (HashMap). If we find the user ID of the comment in-memory (in the user data HashMap), we output record with the retrieved data from HashMap. 


Output has possible null values, when comment doesn’t have a match in the right side small table.

There’s no shuffling or sorting or reduce step necessary -it’s just appending everything together-  leading to performance benefits.
Other join types will require a reduce step, and we can no longer use this pattern.

