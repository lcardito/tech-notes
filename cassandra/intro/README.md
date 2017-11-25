# Snitch

Snitch is how Cassandra does request routing.

> A snitch determines which datacenters and racks nodes belong to. 
> They inform Cassandra about the network topology so that requests are routed efficiently and allows Cassandra to distribute replicas by grouping machines into datacenters and racks

* [**SimpleSnitch**](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchSimple.html): Single DC, development env
* [**GossipingPropertyFileSnitch**](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archsnitchGossipPF.html): Gossip protocol. A property file in the node tells to the cluster where the node is. It's used to distinguish racks, DCs.
* **PropertyFileSnitch**: Entire topology in one property file
* **CloudSnitches**

## Simple Snitch

Using the simple snitch protocol to run a cluster with two docker nodes.

### docker-compose.yml
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
#### Status
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

## Gossiping Snitch
Gossip protocol. A property file in the node tells to the cluster where the node is. It's used to distinguish racks, DCs.

### docker-compose.yml
```yaml
version: '2'
services:
  node_1:
    container_name: "node_1"
    image: cassandra
    environment:
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - CASSANDRA_DC=DC1
      - CASSANDRA_RACK=RC1
  node_2:
    container_name: "node_2"
    image: cassandra
    environment:
      - CASSANDRA_SEEDS=node_1
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - CASSANDRA_DC=DC2
      - CASSANDRA_RACK=RC1
    links:
      - node_1
```
#### Status
`$ docker exec -ti cass_n1 nodetool status`

```
Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.20.0.2  108.65 KiB  256          100.0%            955986e1-933e-44a0-a448-1664d40ca6d3  RC1
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.20.0.3  74.67 KiB  256          100.0%            06423b60-7e83-4bd1-bd7d-c31e531caaa8  RC1```
``` 

The `/etc/cassandra/cassandra.yml` config will reflect this configuration:

```
endpoint_snitch: GossipingPropertyFileSnitch
```

Also in `/etc/cassandra/cassandra-rackdc.properties`

```
dc= DC2
rack= RC1
```

Each logical DC has it's own complete range of tokens allocated to the vNodes provided by the node in that DC.
