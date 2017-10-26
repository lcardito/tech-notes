# Gossiping Snitch
Gossip protocol. A property file in the node tells to the cluster where the node is. It's used to distinguish racks, DCs.

## docker-compose.yml
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
### Status
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

[Next]()