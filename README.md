# Snippets
## _I store here useful snippets that im probably going to forget_
 ---
 
## Kafka Docker

 - Kafka,schema-registry,zookeper,debezium
```yml
---
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:6.2.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-kafka:6.2.0
    container_name: broker
    ports:
    # To learn about configuring Kafka for access across networks see
    # https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://broker:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

  schema-registry:
    image: confluentinc/cp-schema-registry:6.2.0
    container_name: schema-registry
    ports:
      - "8081:8081"
    depends_on:
      - broker
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: broker:29092

  kafka-connect:
    image: confluentinc/cp-kafka-connect-base:6.2.0
    container_name: kafka-connect
    depends_on:
      - broker
    ports:
      - 8083:8083
    environment:
      CONNECT_BOOTSTRAP_SERVERS: "broker:29092"
      REST_PORT: "8083"
      CONNECT_GROUP_ID: kafka-connect
      CONNECT_CONFIG_STORAGE_TOPIC: _connect-configs
      CONNECT_OFFSET_STORAGE_TOPIC: _connect-offsets
      CONNECT_STATUS_STORAGE_TOPIC: _connect-status
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.storage.StringConverter
      CONNECT_VALUE_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: 'http://schema-registry:8081'
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_LOG4J_APPENDER_STDOUT_LAYOUT_CONVERSIONPATTERN: "[%d] %p %X{connector.context}%m (%c:%L)%n"
      CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect"
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
    # -----------------------
      CONNECT_PLUGIN_PATH: /usr/share/java,/usr/share/confluent-hub-components,/data/connect-jars
      command: >
        sh -c " echo 'Installing Connector' &&
                confluent-hub install --no-prompt debezium/debezium-connector-sqlserver:1.6.0 &&
                confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:11.1.2 &&
                confluent-hub install --no-prompt neo4j/kafka-connect-neo4j:1.0.9 &&
                echo 'Launching Kafka Connect worker' &&
                /etc/confluent/docker/run && sleep infinity " 
  
  debezium-ui:
    image: debezium/debezium-ui:1.7
    container_name: debezium-ui
    depends_on:
      - kafka-connect
    ports:
      - 8080:8080
    environment:
      KAFKA_CONNECT_URI: http://kafka-connect:8083
```

## SQL Server

- Enable cdc on db
```sql
USE DbTest;  
GO  
EXECUTE sys.sp_cdc_enable_db;  
GO
```

- Enable cdc on table
```sql
USE Rhdb;

EXECUTE sys.sp_cdc_enable_table
    @source_schema = N'Rhhh',
    @source_name = N'Employee',
    @supports_net_changes = 1,
   	@role_name = N'cdc_admin';
```

- List all cdc active tables
```sql
USE TestDb;

GO
SELECT 
    s.name AS Schema_Name, 
    tb.name AS Table_Name, 
    tb.object_id, tb.type, 
    tb.type_desc, 
    tb.is_tracked_by_cdc
FROM sys.tables tb
INNER JOIN sys.schemas s on s.schema_id = tb.schema_id
WHERE tb.is_tracked_by_cdc = 1
```

- Drop all database connections
```sql
DECLARE @DatabaseName nvarchar(50)
SET @DatabaseName = N'Db'

DECLARE @SQL varchar(max)

SELECT @SQL = COALESCE(@SQL,'') + 'Kill ' + Convert(varchar, SPId) + ';'
FROM MASTER..SysProcesses
WHERE DBId = DB_ID(@DatabaseName) AND SPId <> @@SPId

EXEC(@SQL)
```
