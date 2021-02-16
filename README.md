# Demonstration Oracle CDC Source Connector with Kafka Connect

## Get Oracle CDC Source Connector
Be sure to review license at https://www.confluent.io/hub/confluentinc/kafka-connect-oracle-cdc
```
confluent-hub install confluentinc/kafka-connect-oracle-cdc:latest
```

## Get Oracle Docker
From [Stackoverflow](https://stackoverflow.com/questions/47887403/pull-access-denied-for-container-registry-oracle-com-database-enterprise) ...


- log into https://hub.docker.com/
- search "oracle database"
- click on "Oracle Database Enterprise Edition"
- click on "Proceed to Checkout"
- fill in your contact info on the left, check two boxes under "Developer Tier" on the right, click on "Get Content"


```
docker login --username YourDockerUserName --password-stdin
<<Enter your password>>

docker pull store/oracle/database-enterprise:12.2.0.1
```



## Setup Oracle Docker

```
docker-compose exec oracle /scripts/go_sqlplus.sh /scripts/oracle_setup_docker
```


```
docker-compose exec oracle bash
sqlplus '/ as sysdba' @/scripts/oracle_setup_docker.sql
```


## Connector Configuration 
```
curl -X DELETE localhost:8083/connectors/SimpleOracleCDC_1

curl -s -X GET -H 'Content-Type: application/json' http://localhost:8083/connectors/SimpleOracleCDC_1/status | jq

curl -s -X POST -H 'Content-Type: application/json' --data @config1.json http://localhost:8083/connectors | jq
```

## Check topic
```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic ORCLCDB.C__MYUSER.EMP --from-beginning
```
