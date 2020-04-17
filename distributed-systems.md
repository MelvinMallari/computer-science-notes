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