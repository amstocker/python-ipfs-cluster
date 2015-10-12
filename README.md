ipfs-cluster
============

Lately there has been a lot of discussion around implementing a publish/subscribe framework that would facilitate the next wave of dynamic applications built on IPFS.  With a pub-sub protocol, one could deploy an application that communicates with other IPFS nodes and coordinates activities such as data replication and conflict resolution of dynamic datasets.

The goal of ipfs-cluster is to be a light application built on such a pub-sub protocol that can coordinate the replication of dynamic pin-sets on IFPS.  I imagine two processes that together will acheive this goal:

- Replication by Degrees of Separation
    - All new objects added through any one node will be propagated throughout the network via the pub-sub protocol.
    - Each node will listen for new objects advertised on the subscribed channels and pin objects that were created a distance less than or equal to a configurable `MAX_DISTANCE` away.
    - This means that `MAX_DISTANCE` effectively controls the replication factor depending on the topology of the cluster.  For example, in a cluster where the average vertex degree of a node is `K`, and you would like to ensure a replication factor of `R` where `R=K^{MAX_DISTANCE}`, then the minimum acceptable value for `MAX_DISTANCE` would be `ceil(log(R)/log(K))`.  In practice, it seems like `MAX_DISTANCE=2` is sufficient to ensure good replication.
    - This is an effective way to partition the replication of data because it means that every node does not need to be aware of the overall topology of the cluster and also there would be no need for a master node.
- Eventually Consistent Operational History
    - In order to guarantee eventual consistency of the pinset across all nodes, each node will have to keep track of the objects that *the node itself* has added and removed to the pin-set, so that when a neighboring node comes online it would be able to properly replicate the data within its radius of separation.
    - This could be done by each node managing a block-chain structure where it periodically (every minute? ten minutes?) consolidates all new operations (if any) into a block and then updates the head of its blockchain where each block looks like `{[sequence number], [chronological set of operations]}`.  Then new nodes or nodes that have been offline will be able to query neighboring nodes for the head block of their operational history, and get the history back to the sequence number it last left off at.
    - This operational history could *also* be replicated which would allow nodes to still get the recent operational history of nodes that have gone offline.
