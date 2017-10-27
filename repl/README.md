# Replication & Consistency

## Terminology

* **keyspace** = table space (Replication Strategy)
* **table** = as the SQL tables
* **partition** = primary interaction point when reading or writing data
* **row** = as the row

### SimpleStrategy

```cql
create keyspace pluralsight with replication = 
    {'class': 'SimpleStrategy' , 'replication_factor' : 3}
```

_replication_factor_ = store 3 copies of all the partitions in all the table written to the cluster in this keyspace

### NetworkTopologyStrategy

```cql
create keyspace pluralsight with replication = 
    {'class': 'NetworkTopologyStrategy' , 'DC1': 3, 'DC2': 1}
```

How many copies of the data we want in each DCs.

It used the knowledge of RACKS as well to distribute the data around.

Good read: [Adding nodes to an existing cluster](https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_add_node_to_cluster_t.html)

### Demo

In the example with _SimpleStrategy_ each token will store a copy of the data in each node. As the replication strategy says.

When specifying _NetworkTopologyStrategy_ Cassandra will use the RACKS knowledge to distribute the data!