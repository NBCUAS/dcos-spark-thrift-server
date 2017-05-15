# Spark Thrift Server

[Spark Thrift Server](http://spark.apache.org/docs/latest/sql-programming-guide.html#running-the-thrift-jdbcodbc-server) provides JDBC/ODBC access to Spark in DC/OS. This is very useful, as it allows you to access and manipulate your data by leverage the power of Spark from any tool that supports JDBC/ODBC interface.
This includes clients, such as DBEaver, beeline, as well as BI tools, such as Tableau, Pentaho, etc.

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

See [spark-thriftserver.json](https://github.com/NBCUAS/dcos-spark-thrift-server/blob/master/marathon/spark-thriftserver.json)
for full Marathon configuration file

## Spark Properties

The marathon configuration of the Spark Thrift Service is setup to use [dynamic allocation](http://spark.apache.org/docs/latest/configuration.html#dynamic-allocation). You will, therefore, see the following options set:

| Config                                | Value |
| ------------------------------------- |-------|
| spark.shuffle.service.enabled         | true  |
| spark.dynamicAllocation.enabled       | true  |
| spark.local.dir	                    | /tmp/spark |
| spark.mesos.executor.docker.volumes	| /var/lib/tmp/spark:/tmp/spark:rw |

Some other options are set via environment variables and can be changed as needed:

| Config                                | Value |      
| ------------------------------------- |-------|
| executor-memory                       | 8g    |
| driver-memory                         | 4g    |
| spark.cores.max                       | 160   |
| spark.executor.cores                  | 2     |
| spark.mesos.extra.cores               | 2     |
| spark.dynamicAllocation.maxExecutors  | 100   |

## Connecting to JDBC/ODBC Server

Once you have the server running, you can connect to it with your favorite client. Here, we are using [beeline JDBC client](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–CommandLineShell)
```
docker run --rm -ti --net=host --entrypoint=./beeline.sh sutoiku/beeline:hive-1.2.0 -u jdbc:hive2://spark-thriftserver.marathon.mesos:9000/default
```
