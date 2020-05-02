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
