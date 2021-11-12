# Stream MongoDB to MySQLUsing Kafka Connect

## Building Containers

```bash
docker-compose -p kuc build
```

```bash
docker-compose -p kuc up -d
```

## MongoDB
First, let us access MongoDB using Robo 3T which available for download in https://robomongo.org/download

Create new database called `local`. Inside `local` database, create a new collection called `my_collection`.

On `my_collection` collection, create new document. For example:
```json
{
    "_id": ObjectID(),
    "name": "John Doe",
    "gender": "male",
    "email": "john.doe@example.com"
}
```

## Kafka
For easiness, we are using web based UI for Kafka called [AKHQ](https://github.com/tchiotludo/akhq). It is accessible on http://localhost:8080

## MongoDB CDC

Create new source connector using this command:
```bash
curl -X POST \
  http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "cdc-mongo-001",
    "config": {
        "connector.class" : "io.debezium.connector.mongodb.MongoDbConnector",
        "tasks.max" : "1",
        "mongodb.hosts" : "my-replica-set/mongo1:30001,mongo1:30003,mongo1:30003",
        "mongodb.name" : "mongo",
        "database.include" : "example",
        "database.history.kafka.bootstrap.servers" : "kafka:9092"
    }
}
'
```

```bash
curl -X POST \
  http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "cdc-mongo-002",
    "config": {
        "connector.class" : "io.debezium.connector.mongodb.MongoDbConnector",
        "tasks.max" : "1",
        "mongodb.hosts" : "my-replica-set/mongo1:30001,mongo1:30003,mongo1:30003",
        "mongodb.name" : "mongo",
        "database.include" : "example",
        "database.history.kafka.bootstrap.servers" : "kafka:9092",
        "transforms": "route",
        "transforms.route.type" : "org.apache.kafka.connect.transforms.RegexRouter",
        "transforms.route.regex" : "([^.]+)\\.([^.]+)\\.([^.]+)",
        "transforms.route.replacement" : "$3"
    }
}
'
```

Create JDBC sink
```bash
curl -X POST \
  http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  -d '{
    "name" : "jdbc-sink-001",
    "config" : {
        "connector.class" : "io.confluent.connect.jdbc.JdbcSinkConnector",
        "tasks.max" : "1",
        "topics" : "mongo.example.users",
        "connection.url" : "jdbc:mysql://mysql:3306/example",
        "connection.user": "root",
        "connection.password": "example",
        "auto.create" : "true",
        "auto.evolve" : "true",
        "insert.mode" : "upsert",
        "delete.enabled": "true",
        "pk.fields" : "id",
        "pk.mode": "record_key",
        "transforms": "route,mongoflatten",
        "transforms.mongoflatten.type" : "io.debezium.connector.mongodb.transforms.ExtractNewDocumentState",
        "transforms.mongoflatten.drop.tombstones": "false",
        "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
        "transforms.route.replacement": "$3",
        "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter"
    }
}
'
```