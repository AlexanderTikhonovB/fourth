# pymongo-api

## Как запустить

Внести изменения в compose.yaml

```shell
docker compose up -d
```
Подключаемся к configSrv
docker exec -it configSrv mongosh --port 27017
Иницализируем 
rs.initiate({_id : "config_server", configsvr: true, members: [{ _id : 0, host : "configSrv:27017" }]});

подключаемся к shard1
docker exec -it shard1-1 mongosh --port 27018
Инициализируем
rs.initiate({_id: "shard1", members:[
{_id: 0, host: "shard1-1:27018"},
{_id: 1, host: "shard1-2:27019"},
{_id: 2, host: "shard1-3:27021"}
]})

подключаемся к shard2
docker exec -it shard2-2 mongosh --port 27023
Иницализируем 
rs.initiate({_id: "shard2", members:[
{_id: 0, host: "shard2-1:27022"},
{_id: 1, host: "shard2-2:27023"},
{_id: 2, host: "shard2-3:27024"}
]})

Подключаемся к router
docker exec -it mongos_router mongosh --port 27020
Добавляем в router шарды shard1-1, shard2-1
sh.addShard("shard1/shard1-1:27018,shard1-2:27019,shard1-3:27021")
sh.addShard("shard2/shard2-1:27022,shard2-2:27023,shard2-3:27024") 
создаем базу 
sh.enableSharding("somedb");
и коллекцию helloDoc
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
наполняем данными 
use somedb
for(var i = 0; i < 2000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i})


## Как проверить
Подключаемся к router
docker exec -it mongos_router mongosh --port 27020
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments() 
2000 
переключаемся на shard1
docker exec -it shard1 mongosh --port 27018
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments()
1016
проверяем статус replica
rs.conf()
{
  _id: 'shard1',
  version: 1,
  term: 1,
  members: [
    {
      _id: 0,
      host: 'shard1-1:27018',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 1,
      host: 'shard1-2:27019',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 2,
      host: 'shard1-3:27021',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    }
  ],
  protocolVersion: Long('1'),
  writeConcernMajorityJournalDefault: true,
  settings: {
    chainingAllowed: true,
    heartbeatIntervalMillis: 2000,
    heartbeatTimeoutSecs: 10,
    electionTimeoutMillis: 10000,
    catchUpTimeoutMillis: -1,
    catchUpTakeoverDelayMillis: 30000,
    getLastErrorModes: {},
    getLastErrorDefaults: { w: 1, wtimeout: 0 },
    replicaSetId: ObjectId('6994403d46c4a19152908222')
  }
}
переключаемя на shard2
docker exec -it shard2-1 mongosh --port 27022
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments()
984
проверяем статус replica
rs.conf()
{
  _id: 'shard2',
  version: 1,
  term: 1,
  members: [
    {
      _id: 0,
      host: 'shard2-1:27022',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 1,
      host: 'shard2-2:27023',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 2,
      host: 'shard2-3:27024',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    }
  ],
  protocolVersion: Long('1'),
  writeConcernMajorityJournalDefault: true,
  settings: {
    chainingAllowed: true,
    heartbeatIntervalMillis: 2000,
    heartbeatTimeoutSecs: 10,
    electionTimeoutMillis: 10000,
    catchUpTimeoutMillis: -1,
    catchUpTakeoverDelayMillis: 30000,
    getLastErrorModes: {},
    getLastErrorDefaults: { w: 1, wtimeout: 0 },
    replicaSetId: ObjectId('6994401bd2f654ec9a90a9d6')
  }
}

### подключение cache
создать директорию redis в корне
скопировать файл redis.conf из прилагаемого описания кэширования
Redis - внести изменение в конфигурационный файл redis.conf
cluster-enabled no

### НЕ ВЫПОЛНЯЛОСЬ в связи с необходимостью модификации приложения
Подключиться к docker redis
docker exec -it redis_1 bash
Создать кластер
redis-cli --cluster create redis_1:6379 redis_2:6379 redis_3:6379 redis_4:6379 redis_5:6379 redis_6:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica redis_5:6379 to redis_1:6379
Adding replica redis_6:6379 to redis_2:6379
Adding replica redis_4:6379 to redis_3:6379
M: a6223a8dfa0521a214472087b16a49a818183d71 redis_1:6379
   slots:[0-5460] (5461 slots) master
M: 4c0592599bdabd2b406a36406667b9092170d0e4 redis_2:6379
   slots:[5461-10922] (5462 slots) master
M: 3e75f7e00e79938f192599a028c6ef5d5b5eb139 redis_3:6379
   slots:[10923-16383] (5461 slots) master
S: 2d9772514b8ae312819a6cd7bd8bdf4624673437 redis_4:6379
   replicates 3e75f7e00e79938f192599a028c6ef5d5b5eb139
S: 74a36c314bce94debf29e09704b96f21b0440318 redis_5:6379
   replicates a6223a8dfa0521a214472087b16a49a818183d71
S: 601eeec77d3decc5f85c984787003c7b3c1f33bc redis_6:6379
   replicates 4c0592599bdabd2b406a36406667b9092170d0e4
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
>>> Performing Cluster Check (using node redis_1:6379)
M: a6223a8dfa0521a214472087b16a49a818183d71 redis_1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 4c0592599bdabd2b406a36406667b9092170d0e4 173.17.0.11:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 2d9772514b8ae312819a6cd7bd8bdf4624673437 173.17.0.8:6379
   slots: (0 slots) slave
   replicates 3e75f7e00e79938f192599a028c6ef5d5b5eb139
S: 74a36c314bce94debf29e09704b96f21b0440318 173.17.0.3:6379
   slots: (0 slots) slave
   replicates a6223a8dfa0521a214472087b16a49a818183d71
S: 601eeec77d3decc5f85c984787003c7b3c1f33bc 173.17.0.13:6379
   slots: (0 slots) slave
   replicates 4c0592599bdabd2b406a36406667b9092170d0e4
M: 3e75f7e00e79938f192599a028c6ef5d5b5eb139 173.17.0.15:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
redis-cli cluster nodes
601eeec77d3decc5f85c984787003c7b3c1f33bc :0@0 slave,noaddr 4c0592599bdabd2b406a36406667b9092170d0e4 1771346518085 1771346518019 2 disconnected
74a36c314bce94debf29e09704b96f21b0440318 173.17.0.4:6379@16379 slave a6223a8dfa0521a214472087b16a49a818183d71 0 1771346808194 1 connected
2d9772514b8ae312819a6cd7bd8bdf4624673437 173.17.0.11:6379@16379 slave 3e75f7e00e79938f192599a028c6ef5d5b5eb139 0 1771346807588 3 connected
3e75f7e00e79938f192599a028c6ef5d5b5eb139 173.17.0.3:6379@16379 master - 0 1771346808000 3 connected 10923-16383
a6223a8dfa0521a214472087b16a49a818183d71 173.17.0.13:6379@16379 myself,master - 0 0 1 connected 0-5460
4c0592599bdabd2b406a36406667b9092170d0e4 173.17.0.10:6379@16379 master - 0 1771346807184 2 connected 5461-10922

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs
