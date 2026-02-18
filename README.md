# pymongo-api

## Как запустить
Для запуска проекта перейти в директорию
sharding-repl-cache

```shell
docker compose up -d
```

### настроить и наполнить данными mongo DB
```shell
./create_mongo.sh 
```

## Как проверить
Подключаемся к router
docker exec -it mongos_router mongosh --port 27020
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments()
переключаемся на shard1
docker exec -it shard1 mongosh --port 27018
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments()
проверяем статус replica
rs.conf()
переключаемся на shard2
docker exec -it shard2-2 mongosh --port 27023
переключаемся на somedb
use somedb
проверяем
db.helloDoc.countDocuments()
проверяем статус replica
rs.conf()

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs
выполнить дважды вызов http://localhost:8080/helloDoc/users
проверить логи контейнера sharding-repl-cache-pymongo_api 
один вызов должен быть 1 мин
последующие вызовы будут менее 100 м.с.
