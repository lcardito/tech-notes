# Replication & Consistency

## Terminology

* **keyspace** = table space (Replication Strategy)
* **table** = as the SQL tables
* **partition** = primary interaction point when reading or writing data
* **row** = as the row

### SimpleStrategy

```cqlsh
create keyspace pluralsight with replication = {'class': 'SimpleStrategy' , 'replication_factor' : 3}
```

_replication_factor_ = store 3 copies of all the partitions in all the table written to the cluster in this keyspace

### NetworkTopologyStrategy

```cqlsh
create keyspace pluralsight with replication = {'class': 'NetworkTopologyStrategy' , 'DC1': 3, 'DC2': 1}
```

How many copies of the data we want in each DCs.

It used the knowledge of RACKS as well to distribute the data around.

Good read: [Adding nodes to an existing cluster](https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_add_node_to_cluster_t.html)

### Demo 1

In the example with _SimpleStrategy_ each token will store a copy of the data in each node. As the replication strategy says.

When specifying _NetworkTopologyStrategy_ Cassandra will use the RACKS knowledge to distribute the data!

## Read and writes

**Coordinator node**: Node to which an external client talks to to perform an action on the cluster. This node will figure out which other node(s)
needs to deal with the transaction.

### Tunable consistency

How many replica nodes need to ack the transaction before the coordinator node returns a successful response.
This can be set at a transaction level and it's possible values are per table:
- ONE
- TWO
- THREE
- QUORUM
- ALL (each node dealing with a copy of the data)
- ANY (even just the coordinator)

In a DC:

- EACH_QUORUM (quorum in each DC)
- LOCAL_QUORUM (quorum in the DC where the coordinator is)
- LOCAL_ONE ()

When *writing* data to the cluster these values represent the number of nodes from which we expect the ack. The coordinator node
is the one responsible of delegating the writes and collecting the ack. 

**Hinted handoff**: The coordinator will keep trying to store the data in all the replica node if one of them is down.

When *reading* data to the cluster the values represent the number of **digest** comparision we're asking to the cluster.

**Read repair** When one or more of the nodes does not match the digest the coordinator will ask for the full data
trying to figure out which is the correct version and it will write it back to the out of date nodes. This happens in the
background when the lower level of consistency is set. This only happens on the portion of the data that we read.
 
`nodetool repair` -> will run the check on the all the data set
 
**Strong Consistency**
(Write Consistency + Read Consistency) > Replication Factor


### Demo 2

Will create a keyspace with replication_factor = 3. As we've got a 3 node cluster (split in 2 racks) this means that the data will be stored on all 
the replica nodes.

If Cassandra cannot meet the consistency level it will fail on both read and writes.

### Demo Extra

- Launch a simple three node cluster
- Create “pluralsight” keyspace with SimpleStrategy and replication factor of 3
- Create “courses” table in the “pluralsight” keyspace
- Run the “nodetool pausehandoff ” command
- Shutdown one of the nodes with “docker stop”
- Run cqlsh and insert one row into the courses table
- Bring up the node with “docker start” that you just shutdown
- Run cqlsh and set the consistency to “all”
- Set tracing to “on”
- Run a select statement to retrieve the row you inserted above
- Look for “Digest mismatch” in the tracing output
 

