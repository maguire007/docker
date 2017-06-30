
## Sentinal Redis Cluster deployed to Rancher Cattle for spring session persistence 

## Setup 
docker build -t yourrepo/redis:sentinel .


## Example compose with rancher cattle
```
version: '2'
services:
  slave:
    image: redis:3
    links:
    - master:redis-master
    command:
    - redis-server
    - --slaveof
    - redis-master
    - '6379'
    labels:
      io.rancher.scheduler.affinity:host_label: where-your-slaves-should-go=true
  sentinel:
    image: yourrepo/redis:sentinel
    environment:
      SENTINEL_DOWN_AFTER: '5000'
      SENTINEL_FAILOVER: '5000'
    links:
    - master:redis-master
    - slave:slave
    ports:
    - 26379:26379/tcp
    labels:
      io.rancher.scheduler.affinity:host_label: where-your-sentinalnodes-should-go=true
  master:
    image: redis:3
    labels:
      io.rancher.scheduler.affinity:host_label: where-your-masternodes-should-go=true



# Example rancher-compose to support several failure scenarios across multiple hosts
```

version: '2'
services:
  slave:
    scale: 2
    start_on_create: true
  sentinel:
    scale: 4
    start_on_create: true
  master:
    scale: 1
    start_on_create: true




## References:

Based on https://github.com/AliyunContainerService/redis-cluster

Annotated sentinel.conf
http://download.redis.io/redis-stable/sentinel.conf

Spring Session with Redis
http://docs.spring.io/spring-data/redis/docs/current/reference/html/

https://redis.io/topics/sentinel


Few Redis Queries to test the cluster with:

Lists all keys 
redis-cli
keys * 


Delete all keys
redis-cli
keys '*' | xargs redis-cli del

# Cluster Survives several Tested Scenarios of Failure

Slave container stopped
Sentinal Container stopped
Master Container stopped




