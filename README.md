# Spark Thrift Server

[Spark Thrift Server](http://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server) provides JDBC/ODBC access to Spark in DC/OS. 
It allows JDBC/ODBC clients to execute SQL queries over JDBC and ODBC protocols on Apache Spark. 
This is very useful, as it allows one to access and manipulate data at scale by leverage the power of Spark from any tool that supports JDBC/ODBC interface.
This includes clients such as DBeaver and beeline, as well as BI tools such as Tableau and Pentaho.

## WARNING: External Volume

Docker volume used in this Marathon config is specific to DC/OS deployed AWS with CloudFormation. 
If you are using other cloud provider or bare metal make sure to change this configuration.

* EC2 ephemeral storage is mounted as `/var/lib` in DC/OS CloudFormation

## Deploying into DC/OS

Run these commands:

```
> cd ./marathon
> dcos marathon app add < spark-thriftserver.json
```

See [spark-thriftserver.json](https://github.com/NBCUAS/dcos-spark-thrift-server/blob/master/marathon/spark-thriftserver.json)
for full Marathon configuration file

## Dynamic Allocation

This marathon configuration of the Spark Thrift Service is setup to use 
[dynamic allocation](http://spark.apache.org/docs/latest/configuration.html#dynamic-allocation). 
To use dynamic allocation, you need to have [Spark Shuffle Service](https://github.com/NBCUAS/dcos-spark-shuffle-service) runnin. 
You will see the following Spark options set in the marathon configuration file which enable dynamic allocation and properly work with external shuffle service:

| Config                                | Value |
| ------------------------------------- | ----- |
| spark.shuffle.service.enabled         | true  |
| spark.dynamicAllocation.enabled       | true  |
| spark.local.dir	                    | /tmp/spark |
| spark.mesos.executor.docker.volumes	| /var/lib/tmp/spark:/tmp/spark:rw |

## Spark Properties

Some other options are set via environment variables and can be changed as needed:

| Config                                | Value |      
| ------------------------------------- | ----- |
| executor-memory                       | 8g    |
| driver-memory                         | 4g    |
| spark.cores.max                       | 160   |
| spark.executor.cores                  | 2     |
| spark.mesos.extra.cores               | 2     |
| spark.dynamicAllocation.maxExecutors  | 100   |

## Connecting to JDBC/ODBC Server

Once you have the server running, you can connect to it with your favorite client.
Here, we are using [beeline JDBC client](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–CommandLineShell)

```
docker run --rm -ti --net=host --entrypoint=./beeline.sh sutoiku/beeline:hive-1.2.0 -u jdbc:hive2://spark-thriftserver.marathon.mesos:9000/default
```

Please note that we are running in non-secure mode, and so when prompted simply enter the username on your machine and a blank password.