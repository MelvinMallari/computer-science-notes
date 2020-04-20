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

