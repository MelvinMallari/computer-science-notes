# Design of Data-Intensive Applications by Martin Kleppman

## Chapter 3 Storage and Retrieval 
Data structures, algorithms and strategies to store and retrieve data in a database. 


### Advantages of LSM-Trees
* B-Trees requires writes of pages at least twice (to the tree and Write Ahead Log (WAL))
* B-Trees require writes of entire pages, as opposed to LSM-Tree compaction
* B-Trees are worse off in write heavy applications
* LSM-Trees can be compacted better, B-Trees produce more fragmentation

### Downsides of LSM-Trees
* Compaction process may coincide and interfere with read/write operations
* B-Tree performance more predictable
* Compaction may not keep up with write load
* In B-Trees, key exists in exactly one place in the index. In LSM-Trees they exist in multiple. 
* B-Trees are ingrained in historical database implementations although LSM-Trees are gaining traction. 

### Multicolumn Indices
* combines several fields into one key via appending
* e.g. phone book(lastname, firstname: phone number)

### Keeping everything in memory
* Awkwardness associated in persisiting data with Disks and SSD (in terms of actual data placement)
  * We tolerate this because disks and ssds are cheap compared to ram, and data isn't lost on power loss
* Can achieve durable RAM with special hardware, writing to logs, taking snapshots or replication
* In-Memory Databases achieve performance by not needing to encode data in a form that can be written to disk. 

### Transacation Processing of Analytics
* transaction: a group of reads and writes that form a logical unit. 
* analytics: making bulk queries usually over aggregate data to perform data analysis
* Online Transaction Processing (OLTP): databases reserved for business operations
* Online Analytics Processing (OLAP): databases reserved for data analaytics
* Extract-Transform-Load (ETL): process of extracting, transforming and cleaning data to read friendly schemas,
    and loading to data warehouses. 
* Typically use a seperate database for analytics called a data warehouse

### Summary
* Storage engines fall into two categories: optimized for transaction processing (OLTP) or analytics (OLAP)
  * OLTP usually user facing. Apps usually touch small num records in a query. Disk seek time often bottleneck.
  * Data warehouse, each query usually very demanding. Disk bandwidth often bottleneck. Column oriented storage increasing popular.
* Two main schools of thoughts for OLTP:
  * log structured school: only permit appending. Never updates a file that has been written (SSLTables, LSM-trees)
    * key idea is to systematically turn random-access writes into sequential writes to increase write throughput. 
  * update in palce school: treats disk as set of fixed-size pages to be overwritten. 
    B-trees is example. Use in many relational, nonrelational dbs.

## Chapter 4 Encoding and Evolution
* Formats for encoding data usually in two representations:
  * In memory data is kept in objects, structs, lists, arrays, hash tables, trees etc. 
  * When you want to write data to a file or send over netowrk you have to encode 
    it as some kind of self-contained sequence of bytes.
* Translation between two different representations known as _encoding_, _serialization_ or _marshalling_
* The reverse process is called _decoding_, _deserialization_, _unmarshalling_

### JSON, XML and Binary Variants 
* Standard encoding formats have various weaknesses but are widely standard and continue to get the job done
* JSON is less verbose than XML and therefore uses less space, but both use a lot more space that binary encodings

### Thrift and Protocol Buffers
* Thrift: FB, Protocol Buffers: Google 
* Each has there own schema and the protocols determine how the data is translated into binaries. 
* Benefit of this is the binary encodings are smaller, lighterweight so data transfer is less cumbersome. 
* Example Thrift Schema
```
struct Person {
  1: require string        userName,
  2: optional i64          favoriteNumber, 
  3: optional list<string> interests
}
```

* Example Protocol Buffer Schema
```
message Person {
  require string user_name       = 1;
  optional int64 favorite_number = 2;
  repeated string interests      = 3;
}
```

### Fields tags and Schema Evolution
* Schemas inevitably change. They must be backward & forward compatible. 
  * Backward compatible: new code can read records by old code
  * Forward compatible: old code read records by new code
* How is forward compatibility achieved?
  * If a field value is provided it is ommitted in the encoding. 
  * Field names can be changed, they aren't referenced by the encoded data anyway
  * Field Tags MUST remain the same, since that would make existing encoded data invalid
  * Old code just recognizes fields tags it doesn't know about and just skips the relevant (annotated) amount of bytes
* How is backward compatibility achieved?
  * New code can read the unique Field Tag Number and know what that is
  * New fields must NOT be required, must be optional or have a default value. Otherwise checks fail old data immediately. 
  * Can only remove fields that are optional. Also once removed, cannot reuse removed field's tag number. 

### Merits of Schemas
* Can be much more compact thant "binary JSON" variants since they can ommit field names from encoded data
* Schema valuable form of documentation. Must remain up to date since it's real code
* Must check backward/forward comptability before code pushes
* Type checking at compile time 

## MODES OF DATA FLOW

### Dataflow through Databases
* Backward/forward compatibility important bc old code might read new code and vice veersa
* Must be careful that old code doesn't accidentally update data with new fields without the new fields. 

### Dataflow Through Services: REST and RPC
* Simple model is that you have a server and client. However a server can be a client to another server. 
* This is has been called a Service Oriented Architecture or Microservices architecture. 
* In this type of architecture services can be worked on independently, again touching backward / forward compatibility
* Each service is owned by one team and should be able to update frequently without consulting other services. 

**Web Services** 
* When HTTP is used as the protocol for talking to the service. Misnomer as it can be in other contexts outside of web. e.g.:
  * Client on mobile or web app makes an ajax request to service over the internet
  * Service makes a request to another as part of an organization's microservice architecture
  * Services makes request to another third party service
* REST: not a protocol, but rather a design philosophy built on the principles of HTTP
  * simple data formats
  * URL for identifying resources
  * uses HTTP features for cache control, authentication and content type negotiation
* SOAP: an XML based protocol for making network API requests.
  * Commonly used over HTTP, although it aims to be independent from HTTP and avoids most HTTP features. 
  * comes with its own standards
  * Generally more complex, sometimes interoperability issues with other vendor's implementations, used by big enterprise companies.

**(Remote Procedure Calls) RPCs**
* Circa 1970s. The RPC model attempts to make calling a remote network service look like calling a method or function 
  * local functions are predictable and succeed or fail depending on parameters under your control. 
    network requests are not and fail for reasons outside your control. 
  * local functions have predictable results or failures results. Network requests may return without a result and
    you don't know if your request actually went through or not
  * If you retry a failed network request it could be the case that the requests are getting through 
    and the responses are getting lost. You are making multiple requests
  * Networks requests take longer than function calls. a lot longer
  * Arguments must be encoded to be sent over network. This get's complicated with larger objects as parameters
  * Client and Service may be implemented in different languages. RPC framework must translate datatypes in different languages 
    which may lead to ugliness. 
* Newer gen RPC frameworks are more explicit about the fact that remote requests are diff from local func calls
  * Usually support some type of _promises_ 
* Custom RPC protocols with a binary encoding format can achieve better performance over the generic JSON over REST
* JSON over REST is good for experimentation & debugging, has wide language support with vast ecosystem of tools
* For these reasons, REST dominates public API's. RPC frameworks are mainly used on requests between services in the same organization. 

### Message Passing Dataflow
* Somewhere between RPC and databases. 
  * Similar to RPC in that client's request is delivered to another process with low latency
  * Similar to database in that message is not sent over network connection, but over a _message broker_ or _message queue_
* Several advantages compared to direct RPC
  * Can act as a buffer if recipient in overloaded, improving system reliability
  * Can automatically redeliver messages to a process that has crashed, preventing loss of message
  * Avoids the need to know IP address and port number of recipient (useful in cloud development)
  * Allows one message to be sent to several recipients
  * Decouples sender from recipient (sender just publishes messages and doesn't care who consumes them)
* Sender sends and forgets about. It can receive a response but usually through a different channel
* Message brokers usually don't enforce data models, giving a lot of flexibility

### Summary
* Many services need to support rolling upgrades. This allows faulty releases to be detected and rolled back at smaller scales
* Rolling upgrades allow us to check backward/forward compatability in real time
* JSON, XML and CSV are widespread. These formats are somewhat vague about data types, so you have careful with things like binary strings or numbers
* Binary schema-driven formats like Thrift, Protocol Buffers are compact & efficient with clearly defined f/b compatability semantics.
  * Schemas are useful for documentation. 
  * Downside is that they have to be decoded to be human readable
* Scenarios in which data encodings are important:
  * Databases where writing and reading persisted data requries encoding and decoding
  * RPC and REST APIs where the client encodes a request, server decodes request and encodes a response that the client finally decodes 
  * Async message passing where nodes communicate by sending encoded messages that are decode by recipients
* With care, backward/forward compatability and rolling upgrades are achievable, enabling rapid app evolution and frequent code deployment. 

# PART 2 Distributed Data
* How do we tackle storage and retrieval of data across multiple machines? 
* Why would you want to split your data up?
  * Scalability: if data volume and read/write load goes up, you can spread this across multiple machines
  * Fault Tolerance/high availability: Multiple machines give you redundancy, allowing takeover of failed machines. 
  * Latency: You can have machines spread over the world, reducing distance packets have to travel. 
* Vertical scaling is easier, but no redundancy, exponential scaling. Horizontal Scaling is tough and more complex, but linearly scales. 
* Two common ways data is replicated across multiple nodes
  * Replication: Copying. Procides redundancy, may help performance
  * Partitioning: Splitting. Also known as sharding

## Chapter 5 Replication 
* Replication: keeping a copy of the same data on multiple machines communicating via a network
* In this chapter we will make assumption that data set is small and each machine can hold a whole copy
* Difficulty lies in changing of data. 
* Three popular algorithms for replicating changes: _single-leader_, _multi-leader_, and _leaderless_

### Leaders and followers
* Every write to database must be processed by every _replica_ (node that stores a copy of database).
* Most common solution is called _leader-based replication_ (aka _master-slave replciation_)
  * One replica is designated leader. Clients must send requests to leader which writes new data to its local storage
  * Whenever leader makes a write, it sends a data change to its followers as part of a _replication log_ or _change stream_
  * Client can read from leader or follower, but writes can only be processed by the leader
* There are _synchronous_ and _asynchronous_ writes to the followers
  * with _synchronous_ replication, 
    * advantage is follower is guaranteed to have up-to-date copy of data, readily available if leader fails
    * disadvantage is that writes cannot be processed if any of the followers fail
    * this disadvantage makes it impractical that all nodes are synchronous writes, instead we usually have only one synchronous follower. This is called a _semi-syncronous_ configuration.
* Often leader-based replication is configured to be completely async. This means writes are not guaranteed durable. 

### Setting up new followers
* Can't just copy from one node to another, data is in flux. 
* Process is:
  * take a snapshot of leader's database 
  * copy snapshot to new follower node
  * follower connects to leader, requests all changes since snapshot was taken.
  * follower processes the backlog of data changes until it's _caught up_

### Handling node outtages
* Follower Failure: Catch-up recovery
  * follower keeps a log of data changes received from leader
  * from it's log it knows its last processed transaction and request the backlogged changes from the leader
* Leader failure: Failover
  * One of the followers needs to be the new leader, in a process called _failover_
    * _Determine that the leader has failed_: if the leader node hasn't responded for some period of time it is assumed dead.
    * _Choosing new leader_: Done through election process (chosen by majority of remaining replicas) or elected by a previously elected _controller node_. Best candidate usually one with most up-to-date log. Getting node agreement is a consensus problem.
    * _Reconfigure the system to use the new leader_: clients must now send requests to new leader. Old leader must be ensured to transition into a follower. 
  * Many things can go wrong during failover:
    * Old leader rejoins cluster after new leader chosen, what happens any write discrepancy? Common solution is to discard old leader's writes, but this may violate durability expectations.
    * Discarded writes are dangerous if other storage systems outside the database need to be coordinated with db's contents.
    * Fault scenario in which two nodes believed to be leader called _split brain_. Safety shtudown mechanism must be carefully designed in this scenario.

### Implementation of Replications log
**Statement-based replication**
Simple implementation is leader logs every write request and sends that statement log to followers. This means that every follower executes that SQL statement. Breaks down during the following
  - any nondeterministic function like NOW() or RAND() produces incorrect replication
  - autoincrementing columns
  - statements that have side effects
Other replication methods are mostly used now a days.
**Write-ahead log (WAL) shipping** 
The log is an append-only sequence of bytes containing all writes to the database. Leader sends this to its followers. Used by PostgreSQL nand Orgacle. Disadvantage is that that the logs describe the data at a very low level in a way that's coupled to the storage engine. Operationally this means that all nodes must go down to during versioning update because different storage enginer versions are incompatible. 
**Logical (row-based) log replication**
 A sequence of records describing writes to a database at the granularity of a row. Logical logs are decoupled from storage engine internals, allowing backwards compatibility.
 **Trigger-based replication** 
A trigger lets you register custom application code that automatically executes data change in the db. Great overhead and prone to bugs and limitations, but is useful due to flexibility.

### Problems with replication lag
* If you read from a follower that has yet to be update, you will get conflicting information
* Follower will catch up eventually given a long enough wait in what is known was _eventual consistency_
**Reading your own writes**
* Possible to read a replica that has yet to process your async write. 
* Need _read-after-write consistency_
  * When reading something that may have been modified, always read from leader
  * If most thing in app are editable by user, have a period after writes where user just reads from reader
  * Client records timestamp of most recent write. Read replica makes sures it has a write after this timestamp. Could be logical timestamp (certain order) else clock sync critical.
* Sometimes you need _cross device read after write consistency_
  * Can't just remember timestamps, this metadata would have to be centralized
  * Must route request from all user's device to the same datacenter that holds the leader

**Monotonic Reads**
* Possible to see things move backwards in time. 
* Multiple queries can reach different replicas at different update levels.
* Eventual Consistency < Monotonic Reads < Strong Consistency
  * User reads only from one replica, however if the replica fails, they will be routed to another replica

**Consistent Prefix Reads**
* Must guarantee that certain things (like conversations/comments) are stored in proper order
* Simple solution is any writes causally related will go to the same partition

### Multi-leader Replication
* Doesn't make sense within a single datacenter configuration, but does in a multi datacenter one
* Benefits
  * Performance: distributed leaders are closer to users, reducing latency
  * Outtage Tolerance: If a data center with a leader fails, each data center continues operating independently and replica catches up when it boots up
  * Network Problem Tolerance: temporary network interrupts bother multi-leader configurations less
* offline operation and collaboritive editing are essentially multi-leader replication configurations

### Handling Write Conflicts
* Simplest strategy is to avoid them when possible. If the app makes sure all the writes go through the same leader, then conflicts cannot occur
* In the case where multiple leaders have conflicting writes, the database must resolve in a convergent way
  * Assign an ID, and pick the higher ID as the write. Or use time stamp in _last write wins_. Dangerously prone to data loss
  * Use the replica's number to decide. Data loss
  * Merge the values together e.g. ("B/C")
  * Store the changes in a data structure and prompt the user
* Can write custom conflict resolution on write and read

### Multileader replication topologies
* There exist different topologies in which leaders update each other circlular, star, all-to-all
* all-to-all is the least fragile, node's in a path may fail and prevent update propogation
* all-to-all does face replication issues due to varying network communication speeds

### Leaderless Replication
* Some storage systems like Dynamo allow writes to any replica in a leaderless system
* Some via direct client writes, others to writes to a controller node that handles direction of writes

**Writing to a database when a node is down**
* Write is considered successful if most writes are succesful
* When reading, read from multiple nodes. This prevents trouble with reading an outdated node
* How does a downed node catch up?
  * Read Repair: update nodes as you read and see stale information
  * Anti-entropy process: background processes that constantly look for data differences and copy missing data from one replica to another
  * Note that just read repair would mean data not read often will get very stale

### Quorums for reading and writing
* w: writes, r:reads, n: num nodes. As long as w + r > n we expect to get up to date value when reading because at least one of the r nodes we read must be up to date. 
* w, r, n parameters are configurable, can customize for application need

### Limitations of Quorum Consistency
* Often r and w are chosen to be > n/2. This ensures w + r > n up to n/2 node failures
* Above is not a rule, quorum is only reached when r and w have overlaps, other quorum assignments possible
* You may set w and r to smaller numbers such that w + r =< n
  * In this configuration, more likely to read stale values
  * Also more latency and higher availability. Can continue processing reads/writes even despite network latency or replica unavailability.
* Even under w + r > n, edge cases where stale value returned possible:
  * If a Sloppy Quorum is usedw writes may end up on differnt nodes than r reads
  * If two writes occurr concurrently, it is not clear which one happened first. Only safe solution is to merge concurrent writes. If a winner is picked based on timestamp, writes can be lost due to clock skew.
  * read can come concurrently with a write
  * if a node fails and is restored with stale data, number of replicas storing new value falls below quorum condition
  * Edge cases w/ unlucky timing

### Sloppy Quorums and Hinted Handoff
* DB's with appropriately configured quorums can tolerate failure of individual nodes without need for failover.
* Can also tolerate slow individual nodes because they can return when w or r nodes respond
* These characteristics make leaderless replication appealing for high availability and low latency that can tolerate occasional slow reads
* However network can cut off large amount of nodes, effectively "killing" them
* In this event, should we return errors for all requests? or should we write to remaining live nodes.
* Sloppy Quorum: writes still require w and r succesful response, but may include nodes not among the designated "n" nodes
* Hinted Handoff: Any writes sent to temporary nodes are sent to appropriate nodes
* Sloppy Quorums increase write durability at the expense of read consistency

### Detecting Concurrent Writes
* In dynamo-style dtabases conflicts can arise during concurrent writes and read repair/hinted handoffs.
* Concurrent write conflict solutions:

**Last write wins**
* Discard "old" writes, and always write more "recent" ones. Consensus is needed on what's older and newer
* Can attach a timestamp to each write and pick the biggest timestamps
* LWW achieves the goal of eventual convergence at the expense of durability

**The "happens-before" relationship and concurrency**
* An operation A happens beofre another operation B if B knows about A, or depends on A in some way
* Two operations are oncurrent if netiher happens beofre the other
* When you have two operations A, B. There are 3 possiblities. A before B. B before A. or A & B concurrent. Need an algorithm that determines causal or concurrent
* User versioning during reads and writes to consider which operations happen before and after

**Merging concurrently written values**
* Clients may have to clean concurrently written values
* Union of two writes usually reasonable
* Cannot simply delete item from DB if removed, must mark with appropriate version when deleted durinng sibling merge (marking with a "tombstone")
* There are data structures that exist that can perform this merging automatically

**Version Vectors**
* Single version number no sufficient w multiple replicas
* Need version number per replica as well as per key
* Each replica increments its own version number when processing a write and keeps track of version numbers it has seen from other replicas
* collection of numbers from all the replicas is called a _version vector_

### Chapter 5 Summary
* Replication achieves 
  * High Availability: system still up even if some nodes down
  * Disconnected Operation: system works, even if connections interrupted
  * Latency: Nodes aroudn the world makes finding closer one more likely
  * Scalability: More nodes == more nodes you can read from
* Three main approaches to replication:
  * Singer Leader Replication: Clients send all writes to a leader which sends a stream of data change events to other replicas. Reads can be performed by everybody
  * Multi-Leader replication: Writes sent to multiple leader nodes that send stream of data to each other and followers
  * Leaderless Replication: Clients sned each write to several nodes and read from several nodes in parallel. 
* SLR Popular because simple, no conflict resolution needed.
* MLA and LR more robust in presence of faulty nodes, network interruptions and latency spikes at the cost of being more complex and providing weak consistency guarantees
* MLA and LR allow concurrent writes and therefore have issues with conflicts

## Chapter 6 Partitioning
* Also known as sharding. The breaking up of data across multiple machines.
* Mainly want to partition for _scalability_
* Usually combined with replication. Choice of parititioning scheme independent of replication scheme

### Partitioning of Key-Value Data
* Goal is to spread the data evenly across nodes
* If partitionin is unfair we call it skewed. A paritition with high load is called a hot spot
* You could simply assign data to nodes randomly. However, when you read you have no idea where the data is, so you must query all nodes.
* Let's assume for now that you have a simple key-value data model in which you always access a record by its primary key. You could quickly find what you're looking for

### Partitioning by Key Range
* Assign a continuous range of keys to each partition. These ranges are not necessarily evenly spaced .. partitions are adapted to the data to distribute it evenly
* Within each partition key keys in sorted order. Key can be a concatenated index to fetch several related records in one query. 
* Must be careful when choosing the acess pattern to avoid hot spots. (e.g. choosing to partition by timestamp(day)- writes all go to a single partition leading to a hotspot)

### Partitioning by Hash Key
* Many distributed stores use a hash function to determine the partition of a given key. A good hash function takes skewed data and makes it evenly distributed
* Each partition falls into a range of hashes.
* By using the hash of the key for partitioning we lose the ability to do efficient range queries. Keys that were adjacaent are now scattered
* Can use a compromise between the two partitioning strategies known as a compound primary key

  * Hash the first key, sort by the second concatenated key

### Skewed Workloads and Relieving Hot Spots
* Hashing keys isn't enough to avoid Hot Spots entirely. (e.g. celebrity users may cause a storm of activity when they do something)
* This results in a large volume of writes to the same key
* Simple solution is to add a random number to the beginning or end key. This distributes writes across different nodes. Tradeoff is reads have to access all nodes with data.

### Partitioning and Secondary Indexes
* Usually doesn't identify are record uniquely, but rather is a way of searching for occurrences of a particular value
  * e.g. find all actions by user 123, find all articles containing the word hogwash
* Problem with secondary indexes is they don't map neatly to partitions. 

### Partitioning Secondary Indexes by Document
* e.g. whenever a red car is added to the database that database partition automatically adds it to the list of document ID's for the index entry color red.
* Reading requires care. Partitions have their own data. Unless you have done something special, there should be no reasons a particular field should only exist in a single partition. This means that you must query and combine reads on all partitions. This approach is called _scatter/gather_
* This makes read queries on secondary indices quit expensive

### Partitioning Secondary Indexes By Term
* Rather than have each partition have it's own local index, we can construct a global index that covers data in all partitions.
* This is called term partitioned, because the term we're looking determins which partition we look at. 
* Can partition the index by the term itself, or the hash of the the term.
* Global Secondary indexes make reads more efficient. Rather than scatter/gather over all partitions, we just need to make a request to the partition that has the term we want
* Downside is that we make writes slower, because we may affect multiple partitions of the index

### Rebalancing Partitions
* Things change in database: e.g. want more CPU's, disks, RAM, machines fail and require take over
* These changes require rebalancing. Rebalancing must meet some minimum requirements:
  * After rebalancing, the load should be share fairly between nodes in the cluster
  * Database should accept reads and writes while rebalancing
  * Minimize data movement for to maximize process efficiency

### Strategies for rebalancing
**How NOT to do it: hash mod N**
* Changing the number of nodes N, completely changes the mod of the hash, making moves super chaotic/inefficient

**Fixed number of partititons**
* Simple Strategy. Could have a fixed number of partitions. Partitions make up nodes. Can scale up by creating a new node that "steals" partitions of existing nodes
* Num of partitions is the max amount of nodes you can have. Shouldn't choose to high a num b/c of associated overhead costs. Too low of a number and you quickly hit a ceiling.

**Dynamic Partitioning**
* For databases with key range partitioning, a fixed number of partitions with fided boundaries would be very inconvenient. If you get the boundaries wrong, you can get all the data in one partition, with other empty partitions
* In dynamic partitioning if a partition grows too big, it can be split and vice versa
* At first when the dataset is small, all writes are handled by a single node until the partition is large enough to be split. This is mitigated by _pre-splitting_ the node on initiation.

**Partitioning proportionally to nodes**
* make the number of partitions proportional to the node. Have a fixed number of partitions per node
* the size of each partition grows proportionally to the dataset size while the number of nodes remains unchanged. When you increase the number of nodes, partitions became smaller again. 
* since larger data volume generally require a larger number of nodes to store, this approach keeps size of each partition fairly stable.
* Whena new node joins the cluster it randomly chooses a fixed number of partitions to split and takes ownership of half of those split partitions, leaving the other half in place. 

### Operations: Automatic or Manual Rebalancing
* Gradient exists between manual or automatic rebalancing. Some db systems generate recommendations but require an administrator to commit before it takes effect
* Automatic rebalancing more convenient, however rebalancing is expensive. Want to minimize operation 

### Request Routing
* When a client wants to make a request, how does it know which node to connect to? As partitions are rebalanced, assignment of partitions to nodes change. Problem is called _service discovery_
* Few different approaches to the problem:
  * Allow the client to contact any node (e.g. via round robin load balancer). Pass along until it hits relevant node.
  * Send all requests to a routing tier first that handles and forwards accordingly. Acts as a parition-aware load balancer
  * Require the client to be aware of the relevant node
* Problem is: how to maintain resolution between component handling partition assignment and partition assignment changes?
* Many distributed data systems rely on a seprate coordination service called ZooKeeper to keep track of cluser metadata. ZooKeeper maintains authoritative mapping of partitions to nodes. 
* Some systems use a _gossip protocol_ to talk update each other regarding partitioning assignment. Every node can reroute to requests to the correct node

### Ch 6 Summary
* We partition data when it is too large to be handled by one machine. The goal is to spread data and load across multiple instances and avoid hot spots
* Two main approaches to partitioning:
  * Key Range Partitioning: Keys are sorted and a partition owns all the keys from a min to max. 
    * Partitions are typically rebalanced by dynamically splitting the range into two subranges when the partitions are too big.
  * Hash Partitioning: A hash function is applied to each key, and a partition owns a range of hashes. This method destroys the ordering of keys, making range queries inefficient, but this distributes load evenly.
    * Common to create a fixed number of partitions in advance to assign several partitions to each node and move parittions form one node to another when nodes are added or removed. Dynamic partitioning also used. 
* Hybrid approaches possible, e.g. with a compound key: one part of key to identify the partition, another for the sort order.
* A secondary index also needs to be partitioned. There are two methods:
  * Document partition indexes (local indexes): secondary indexes are stored in the same partition as the primary key and value. This means that only a single partition needs to be udpated on write, but a read of the secondary index requires a scatter/gather across all partitions.
  * Term-partitioned indeces (global indexes): Secondary indexes are partition seperately, using indexed values. Entry in the secondary idnex may include rcords from all partitions of the primary key. During a write, several partitions must be updated, however reads can be from a single parittion. 

## Chapter 7 Transactions
Lots of things can go wrong in data systems:
* software or hardware can fail at any time (including during a write operation)
* app may crash at any time (including during series of operations)
* network interuption can cut off app from database or nodes from each other
* several clients may write to the database overwritting each other's data
* client may read partially updated data
* race conditions between clients can cause surprising bugs
* system has to be fault tolerant and have mechanisms in place to handle all this. this is hard
* a _transactions_ is way to group several reads and writes as a logical unit. These are executed as one operation. the transaction either succeeds (commits) or it fails (abort, rollback). If failure the app can safely retry.
* not every app needs transactions, some benefits can be had from weakening transactional guarantees

### The slippery concept of a transaction
* Lots of subjective interpretation on the meaning of "ACID" compliant.
* Systems that do not meet ACID creteria are sometimes called BASE for Basically Availabe Soft state and Eventual Consistency
* ACID: Atomicity, Consistency, Isolation, Durability

**Atomicity**
* Atomicity means different things in different contexts in computing
* In multi-threaded program, an atomic operation is one where the state is either before the operation or after, no in between
* ACID atomicity descrives what happens if a client wants to make several writes but a fault occurs after some of the writes have been processed.
* e.g. if a process crashes, network is interrupted, disk becomes full, etc- if the writes are grouped together into an atomic tx and the tx cannot be completed (committed), the the tx is aborted and the db must discard or undo any writes it has made so far in the tx.
* Can be thought of as abortability (although this isn't the widely accepted word for it)

**Consistency**
* ACID consistency is the idea that certain _invariants_ about your data must always be true. 
* If a tx starts with a database that is valid according to these invariants, and any writes during the tx preserves validity, then you can be sure the invariants are always satisfied.
* However, it is the app's responsibility to define its transactions correctly so that they preserve consistency. This is not something the database can guarantee. 
* If you write bad data that violate your invariants the database can't stop you. Database can only enforce some basic constraints (uniqueness, foreing key).
* Letter C doesn't really belong in ACID

**Isolation**
* _isolation_ means that two concurrently executing transactions are isolated from each other.
* The classic database textbook formalize isolation as _serializability_, which means that each transaction can pretend that it is the only transaction running on the entire database. 
* the database ensures that when the transactions are committed, the result is the same as if they had run _serially_ (one after the other)

**Durability**
* The promise that once a tansaction has committed succesfully any data written will not be forgotten
* This means writing to nonvolatile storage (disks/ssds), write-ahead-logs. In replicated databases, it may mean data has been copied to some number of nodes.
* No such thing as perfect durability.

### Single object writes
* storage engines aim to provide atomicity and isolation on the level of a single object on one node
* Atomicity can be achieved by using a log for crash recovery and isolation can be implemented using a lock on each object during access.

### Multi object transactions
* many have abandoned multi-object transactions because they are difficult to implements across partitions
* They can also get away in some scenarios where high availabiltiy or performance is required. 
* There are instances when multi object transactions make sense:
  * when updating forein keys in a relational db, we can make sure references are valid when inserting several records
  * When updating denormalized data in a document model system
  * when updating secondary indexes
* These applications can still be implemented without tx's but error handling becomes more complicated without atomicity and lack of isolation can cause concurrency problems

### Handling errors and aborts
* If the database is in danger of violating ACID properties, abort the tx right away
* Unfortunately, some systems abort and don't roll back the entire tx (rails, django ORM)
* although retrying an aborted tx is a simple and effefctive error handling mechanism, it isn't perfect:
  * If the write was actually successful, but network cuts before success received by client
  * if the error due to overload, retrying makes that worse
  * retry only worth after transient errors (network, isolation, deadlock) not permanent errors
  * if the tx has third party side effects, don't want multiple retries
  * if client fails while retrying, tx data is lost

### Weak Isolation Levels
* Serializable Isolation solves a lot of concurrency issues, but incur performance costs
* Some dbs choose not to pay those performances costs and implement weak isolation levels despite the harder to understand more subtle bugs they create

**Read Committed**
* When reading from a db, you will only read data that was committed (no _dirty reads_)
* When writing from a db, you will only overwrite data that was committed (no _dirty writes_)
* It is the default setting for Oracle, PGSQL, memsql
* To prevent dirty writes, db commonly lock the row when writing
* To prevent dirty reads, simple solution is to also use locks. However this can be slow. Common solution is to remember the old value and new committed value.
  * reads before the committed tx get the old value, reads after get the new
* One weakness of this is if data is read in the middle of a batched transaction, the read data would be skewed (_read skew_)
* There are cases where read skew is unacceptable
  * making a copy of a database - inconsistent reads become permanent if restored from backup
  * Analytic queries and integrity checks- common to scan over entire database. If you pickup a read skew during this scan, data won't make sense

### Snapshot Isolation and Repeatable Read
* Each transaction reads from a _consistent snapshot_ of the database. tx sees all data committed in the db at the start of the tx. 
* Even if the data is changed by another tx, each tx sees only the old data from that particular point in time
* very useful for backups and analytic queries. supported by psql, mysql, oracle, sql server
**How to implement**
* write locks to prevent dirty writes
* key principle is _readers never block writers, and writers never block readers_. this allows long running read queries as well as processing writes normally
* Database must potentially keep sveral different committed versions of an object side by side.
  * This technique is called _multi-version concurrency control (MVCC)_
* Snapshot isolation uses sepearate snapshots for an entire tx (as opposed to query for read committed)
* Whenever a tx writes anything to the db, the data it writes is tagged by the tx ID of the writer
* Each row has a created_by and optionally a deleted_by field containing the id of the tx that inserted this row into the table

**Visibility Rules**
* when a tx reads from the db, tx ids are used to decide which objects it can see and which are invisible
  * at the start of each tx, the db makes a list of all other tx in progress. any writes those tx have made are ignored
  * any writes made by aborted tx's are ignored
  * any writes made by txs with a later tx id are ignored 
  * all other writes are visible to app's queries
* Put another way an object is visible if both of the following conditions are true
  * at the time when reader's tx started, the tx that created each object had already committed
  * the object is not marked for deletion, or if it is, the tx that request deletion had not yet committed at the time when reader's tx started
* a long running tx using a snapshot for a long time, continuing to read values that (from other tx's POV) have long been overwritten or deleted.
  By never udpating values in place but instead creating a new version every time a value is changed, the db can provide a consisten snapshot while incurring only a small overhead

### Preventing Lost Updates
* the _lost update problem_ can occur when if an applicatin reads some value from the db, modifies it, and writes back the modified value (read-modify write cycle).
* If two tx's do this concurrently, one of the modifications can be lost (the later write _clobbers_ the earlier write)
* A variety of solutions have been developed for this common problem:

**Atomic write operations**
* Atomic operations are usually implemented by taking an exclusive lock on an object when it is read so no other tx can read until the update has been applied.

**Explicit Locking**
* Can be explicit when starting, ending a tx. e.g. `BEGIN TRANSACTION; ... COMMIT;`

**Automatically detecting lost updates**
* alternative is to allow them to execute in parallel and, if the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.
* can perform this check efficiently in conjunction with snapshot isolation
* great feature because it doesn't require application code to use any special db features

**Compare-and-set**
* in db's that don't provide tx's, you sometimes find an atomic compare-and-set oepration.
* Purpose of this operation is to avoid the update to occur only if the content of the page hasn't changed since the user started editing it.
* if content has changed and no longer matches old content, this update will have no effect.

**Conflict resolution and replication**
* locks and compare-and-set operations assume there is a single up-to-date copy- not applicable for multi-leader or leaderless configurations
* a common approach is to allow concurrent writes to create several conflicting versions of a value (known as _siblings_) and use application code or special data structures to merge these versions after the fact
* atomic operations can work well in a replicated context especially if they are commutative (work in any order), e.g. increment a counter
* Last Write Wins (LWW) leads to data loss, unfortunately this is the default configuration for many db's

### Write Skew and Phantoms
* Imagine a situation where two doctors are trying to give up their on call shifts
  * Alice and Bob both create a transaction where they reschedule their shifts, the transaction's concurrently check that `currently_on_call >= 2`, which returns true for both of their tx's, and th tx's succeeds
  * This isn't a dirty write or lost update because the tx's update two different objects.
  * This however has created a race condition
* write skew happens when different tx's read the same objects, then update some of those objects
* With write skew, our solutions are more restricted:
  * atomic single object operations don't help as mltiple objects are involved
  * automatic detection of lost updates doesn't help here
  * db constraints may or may not be an option. constraints have to affect multiple objects
  * if you can't use serializable isolation level, the second-best option is to explicitly lock the rows that transactions dpend on. 

### Phantoms causing write skew
* Write skew basically happens when the 3 steps happen
  * A SELECT query checks whether some requirement is satisfied by searching for rows that match some search condition
  * depending the result of the first query, the app code decides how to continue
  * if app decides to go ahead, it makes a write to the database and commits the tx. the effect of this write changes the precondition of the decision of step 2. 
* You can attach a lock to a row that doesn't exist, but if step 1 checks for something that DOESN'T exist you can't
* The effect where a write in one tx changes the result of a search query in another tx is called a _phantom_.
* Can artificially introduce a lock object into the database an approach called **materializing conflicts**, because it takes a phantom and turns it into concrete set of rows that can be locked.

### Serializability
* Lots of problems with race conditions
  * isolation levels hard to understand and incosistent among different db systems
  * hard to tell from app code whether it is safe to run a particular isolation level
  * there are no good tools to help us detect race conditions
* Solution to all these problems is to use _serializable_ isolation
* _serializable_ isolation guarantees that even though transactions may execute in parallel the end result will be if they had executed serially
* To understand why all db's don't implement serializability, we will look at the different implementations

**Actual Serial Execution**
* The simplest way to avoid concurrency problem is to remove concurrency entirely: execute only one transaction at a time, in serial order, on a single thread.
* Only recent developments have made this possible, performance-wise:
  * RAM became cheap enough such that it was feasible to keep an entire dataset in memory
    * when all the data in the tx needs to access is in memory, tx's can execute much faster than if they had wait for the data to be load in the disk
  * Database designers realized that OLTP tx's are usually short and only make a small number of reads and writes
* Approach used by VoltDB/H-Store, Redis and Datomic 
* Stored procedure: application must send the entire transaction code to the db ahead of time as a _stored procedure_.
  * This is done to minimize network hops between the app and query processor
* Pros and cons of stored procedures
  * Cons:
    * each db has its own language for stored procedures and they look kinda ugly by modern standards
    * code running in a db is difficult to debug and manage
    * db is much more performance sensitive than an app server, badly written stored procedure can cause more trouble
* serial tx's make concurrency control much simpler, but in this method, the single-threaded tx processor can be a serious bottleneck
* gains can be made if you partition your data correctly such that multiple cores can have a core dedicated to each partition
* Summary of serial execution
  * Every tx must be small and fast- it only takes one slow one to stall all others
  * limited to use cases where the active dataset can fit in memory
  * Write throughput must be low enough to be handled in a single cpu core or else txs need to be partition without requiring cross-partition coordination
  * cross-partition coordination tx's possible, but there is a hard limit on extent that they can be used

**Two-Phase Locking (2PL)**
* Concurrent reads are allowed on the same object, but as soon as anyone wants to write an object, exclusive access is required
* if tx A has read an object and tx B wants to write to the object, B must wait for A to commit or abort before it can continue
* if tx A has written an object and tx B wants to read that object, B must wait until A commits or aborts before it can continue
* in 2PL writers don't just block other writers, they also block other readers and vice versa
* Two types of lcok, shared mode or exlusive mode
  * shared mode: if a tx wants to red an object, it must first acquire the lock in shared mode. several tx are allowed to hold lock in shared mode simultaneously
    * if another tx already has an exclusive lock on the object these tx must wait
  * if a tx wants to write, it must acquire the lock in exclusive mode
  * if a tx first reads and writes an object it may upgrade its shared lock to an exclusive lock
  * After a tx has acquired the lock it must continue to hold the lock until the end of the tx
    * first phase: acquire the lock
    * second phase: release the lock
* transaction throughput and response times of queries are significantly worse under 2PL thatn under weak isolation
* this is partly due to overhead of acquiring and releasing all these locks, and mostly due to reduced concurrency
* deadlock: condition where a set of processes have requests for resources that cannot be satisfied
* deadlocks happen more frequently in a 2PL system- increasing overhead cause have to retry 
* predicate locks: locks on objects that much a certain condition
* predicate locks prevent effects of _phantoms_
* unfortunately, predicate locks do not perform well. if lot of locks by active tx- can be time consuming to check for matching locks
* _index range locking_ instead relaxing relaxes a predicate to match a greater set of objects, reducing the overhead of checking for matching locks

### Serializable Snapshot Isolation (SSI)
* SSI is fairly new but promises to be a good compromise with performance and serializability
* pessismistic vs optimistic concurrency control
  * Two-phase locking based on principle that if anything might go wrong, better to wait until situation is safe again
  * Serial execution is pessimistic to the extreme
  * SSI is a optimistic concurrency control technique. instead of blocking if something potentially dangerous happens, tx continue anyway.
    * when a tx wants to commit, the db checks weather anything bad happened. if so, it is aborted and retried.
    * optimistic concurrency control has better performance if system has spare throughput bandwidth, otherwise retries make things worse
* possible in snapshot isolation that premise of query changes before tx committed. How does a db know if a query result might have changed?
  * detecting reads of a stale MVCC object version
    * to prevent this from happening, db needs to track when a tx ignores another tx's writes due to MVCC visibility rules.
    * when tx wants to commit, db checks whether any of the ignore writes have now been committed, if so abort.
  * detecting writes that affect prior reads (write occurs after the read)
    * when tx writes to db, it must look in the indexes for any other tx that have recently read the affected data. 
    * lock acts like a tripwire notifying the tx's that the dat they read may no longer be up to date. 
* performance tradeoff b/w read/write tracking granularity overhead and num tx's abort occurrences.
* In some cases doesn't matter if the query premise has changed, tx is serializable anyways
* compared two 2PL big advantage of SSI is that reads don't block writes and vice versa
  * read writes are non-blocking at all
* Compared to serial execution, SSI isn't limited to a single thread
* long running tx's more likely to be aborted, so favors shorter ones

### ch 7 Summary
* transactions are an abstraction layer that allow apps to pretend certain concurrency problems don't exists
* without tx's various error scenarios (crashes, network interruption, power outages) mean that data can become insonsistent in various ways
* dirty reads: one client reads another client's writes before they have been committed.
* dirty writes: one client overwrites data that another client has written, but not yet committed
* read skew: client sees different parts of the database at different points in time. most commony addressed w/ snapshot isolation
....

## ch 8 The Trouble with Distributed Systems

### Unreliable Networks
* requests can be interrupted an any point in the send-receive cycle, for a variety of inane reasons 
* there is a balanced tradeoff between waiting too long for a response and cutting off too many lagged responses
* networks can become "congested" when a large amount packets are queued up to be processed

**TCP vs UDP**
* Latency sensitive applications (VoIP) use UDP over TCP
* TCP vs UDP is a tradeoff between reliability and variability
* UDP is good in situations where delayed data is worthless
* TCP is good in situations where you cannot have data loss

### Synchronous vs Asynchronous Networks
* circuit-switched networks are syncronous - think landline calls
* packet based networks are asyncronous
* Syncronous networks are statically partitions and therefore can have guaranteed throughputs
  * This comes at the cost of network utilization
* Asyncronous networks have high network utilization- packets jossle for bandwidth
  * This comes at a cost of guaranteed throughput

### Unreliable Clocks
* Applications depend on clocks in various ways like:
  * has this request timed out yet?
  * 99th percentle response time?
**Monotonic Versus Time of Day clock**
* time of day clock: returns the current date and time acording to some calendar- usually syncronized with Network Time Protocol NTP
* monotonic clock: suitable for measuring a duration of time (time interval)- here the difference matters
* Monotonic clocks don't need sync, but time-of-day clocks need to set according to an external time source or bad things start to happen

### Relying on Synchronized Clocks
* If you have software that relies on clock synchronization, it is essential to montiro the clock offsets bw all machines and declare off nodes dead. 
* Can't rely on timestamps for the ordering of events, bc if there is drift the clocks are off wrt each other. Who the last write is then subjective
* Clock readings could have a confidence interval [earliest, lastest]

### Process Pauses
* One way a node knows that it is still a leader is by obtaining a _lease_ from the other nodes (similar to a lock)
* Node runs code to check if it still has a lease to process a request. If within a time buffer of lease expiration- run the code
  * Problem with this is if there is an unexpected pause in the execution of the program (thread stops for say 15 seconds and you have a 10 second buffer)
    * The thread being paused for a significant portion of time is not an uncommon or impossible event
  * In this case, another node may be the leader. Since the thread is paused, nobody told the previous leader. It may process a future request unsafely.
* An emerging idea is to treat GC pauses like brief planned outages of a node and let other nodes handle requests from clients while one node is collecting its garbage

### Knowledge, Truth, and Lies
* a node in a network cannot know anything forsure, it can only make guesses based on messages it receives
* in distributed systems, we can make assumptions about the behaviour (the _system model_) and design the system according to those assumptions

### The truth is defined my the majority
* a node is declared dead by a majority, even if it is alive (responses not heard, GC pausing threads etc)
* frequently a system requires there to be only one of something (one leader, one username)
  * it is possible that, say during a GC pause, a node doesn't know it's no longer the leader, proceeds with a write, and corrupts data
* Problem above can be solved with _fencing tokens_ in which leaders have an incrementing token id, and any writes with a stale token is rejected

### Byzantine Faults
* distributed systems become much harder when we can no longer assume that nodes are being honest (e.g. bad actors)

**Byzantine Generals Problem**
* There are n generals who need to agree, and their endeavor is hampered by the fact that there are some traitors in their midsts.
* Most of the generals are loyal and saend truthful messages, but traitors may try to deceit or confuse others by sending fake or untrue messages.

* A system is _Byzantine fault-tolerant_ if it continues to operate correctly even if some of the nodes are malfunctioning and not obeying protocol.
  * e.g. aerospace where some sensors may be compromised due to radiation
  * blockchain systems where there are bad actors

* in web systems we usually do not need this level of fault tolerance
* we can however card against weak forms of lying easily:
  * corrupted network packets
  * sanity checking of values
  * syncing multiple NTP clients to cross-check their responses

### System Model and Reality
* system models are abstractions that describe what things an algorithm may assume. Three common ones:
  * Synchronous model
    * assumes bounded network delay, process pauses, clock error. Meaning none of those things exceed some upper limit. 
    * not a very realistic model because unbounded delays and pauses occurr
  * Partial synchronous model
    * behaves like a synchronous system most of the time, but sometimes exceeds bounds
    * a realistic model because most things happen on time and sometimes not
  * Asyncronous Model
    * algorithms are not allowed to have any timing assumptions. Does not even have a clock (so no timeouts)
* Types of node failures:
  * crash-stop-fault: nodes can only fail by crashing. once it stops responding, it's gone forever.
  * crash-recovery fault: node may crash at any moment and come back at some unknown time
  * Byzantine (arbitrary) faults: nodes can do anything, including tricking and deceiving other nodes

* to define and algorithm as correct, we can describe its properties. e.g.
  * uniqueness: no two req for a fencing token may return the same value
  * monotonic sequence: sequences have to be incremental
  * availability: a node that requests a fencing token and does not crash eventually receives a response
* safety: nothing bad happens
  * if a safety property is violated, we can point to a particular point in time which it was broken
* liveness: something good eventually happens
  * may not hold at some point in time, but there is hope it may be satisfied in the future
* theoretical abtract models are sometimes wrong, but are useful to reason about systems (otherwise there's no ground to stand)

### Summary
* There are lots of things that can go wrong in a distributed system. partial failures are the defining characteristic of distributed systems
* to tolerate faults is to first detect them, but even this is hard:
  * no accurate way to determine this, so most systems use timeouts
  * timeouts can't distinguish a dead node from network issues
  * nodes may become _limp_: their connection is alive but constrained to an incredibly low speed.
* Once a fault is detected making a system to tolerate it is not easy either:
  * no global variable, shared memory or common knowledge or shared state- nodes can't even agree on time
  * only way to communicate is over a unreliable network
  * nodes much reach a quorum for major decisions
* Scalability, fault tolerance and low latency is why we deal with the mess of distributed systems

### Chapter 9 Consistency and Consensus
* best way to build fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once and let app rely on those guarantees
* One of the strongest consistency models in _linearizability_
* linearizability is a _recency guarantee_. Basic idea is to make a system appear as if there were only one copy of data and all operations on it are atomic
* linearizability vs serializability: in linearizability, order matters. serializability is where things must come in order, but any order can work. 
* where is linearizability useful? 
  * consistency on who is the leader
  * consistency on constraints and uniqueness (e.g. who has a username?)
  * cross-channel timing depencies: when two services have rely on a consistent state, no race conditions

### Implmenting a Linearizable System
* Lets consider systems and see if they can be made linearizable.
  * Single-leader replication (potentially linearizable)
    * if you make reads from a leader or synchronously updated followers, they have the potential to be linearizable.
    * not every single-leader db is actualy linearizable either by design (SSI) or due to concurrency bugs
    * this requires that we know who the leader is- remember problems with this
  * Consensus Algorithms (linearizable)
    * contain measures to prevent split brain, this is how zookeeper and etcd work
  * multi-leader replication (not linearizable)
    * not linearizable bc they concurrently process writes on multiple nodes and asynchronously replicate them to other nodes
    * for this reason can produce conflicting writes that requires resolution
  * Leaderless Replication (probably not linearizable)
    * it's possible to have non-linearizable reads even with quorum 

### The Cost of Linearizability
* CAP theorem: Consistency, Availability, Partition - choose three
  * if your app _requires_ linearizability, and some replicas are disconnected from others due to a network problem then they are _unavailable_ until issue fixed
  * if your app does not require linearizability, then the app can remain available in the face of a network problem, but not linearizable
  * Misnomer because you _have_ to pick Partition, because partition tolerance _is_ a type of fault.
  * probably better named _either consistent or available when partitioned_
* Often we make the tradeoff of availability over consistency within hardware (like RAM) for performance reasons

### Ordering guarantees
* Ordering is very important in distributed systems. There's a deep connection bw ordering, linearizability and consensus.

### Ordering and causality
* Causality imposes an ordering of events, cause must come before effect (e.g. question before answer, send before receive)
* _causal order_ is not _total order_
  * Linearizability: has total order of operations. for any two operations we can always say which one happened first
  * Causality: has partial ordering. Two events are ordered if they are causally related, but are incomparable if they are concurrent.
    * some operations are ordered with respect to each other, but others are incomparable
* according to these definitions, there are no concurrent operations in a linearizable data store.
* There must be a single timeline along which all operations are totally ordered. Concurrency means that the timeline branches and merges again. 
* linearizability implies causal order
* linearizability has signifcant performance costs. But you don't need linearizability to achieve causal order.
* Causal Consistency is the strongest guarantee you can have while remaining available during network faults
* to determine causal dependencies, we need some way of describing the "knowledge" of the system
  * in order to achieve this, the db needs to know which version of the data was read by the application

### Sequence Number Ordering
* we need not require the significant performance overhead of tracking all data that has been read. Instead we can use _sequence numbers_ or _timestamps_ to order events.
* In lamport timestamps, each node has a unique identifier and each node keeps a counter of the number of operations it has processed. (counter, nodeID)
* every node keeps track of the maximum counter value it has seen so far and includes the maximum on every request
* when a node receives a request of response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum
* Although Lamport timestamps define a total order of operations that is consistennt with causality, they are not sufficient to solve many distributed systems problems
* use of latest lamport stacks is a useful solutiong _after_ the fact. it is not sufficient when a node has just received a request from a user to create a username and needs to decide _right now_
* Moreover this algorithm is does not allow _availability_

### Total Order Broadcast
* A single-leader replication determines a total order of operatiosn by choosing one node as a leader and sequencing all operations on a single CPU core on the leader
* Challenge is to scale system if throughput greater than single leader capability and how to handle failover. This is known as _total order broadcast_ or _atomic broadcast_
* Total Order Broadcast: a protocol for exchanging messages between nodes:
  * Reliable Delivery: No messages are lost. Delivery to one node is delivery to all
  * Totally Ordered Delivery: Messages delivered to every node in same order
  * This is exactly what you need for replication
* Another way of looking at total order broadcast is that it is a way of creating a log (like a replication, transaction or write-ahead log)
  * Delivering messages is like appending to the log
  * all nodes must deliver teh same messages in same order, all nodes can read the log and see the same sequences of messages
* TOB is useful for implementing a lock service (fencing tokens)

### Distributed Transactions and Consensus
* Consensus is important for things like _leader election_ or _atomic commits_
* atomic commit problem: have to get all nodes to agree on outcome of transaction: either all committ or rollback.
* the FLP result is a proof that consensus between nodes cannot be reached only within a very restrictive systems model (cannot use clocks or timeouts)

### Atomic Commit and Two-Phase Commit (2PC)
* Two Phase Commit is an algorithm for achieve atomic transaction commit across multiple nodes
* process is similar to a traditional western marriage ceremony
  * minister asks the bride and groom individually whether each wants to marry the other
  * typically receives acknowledgement
  * the minister announces the couple husband and wife and the transaction is committed
* 2PC process
  * when app wants to begin distributed tx, it requests a tx id from the coordinator
  * app begins single node tx on each participants. attaches globally unique id to single node tx. if anything goes wrong coordinator can abort
  * when app is ready, coordinator sends prepare request to all participants. By replying yes node promises to commit tx without error if request, surrenders the right to abort the tx without actually committing
  * when coordinator receives all prepare request responses, it makes decision to commit or abort. called the _commit point_
  * commit or abort request sent to all participants. if req fails, coordinator must retry forever until success. no going back. if participant has crashed, the tx will be committed when it recovers. 
* 2 crucial point of no returns: when a participant votes yes it promises it will be able to commit later. when the coordinator decides, the decision is irrevocable. 
* if a coordinator crashes the participants will wait until it returns

### Distributed Transactions in Practice
* two types of distributed transactions:
  * database-internal distributed transactions: internal transactions among nodes of that database
  * heterogeneous distributed transactions: partipants are two or more different technologies. 
* Exactly-once message processing: a message from a message queue can be acknowlege as processed if and only if the database transaction for processing the message was succesfully committed
* XA transactions: X/Open ZA is a standard for implementing two-phase commit across heterogenous technologies
  * XA is a a C CAPI for itnerfacing with a transaction coordinator
* holding locks while in doubt:
  * why can't the rest of system get on and ignore the in doubt transaction that will be cleaned eventually?
  * database transactions usually take exclusive locks to prevent dirty writes
  * db cannot release these locks while transaction commits or aborts
  * if coordinator's log is lost, the locks are held forever or until manually resolved
* must treat coordinator like a kind of database
  * must be replicated or SPOF
  * coordinator logs a form of state, no longer stateless servers
  * XA needs to be compatible with a wide range of data systems

### Fault Tolerant Consensus
* A consensus algorithm must satisfy the following
  * uniform agreement: no two nodes decide differently
  * integrity: no node decides twice
  * validity: if a node decides value v, then v was proposed by some node
  * termination: every node that does not crash eventually decides some value
* termination property formalizes idea that consensus algorithm cannot simply sit around and do nothing forevre, it must make progress
* system model of consensus assumes that when a node "crashes" it suddenly disappears and never comes back
* in this model any algirthm that has to wait for a node to recover is not able to satisfy the termination proeprty- meaning 2PC does not meet requirement for termination

### Consensus algorithms and total order broadcast
* total order broadcast requires messages to be delivered exactly once in the same order to all nodes
* total order broadcast is equivalent to repeated rounds of consensus:
  * agreement property: all nodes decide to deliver message in same order
  * integrity: messages are not duplicated
  * validity: messages are not corrupted or fabricated
  * termination: messages are not lost

### Single-leader replication and consensus:
* Single leader systems takes all writes to the leader and applies them to the followers in the same order
* how come we didn't worry about consensus then?
  * if leader is chosen manually you have a "consensus algorithm of a dicatorial variety
* However to elect a leader you need consensus (or you get split brain/ other faults). How you break out the conundrum?
* Epoch numbering and quorums
  * a node can't trust its own judgement, before a leader can do anything it must check to see if there isn't another leader with a higher epoch number
  * leader node must collect votes from a quorum of nodes. 
  * node votes in favor of proposal if it's not aware of another leader with higher epoch num
  * If the vote on the proposal does not expose a higher epoch number, the current leader can conclude no leader with a higher epoch number has happened.

### Limitations of Consensus
* Consensus alg's bring concrete safety properties (agreement, integrity, valididty) where everything else is uncertain
* they provide total order broadcast
  * and therefore linearizable atomic operations in a fault-tolerant way
* benefits do come at a cost
  * process by which nodes vote on a propsal before they are decided are a kind of synchronous replication
  * consensus systems always require a strict majority to operate
  * frequent failures means frequent leader elections and less actual work

### Membership and Coordination Services
* ZooKeeper et al are often descrived as "distributed key-value stores"
  * How are they diff than db's?
  * designed to only hold small amoutns of data to be kept in memory
* ZooKeeper implements other interesting features:
  * Linearizable atomic operations: if several nodes try to perform the same operation, only one will succeed
  * Total ordering of operations: provides a fencing token by giving tx ID to all operations
  * Failure detection: if a heartbeat ceases for a certain duration, zookeeper declares the session dead
  * Change Notification: provide notifications to changes when a client joins a cluster, or fails etc. removes need to poll

### Allocating work to nodes
* leader election process useful for job schedules and similar stateful systems
* also when you need to assign paritions to certain nodes
* can achieve these tasks with atomic operations, ephemeral nodes and notifications in ZooKeeper
* trying to perform majority votes over many nodes would be terrible, instead zookeeper keeps a fixed number of nodes

### Service Discovery
* ZooKeeper and other services often used for service discovery- to find which IP address to connect to particular service
* if your service already knows who the leader is, makes sense to expose to other service who that is

### Membership Services
* a membership service determines which nodes are currently active and live members of a cluster

### Chapter 9 Summary
* causal ordering is a weaker form in linearizability, but more performant and less sensitive to network problems
* causal orering via lamport stacks isn't enough. service needs to know _now_ not looking back. therefore we need to figure out _consensus_
* Turns out that a wide variety of problems are reducible to consensus, and therefore if you solve one you solve them all (because you solved consensus)
  * linearizable compare and set register: register needs to atomically decide whether to set its value based on its current value
  * atomic transaction commit: db must decide whether to commit or abort a distributed transaction
  * total order broadcast: messaging system must decide messaging order
  * locks and leases: during a race for a lock/lease lock decides which one acquired it
  * membership/coordination service: system must decide which nodes are alive/dead
  * uniqueness constraint: constraint must decide which records are allowed to have values and which to apply constraint violation
* This is all easy when a single node has to make the decisions. However when a leader fails or cannot be reached, there are three ways to handle the situtation:
  * wait for leader to recover. this doesn not satisfy termination property
  * manually fail over by getting humans to choose a new leader node. limited failover speed (relying on humans)
  * use an algoirthm to choose a new leader. this requires consensus.
* Single leader databases can provide linearizabiilty withotu a consensus algorithm on every write, but this only pushes the need for consensus down the road.
* ZooKeeper outsources this consensus service
* leaderless and multi-leader systems typically do not use global consensus

## Chapter 10 Batch Processing
* Service: Waits for a request or instructions from a client to arrive to send a response back.
* Batch Processing System: takes a large amount of input data, runs a job to process it and produces some output data
  * ran periodically
  * primary perfomrance measure is througput
* Stream processing systems (near-real-time systems): somewhere between online and offline/batch processing. 
  * consumes inputs and produces outputs
  * stream job operates on events shortly after they happen
* MapReduce is a fairly low-level programming model compared to parallel processing system

### Batch Processing with Unix Tools
* Unix commands allow simple log analysis
* Unix Philosophy:
  1. Each program does one thing well
  2. Expect the output of every program to become the input of another
  3. Design and build software to be tried early, ideally within weeks. Throw away clumsy parts and rebuild
  4. Use Tools in preference to unskilled help to lighten a programming task

### MapReduce and Distributed File Systems
* MapReduce is a bit like Unix tools, but distributed across potentially thousands of machines
* MapReduce does not modify input. Output files are written once, in a sequential fashion
* MapReeduce jobs read and write files on a distributed filesystem. 
  * in Hadoop's implementation it's called HDFS (Hadoop Distributed File System)
* HDFS is a shared-nothing principle. (each node uses it's own non-special hardware)
* HDFS consists of a daemon process running on each machine exposing a network service that allows other nodes to access files stored in the machine
* HDFS conceptually creates on big file system using disk space on all machines running the daemon
  * SImilar to RAID which provides redudnancy across several disks attached to the same machien

### MapReduce Job Execution
* MapReduce is a programming framework which you can write code to process large datasets in a distributed file system like HDFS
  1. Read a set of input files, break it into records
  2. Call the mapper func to extract a key and value from each input record (map)
  3. Sort all the key-value pairs by key
  4. Call the reducer func to iterate over the sorted key-value pairs. If multiple occurrences of same key, sort makes them adjacent to minimize memory usage via combining (reduce)
* Mapper: 
  - called once for every input record and its job is to extract the key and value from input record. 
  - may generate any number of key-value pairs
* Reducer: 
  - takes key-value pairs produced by the mappers, collects all the values belonging to the same key
  - calls the reducer with an iterator over the collection of values
* mapper prepares data into a form suitable for sorting. Reducer processes the sorted data.
* main difference from Unix pipe commands is that MapReduce can parallelize a computation across many different machines without having to write explicity parallelized code. 
  * mapper and reducer only operate on one record at a time, they don't need to know where the input is coming from, our output is going to
  * framework handles the complexities of moving data between machines

* when a mapper finishes reading its input files and writing its sorted outputs files. The MapReduce scheduler notifies the reducer that they can start fetching the output files from the mapper.[]
* reducers connect to each of the mappers and download the fiels of sorted key-value pairs for their partition
* the process of paritioning by reducer, sorting, and copying data partitions from mappers to reducers is known as the _shuffle_

### mapreduce workflows
* the range of problems you can solve with single mapreduce job is limited
* a single mapreduce job could determine the number of page views per URL, but not the most popular URL- that requires a second round of sorting
* Thus, it is very common for mapreduce jobs to be chained together into workflows- output of one job is the input to another
* map reduce has no concept of indexes
* with mapreduce if you give a job a set of files, you read the entire content of the files
  * this is expensive if you just want to look at a small set of files
* however in analytics it is common to want to look at aggregates
  * e.g. task may need to correlate user activity with user profile information
  * simplest implementation is go over activity events one by one and query user db for every user ID it encounters
    * limited throughput
  * better approach would be to take a copy of the user db and put it in the same distributed file systems as the log of users activity events
    * in this case you have the user db and user activity records as files in the HDFS and can run a mapReduce job

### The output of batch workflows
* what is the result of all this processing, once done? why are we running all these jobs in the first place?
* Google's orignal use of MapReduce was to build indexes for its search engine
* full-text search index are essentially files in which you can efficiently look up a particular keyword and find a list of all the document IDs containing that keyword
* if you need to perform full-text search over a fixed set of documents, batch process of building the indexes
* mappers partition set of documents as needed. reducers build the index for its partition. index file is written to the distributed file system
  * building document-partiion indexes parallelizes well
* another common use is to build machine learning systems such as classifiers (spam filters, anomaly detection, image recognition)
* output of batch jobs often some kind of database
* how does th eoutput of the batch process get back into a database where the web app can query it?
  * good solution is to build a brand new database inside the batch job and write it as files to the job's output directory in the distributed file sytem

### Comparing Hadoop to Distributed Databases
* hadoop is somewhat like a distributed version of Unix
  * HDFS is the filesystem
  * MapReduce is a quirky implementation of a Unix Process
* massively parallel processing (MPP): db's that focus on parallel execution of analytic SQL queries on a cluster of machines
* db's require a particular model, while distributed fs can just be a sequence of bytes
  * in practice it's better to make data available quickly- in any format. it's more valuable than trying to deicde the data model up front.
* MPP's are monolithic, tightly integrated pieces of software that take care of storage layout on disks, query planning, scheduling and execution
  * can achieve good performance on query's it's designed for
* however not all kinds of processing can be expressed in SQL queries
* MapReduce gave engineers the ability to easily run their own code over large datasets

## Chapter 10 Summary
* Unix tool philosophy (in awk, grep, sort) is carreid forard into MapREduce
* Two main problems that distributed batch processing need to solve are 
  * Partitioning:
    * mappers are partition according to input file blocks. output is repartitioned, sorted and merged intoa configuarable number of reducer partitions
  * Fault Tolerance:
    * mapreduce frequently writes to disk, making it easier to recover from an individual failed task without restarting job.
      * this slows down execution in the failure free case
* Several join algorithms for MapReduec 
  * Sort-merge joins
    * each inputs being joined goes through mapper that extracts join key. By partitioning, sorting and merging all records with same key go to same reducer
  * Broadcast hash joins
    * one of two joins is small, so it is not partition and can be entirely loaded into a hash table. 
    * Can start mapper for each partition of the large join input, load hash table for small input into each mapper then scan over large input one record at a time querying the hash table for the record
  * Partition Hash joins
    * if two joisn are partition in the same way (using same key, same hash function and same partitions) then hash table approach can be used indpendently for each partiion
* callbacks in distributed batch processing engines (mappers and reducers) assumed to be stateless. This allows retries without side effects.
* frame can guarantee fault tolerance, your code doesn't have to worry about that
* distinguishing featuer of batch processing si that it reads some input data and produces some output data wihtout modifying input
* data is _bounded_, it has a known and fixed size. a job is eventually complete.

## Chapter 11 Stream Processing
* in reality, a lot of data is unbounded because it arrives gradually over time
* to reduce the delay of batch processing, we can run the processing more frequently or even continuosly by abandoning the fixed time slices entirely
* "stream" refers to data that is incrementally made available over time
* a record is known as an _event_, a small, self-contained immutable object of something that happened with a timestamp
* events are generated once by a _producer_, then potentially processed by multiple _consumers_
* best way to connect producers and consumers is through a notification mechanism. this prevents the need to poll. specialized tooling has been developed to deliver notifications.

### Messaging Systems
* a common approach for notifying consumers about new events
* to differentiate systems ask these two questions:
  * what happens if the producers send messages faster than the consuemr can process them
    * three options: drop messages, buffer messages in a queue, apply backpressure (blocking producer from sending more messages)
  * what happens if nodes crach or temporarily go offline - any messages lost?
    * tradeoff between writing to disk for durability or performance loss.
    * acceptability of message loss dependent on your application
* some messaging systems use direct network communication between produces and consumers without intermediary nodes

### Message Brokers
* a widely used alternative to send messages (also known as a message queue).
* essentially a kind of database that is optimized for handling message streams
* runs as as server with producers and consumers connecting to it as clients
* producers write messages to the broker, and consumers receive them by reading them from the broker
* by centralizing the broker, these systems can more easily tolerate clients that come and go. durability moved to broker
* consequence of queueing is that consumers are generally asynchronous

### Message brokers compared to databases
* some message brokers can participate in two-phase commit protocols
* important diff's with db's:
  * db's keep data until explicity deleted. message brokers delete message once delivered to consumer
  * since they delete messages, brokers assume their working set is fairly small (ie queues are short)
  * db's support secondary indexes. brokers support some way of subscribing to a subset of topics matching some pattern

### Multiple Consumers
* Load Balancing:
  * Each meessage is delivered to one of the consumers. useful when message are expensive to process
* Fan Out:
  * Each message is delivered to all of the consumers. Allows several independent consumers to tune in without affecting each other
* Combination:
  * approaches can be combined

### Acknowledgements and redelivery
* Consumers may crash an any time. To ensure that the message is not lost, message brokers use acknowledgments: a client must explicitly it has finished processing the message

### Partition Logs
* even message brokers that durable write messages to disk quickly delete them again after they have been delivered to consumers, because they are built with transient messaging mindset. 
* Databases and filesystems take the opposite approach. Data is normally expected to be permanently stored.
* Why can't we have hybrid? combining the durable storage approach of databases in the low-latency notification facilities or messaging. in comes _log-based message brokers_
* a log is an append-only sequence of records on disk.
* the same structure can be used to implement a message broker
  * producer sends messages by appending to end of log
  * consumer receives messages by reading the log sequentially
  * if consumer reaches end of log, wait for notification of new message append
* to scale to a higher throughput, the log can be partitioned. 
  * different logs can be hosted on different machines
  * each partitions is a separate log that can be read or written independent from other partitions
* messages within a partition are totally ordered. no ordering guarantee across different partitions.

### Logs compared to traditional messaging
* log based approach trivially supports fan-out messaging.
  * several consumers can independently read the log without affecting each other
* In situations where messages may be expensive to process and you want to paralellize on a message by message basis and message order not important
  * JMS/AMQP style of message broker is preferrable.
* In situations with high message throughput, where each message is fast to process and where message ordering is important
  * log-based approach works well

### Consumer offsets:
* broker does not need to track acknowledgments for every single message. it only needs to periodically record consumer offsets
* message broker behaves like a leader database and the consumer like a follower

### Disk space usage
* if you only ever append to a log, eventually you will run out of disk space. 
* the log is actually divided into segments and from time to time old segments are deleted and moved to archive storage
* have to delete old messages, disk acts as a buffer

### When consumer cannot keep up with producers
* if a consumer falls so far behind that the messages it requires are older than what's retained on disk, it will not be able to read those messages
  * broker must drop old messages further back than the size of buffer 
* you can monitor how far a consumer is from head of log and raise alert if it falls back significantly

### Databases and Streams
* the fact that something was written to a database is an event that can be captured, stored and processed. 
  * this suggests that connection between db and streams runs deeper than just physical storage of logs on disk.
* a replication log is essentially a stream of database write events, produced by the leader

### Keeping systems in sync
* As the same or related data appears in several different places, they need to be kept in sync with each other.
  * if item updated in db, it needs to be updated in cache, search index and data warehouse
  * with data warehouses this is usually performed by ETL process.
* if periodic full db dumps are too slow an alternative used is _dual writes_
  * app code explicitly writes to each system when data changes
  * dual writes have some serious problems
    * one write may fail while others succeed
    * ensuring that they both succeed or fail is a case of atomic commits, which are expensive
  
### Change Data Capture
* database replication logs mostly considered an internal implementation detail of database not a public API
* for this reason, difficult to take all changes made in database and replicate them to a different storage technology like a search index, cache or data warehouse.
* _change data capture (CDC)_: the process of observing all data changes written to a db and extracting them in a form that they can be replicated in other systems

### Implementing change data capture
* we can call log consumer _derived data systems_
* CDC makes one db the leader and turns others into followers
* a log based message broker is well suited for transporting the change events from source db since it perserves order of messages.
* like message brokers, change data captures are usually asynchronous

### Initial Snapshot
* if you have a log of all changes that were ever made to a db, you can reconstruct the entire state of db by replaying the log
* in many cases keeping all changes forever would require too much disk space, and replaying it would take too long so the log needs to be truncated.

### Log Compaction
* if you can only keep a limited amount of log history, you need to go through the snapshot process every time you want to add a new dervied system
  * log compaction provides a good alternative
* process is simple:
  * storage engine looks for duplicate keys
  * throws away duplicates
  * keeps only the most recent update for each key
  * compaction and merging process runs in the background
* in a log structured storage engine, an update with a special null value (a tombstone) indicates that a key was deleted
  * this marks it for removal during log compaction
* if CDC system is set up such that 
  * every change has primary key 
  * every update for a key replaces the previous value for that key
  * then it's sufficient to keep just the most recent 
* whenever you want to rebuild a derived data system 
  * start a new consumer from offset 0 of the log compacted topic
  * sequentially scan over all messages in the log
  * log is guaranteed to contain most recent value for each key in the database

### Event Sourcing
* event sourcing involves storing all changes to application state as a log of change events
* biggest difference is that event sourcing applies the idea at a different level of abstraction
  * in change data capture, the application uses the database in a mustable way.
  * can update and delete records at will
  * log of changes is extrated from the database at a low level 
    * ensures the order of writes extracted from the database matches the order in which they were written
* application logic is explicitly built on basis of immutable events that are written to an event log
  * event store is append-only.
  * updates or deletes are discouraged or prohibited.
  * events reflecting things that happen at the app level rather than low level state changes
* event sourcing is a powerful technique for data modeling
  * from app POV more meaningful to records user's actions as immutable events.

### commands and events
* event sourcing is careful to distinguish between events and commands
  * initially a command
  * if succesful validation and acceptance, it becomes an event
  * events are durable and immutable
* consumer of event stream is not allowed to reject an event
* therefore validation of command needs to happen synchronously before it becomes an event

### State, Streams and Immutability
* principle of immutability is what makes event sourcing and CDC powerful
  * batch processing benefitted from immutability bc you could run experimental processing jobs without damaging anything
* key idea is mutable state and append-only log of immutable events do not contradict each other
  * the _changelog_ represents the evolution of state over time
* if you store the changelog durably, by definition the state is reproducible
* transaction logs record all changes made to a database:
* high speed appends are the only way to change the log
* contents in database hold a caching of the latest record values in the logs
* truth is the log

### Advantages of immutable events
* with an append only log of immutable events, it is much easier to dianose what happened and recover from problem
* it may be useful from analytics point of view to know certain events happened even if later canceled by other events

### Deriving several views from the same event log
* if you separating mutable state from the immutable event, 
  * you can derive several different read-oriented representations from the same log of events
* if you want to introduce a feature that presents your existing data in some new way
  * you can use the event log to build a separate read optimized view for the new feature
  * can run alongsized existing systems without modification
