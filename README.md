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
Once the Oracle database is running, we need to turn on ARCHIVELOG mode, create some users, and establish permissions

First, ensure the database looks like it's running (`docker-compose logs -f oracle`) and then run the following

```
docker-compose exec oracle /scripts/go_sqlplus.sh /scripts/oracle_setup_docker
```


## Connector Configuration 
Establish the `SimpleOracleCDC` connector
```
curl -s -X POST -H 'Content-Type: application/json' --data @SimpleOracleCDC.json http://localhost:8083/connectors | jq
```

Check the status of the connector. You may need to wait a while for the status to show up
```
curl -s -X GET -H 'Content-Type: application/json' http://localhost:8083/connectors/SimpleOracleCDC/status | jq
```



## Check topic
```
kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic ORCLCDB.C__MYUSER.EMP --from-beginning
```


# Insert, update and delete some data

Run `docker-compose exec oracle /scripts/go_sqlplus.sh` followed by this SQL

```
insert into C##MYUSER.emp (name) values ('Dale');
insert into C##MYUSER.emp (name) values ('Emma');
update C##MYUSER.emp set name = 'Robert' where name = 'Bob';
delete C##MYUSER.emp where name = 'Jane';
commit;
exit
```

# DDL 
Run `docker-compose exec oracle /scripts/go_sqlplus.sh` followed by this SQL

```
ALTER TABLE C##MYUSER.EMP ADD (SURNAME VARCHAR2(100));

insert into C##MYUSER.emp (name, surname) values ('Mickey', 'Mouse');
commit;

```

## Schema mutation
Let's see what schemas we have registered now
```console
curl -s -X GET http://localhost:8081/subjects/ORCLCDB.C__MYUSER.EMP-value/versions

curl -s -X GET http://localhost:8081/subjects/ORCLCDB.C__MYUSER.EMP-value/versions/1 | jq '.'

curl -s -X GET http://localhost:8081/subjects/ORCLCDB.C__MYUSER.EMP-value/versions/2 | jq '.'

or you can also use:

curl -s -X GET http://localhost:8081/subjects/ORCLCDB.C__MYUSER.EMP-value/versions/2 | jq -r .schema | jq .
```

Note, schema version 2 has this addition

```
    {
      "name": "SURNAME",
      "type": [
        "null",
        "string"
      ],
      "default": null
    }
```


## Connector Delete Configuration 

If something goes wrong, you can delete the connector like this
```
curl -X DELETE localhost:8083/connectors/SimpleOracleCDC
```


