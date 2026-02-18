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
docker exec -it shard1 mongosh --port 27018
Инициализируем
rs.initiate(
    {
      _id : "shard1", members: [
        { _id : 0, host : "shard1:27018" },
      ]});

подключаемся к shard2
docker exec -it shard2 mongosh --port 27019
Иницализируем 
rs.initiate(
    {
      _id : "shard2", members: [
        { _id : 1, host : "shard2:27019" }
      ]});

Подключаемся к router
docker exec -it mongos_router mongosh --port 27020
Добавляем в router шарды shard1-1, shard2-1
sh.addShard( "shard1/shard1:27018");
sh.addShard( "shard2/shard2:27019"); 
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
984
переключаемя на shard2
docker exec -it shard2 mongosh --port 27019
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments();
1016

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs