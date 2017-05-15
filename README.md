# Spark Thrift Server

JDBC/ODBC access to Spark in DC/OS

## WARNING: External Volume

Docker volume used in this Marathon config is specific to DC/OS deployed AWS with CloudFormation. If you are using
other cloud provider or bare metal make sure to change this configuration.

* EC2 ephemeral storage is mounted as `/var/lib` in DC/OS CloudFormation

## Deploying into DC/OS

Run these commands:

```
> cd ./marathon
> dcos marathon app add < spark-thriftserver.json
```

## Dynamic allocation

The marathon configuration of the Spark Thrift Service is setup to use dynamic allocation. You will, therefore, see the following options set:

| Config                                | Value |
| ------------------------------------- |-------|
| spark.shuffle.service.enabled         | true  |
| spark.dynamicAllocation.enabled       | true  |
| spark.local.dir	                    | /tmp/spark |
| spark.mesos.executor.docker.volumes	| /var/lib/tmp/spark:/tmp/spark:rw |

The following options are set via environment variables and can be changed as needed:

| Config                                | Value |      
| ------------------------------------- |-------|
| executor-memory                       | 8g    |
| driver-memory                         | 4g    |
| spark.cores.max                       | 160   |
| spark.executor.cores                  | 2     |
| spark.mesos.extra.cores               | 2     |
| spark.dynamicAllocation.maxExecutors  | 100   |

## Example

```
docker run --rm -ti --net=host --entrypoint=./beeline.sh sutoiku/beeline:hive-1.2.0 -u jdbc:hive2://spark-thriftserver.marathon.mesos:9000/default
```

```
CREATE TABLE experian
  USING org.apache.spark.sql.parquet
  OPTIONS (path "s3a://nbcuas-audience-graph/experian_data/201704240600");

CREATE TABLE polk
  USING org.apache.spark.sql.parquet
  OPTIONS (path "s3a://nbcuas-audience-graph/polk_data/201704240600");

CREATE TABLE latest_classifier
  USING org.apache.spark.sql.parquet
  OPTIONS (path "s3a://nbcuas-audience-graph/classifier_data/201704210645");

SHOW TABLES;

SELECT * from latest_classifier LIMIT 10;

SELECT count(1) FROM experian e, polk p
  WHERE  ((e.Person1Surname = p.HHLDNM) OR (e.Person2Surname = p.HHLDNM))
    AND e.ZipCode = p.ZIPCD
    AND e.Zip4 = p.ZPPLS4
    AND concat(e.PrimaryAddress, ' ', e.SecondaryAddress) = p.ADDRESS;
    
```
