# iota-iot-analysis
(This repo is working progress)

## CONFLUENT

Self Managed Platform -> https://www.confluent.io/download

Create Topic: iota-gateway with value schema: 

```json
{
  "type": "record",
  "namespace": "org.fabryprog.iota.kafka.pojo",
  "name": "Transaction",
  "version": "1",
  "fields": [
    { "name": "hash", "type": "string", "doc": "Transaction Hash" },
    { "name": "address", "type": "string", "doc": "Address" },
    { "name": "value", "type": "long", "doc": "Transaction value" },
    { "name": "tag", "type": "string", "doc": "TAG" },
    { "name": "timestamp", "type": "long", "doc": "Timestamp" },
    { "name": "payload", "type": "string", "doc": "Payload" }
  ]
}
```

## GATEWAY

IOTA to KAFKA -> https://github.com/Fabryprog/iota-kafka-gateway

Settings:
 - topic -> "iota-gateway"
 - iota zmq node -> "tcp://ultranode.iotatoken.nl:5556"

## KSQL

Create STREAM **IOTA**

```sql
CREATE STREAM IOTA
 WITH (KAFKA_TOPIC='iota-gateway',
       VALUE_FORMAT='AVRO',
       KEY='hash');
```

Create STREAM **IOTA_JSON** to filter JSON data

```sql
CREATE STREAM IOTA_JSON
 AS SELECT * FROM IOTA WHERE SUBSTRING(payload,1,1) = '{'
```

Analisys: group by first 20 characters

```sql
SELECT SUBSTRING(payload,1,20), COUNT(*) FROM IOTA_JSON GROUP BY SUBSTRING(payload,1,20)
EMIT CHANGES;
```
Look at the output, today there are 4 types of JSON with following first JSON element:
 - id
 - v
 - device
 - type
 - channel
 - TanglePigeon
 
N.B. I known! I known! it is very simplistic method. Do you have another idea? :-)

CREATE A STREAM for every JSON flow

```sql
CREATE STREAM IOTA_JSON_ID
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.id') IS NOT NULL);

CREATE STREAM IOTA_JSON_V
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.v') IS NOT NULL);

CREATE STREAM IOTA_JSON_DEVICE
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.device') IS NOT NULL);

CREATE STREAM IOTA_JSON_TYPE
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.type') IS NOT NULL);

CREATE STREAM IOTA_JSON_CHANNEL
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.channel') IS NOT NULL);

CREATE STREAM IOTA_JSON_TANGLEPIGEON
AS SELECT * FROM IOTA_JSON
WHERE (EXTRACTJSONFIELD(payload, '$.TanglePigeon') IS NOT NULL);

```
