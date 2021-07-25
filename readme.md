# Debezium Samples

This project contains two samples, first one reads data from postgresql and push it to kafka, and the second one is read data from mysql push it to kafka and downstream data from kafka to postgresql. Both of the samples uses the inventory database.

- [Debezium From PostgreSql](#debezium-from-postgresql)
- [Debezium From Mysql To PostgreSql](#debezium-from-mysql-to-postgresql)

## Debezium From PostgreSql

```shell
        +----------------+
        |                |
        |   PostgreSQL   |
        |                |
        +----------------+
                +
                |
                |
                v
+----------------------------------+
|                                  |
|           Kafka Connect          |
|  (Debezium, JDBC connectors)     |
|           Kafka Messages         |
|                                  |
+----------------------------------+
```

- Start the topology as defined in <https://debezium.io/docs/tutorial/>

```shell
export DEBEZIUM_VERSION=1.6
docker-compose -f docker-compose-postgres.yaml up
```

- Start Postgres connector

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-postgres-source.json
```

- Consume messages from a Debezium topic

```shell
docker-compose -f docker-compose-postgres.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.inventory.customers
```

- Make changes on customers table and see results on topic.

- Shut down the cluster

```shell
docker-compose -f docker-compose-postgres.yaml down
```

## Debezium From Mysql To PostgreSql

```shell
                   +-------------+
                   |             |
                   |    MySQL    |
                   |             |
                   +-------------+
                          +
                          |
                          |
                          |
                          v
          +----------------------------------+
          |                                  |
          |           Kafka Connect          |
          |  (Debezium, JDBC connectors)     |
          |                                  |
          +----------------------------------+
                          +
                          |
                          |
                          |
                          v
                 +-----------------+
                 |                 |
                 |    PostgreSQL   |
                 |                 |
                 +-----------------+
```

- Start the topology as defined in <https://debezium.io/docs/tutorial/>

```shell
export DEBEZIUM_VERSION=1.6
docker-compose -f docker-compose-mysql-to-postgres.yaml up
```

- Start Postgres sink connector for customers

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-postgres-sink-customers.json
```

- Start Postgres sink connector for products

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-postgres-sink-products.json
```

- Start Mysql source connector

```shell
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-mysql-source.json
```

- List connectors

```shell
curl -i http://localhost:8083/connectors
```

- List topics

```shell
docker-compose -f docker-compose-postgres.yaml exec kafka /kafka/bin/kafka-topics.sh --list --bootstrap-server kafka:9092
```

- Inspect a topic

```shell
docker-compose -f docker-compose-postgres.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic customers
```

- Make changes on customers@mysql or products@mysql table and see results on customers@postgresql or products@postgresql.

- Shut down the cluster

```shell
docker-compose -f docker-compose-mysql-to-postgres.yaml down
```
