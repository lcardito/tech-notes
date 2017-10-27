# Cassandra Course notes

## Info
Pluralsite course available at [Cassandra](https://app.pluralsight.com/library/courses/cassandra-developers/description)

### What is Cassandra
Decentralised Structured Storage System
Developed by Facebook inbox search, optimised for:
- Lots of writes
- Distributed
- Cluster is P2P system.

### References
- [Snitches docs](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html)
- [Docker image](https://hub.docker.com/_/cassandra/)

### Modules
- [Intro](intro/)
- [Replication & Consistency](repl/)

### CheatSheet

#### nodetool status
`docker exec -ti cass_n1 nodetool status`
