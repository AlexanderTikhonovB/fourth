# pymongo-api

## Как запустить

Внести изменения в compose.yaml
Добавить приложение в создаваемую подсеть

```shell
docker compose up -d
```
Подключаемся к configSrv
docker exec -it configSrv mongosh --port 27017
Иницализируем 
rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);

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
и коллекцию hashed
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

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs