# Simple Snitch

Snitch is how Cassandra does request routing.
* **SimpleSnitch**: Single DC, development env
* **GossipingPropertyFileSnitch**: Gossip protocol. A property file in the node tells to the cluster where the node is. It's used to distinguish racks, DCs.
* **PropertyFileSnitch**: Entire topology in one property file
* **CloudSnitches**


Using the simple snitch protocol to run a cluster with two docker nodes.

## docker-compose.yml
```yaml
version: '2'
services:
  node_1:
    container_name: "node_1"
    image: cassandra
  node_2:
    container_name: "node_2"
    image: cassandra
    environment:
      - CASSANDRA_SEEDS=node_1
    links:
      - node_1
```
### Status
`$ docker exec -ti cass_n1 nodetool status`

```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  103.68 KiB  256          100.0%            3ad7d267-8a27-42cb-9c29-35131fd59d34  rack1
UJ  172.17.0.3  15.49 KiB  256          ?                 2995fa35-c94b-473c-a03d-dc13564cc556  rack1
```
 
* **UN**: Up and Normal
* **UN**: Up and Joining
* **Tokens 256**: Even when there's no data this shows the number of virtual nodes (range of token values) 


In the cassandra property file see seed_provider:
```seed_provider:
    # Addresses of hosts that are deemed contact points.
    # Cassandra nodes use this list of hosts to find each other and learn
    # the topology of the ring.  You must change this if you are running
    # multiple nodes!
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          # seeds is actually a comma-delimited list of addresses.
          # Ex: "<ip1>,<ip2>,<ip3>"
          - seeds: "172.17.0.2"
```

[Next](gossiping_snitch/)