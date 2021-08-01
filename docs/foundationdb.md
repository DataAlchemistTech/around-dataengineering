# Foundation DB

![image](https://user-images.githubusercontent.com/7579608/127642075-02fcc50f-789f-457d-8f4d-1abb450bb394.png)


FoundationDB is an open source transactional key value store which combines the felxibilty of flexibility and scalability of NoSQL architectures with the 
power of __ACID transactions__. It offers a minimal and carefully chosen feature set, which has enabled a range of disparate systems to be built as layers on top
FoundationDB adopts an unbundled architecture that decouples an
- _in-memory transaction management system_
-  _distributed storage system_ , (SS) is used for storing data and servicing reads
- _built-in distributed configuration system_ 

Each sub-system can be independently scaled. FoundationDB uniquely integrates a deterministic simulation framework, used to test every new feature of the system.
FoundationDB’s focus on the “lower half” of a database, leaving the rest to its “layers”—stateless applications developed on top to provide various data models and other capabilities

## Architecture

Core Stateful Components -
1. __Coordinators__, to hold metatdata. Based on Paxos
2. __Transaction Logs__, Distributed WAL responsible for accepting commits and writes to the system. Write Once Read Never datastructure & mutations are only being held
 transeantly. As data comes in, its appended into the file and TTL is once the Storage Server have the data
3. __Storage Server__, Individual unit of KV system which can coordinate with eachother and becomes a single big Key Value Store. Each KV holds data for long term coming from Transaction Logs and serving read requests. _Each Storage Server is linked with designated set of Transaction Logs to add __Efficiency__ in the system_

![image](https://user-images.githubusercontent.com/7579608/127644956-561f3a66-fdfe-4f3b-8493-dee97a39495d.png)

### Cluster Controller

Cluster Controller is a leader elected by the Coordinators, to organize all the processes in the cluster. The work and communication is similar to Quoram.
- The ClusterController monitors all servers in the cluster and recruits three singleton processes ie _Sequencer, DataDistributor,
and Ratekeeper_ , which are re-recruited if they fail or crash. 
- _Sequencer_ assigns read and commit versions to transactions. 
- _DataDistributor_ is responsible for monitoring failures and balancing data among StorageServers. 
- _Ratekeeper_ provides overload protection for the cluster

When any process starts up, it first interact with the Coordinator to find the Cluster Controller and then the process updates all its info to the controller. Then the cluster controller interacts with all of the Workers and start assigning different roles like become a log server or SS etc. CC as well responsible for __Failure Monitoring__

### Data Plan

Transaction Management provides transaction processing and consists of a stateless processes
* _Sequencer_, assigns a read version and a commit version to each transaction
* _Proxies_, MVCC read versions to clients and orchestrate transaction commits.
* _Resolvers_, check for conflicts between transactions 

__LogServers__ act as replicated, sharded, distributed persistent queues, where each queue stores WAL data for a StorageServer.  

The SS consists of a number of __StorageServers__ for serving client reads, where each StorageServer stores a set of data shards, i.e., contiguous key ranges. StorageServers are the majority of processes in the system, and together they form a _distributed B-tree_

## References

- Paper https://www.foundationdb.org/files/fdb-paper.pdf
- Technical Overview https://www.youtube.com/watch?v=EMwhsGsxfPU
- Paper Overview https://www.youtube.com/watch?v=A6Ob1lebIzQ
-  