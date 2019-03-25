# KE on MapR Client Container Config Notes 

## Client Container environment: 
* CentOS 7
* KE 3.2
* MapR 6.0.1
* MEP 5.0.2 
* Java-1.8.0-openjdk-devel

Pull out custom docker image
````shell
$ docker pull kyligence/keonmapr:latest
````

## Container Start options
The entire container start script is attached in `mapr_client.sh`, MAPR_CLUSTER , MAPR_CLDB_HOSTS
is suppose to be assigned to the proper value. The script is just a reference for the container 
deployment I think as I made a little option on such as mount the maprfs in the container.

## Configurations need to be made to the container before KE start 
Make a directory on mapr hdfs for KE which will be KE’s working dir, for example we name 
it WORKING_DIR, you can name it whatever you want, this name will also be a config option 
in kylin.properties which we will mention later

````shell
hadoop fs –mkdir /WORKING_DIR
hadoop fs –chown –R mapr:mapr /WORKING_DIR
````

These configuration files need to adapted to the container from the mapr cluster, As I demoed 
before, I just copy those files or configurations from the mapr cluster to the container.

* hive-site.xml (This config file made hive cli works as normal)
* yarn-site.xml (KE need the config of yarn.resourcemanager.address to submit its job to)
* mapred-site.xml(KE need to get jobhistory server information from this)


KE also needs a minimum configuration as follows(its config file location is /opt/ke/conf/kylin.properties):
kylin.env.zookeeper-connect-string=[ZOOKEEPER_HOST1]:[ZOOKEEPER_PORT1],[ ZOOKEEPER_HOST2]:[ZOOKEEPER_PORT2]…

For example: kylin.env.zookeeper-connect-string=172.33.38.53:5181,172.33.38.55:5181

````shell
kylin.env.hdfs-working-dir=maprfs:///WORKING_DIR
````
WORKING_DIR is the director name we created above

kylin.metadata.url= kylin@jdbc,url=jdbc:mysql://[MYSQL_HOST]:[MYSQL_PORT]/kylin?createDatabaseIfNotExist=true,username=[MYSQL_USER],password=[MYSQL_PWD],passwordEncrypted=false,maxActive=40,maxIdle=10,driverClassName=com.mysql.jdbc.Driver

Above is the metadata url for KE to save its metadata
MYSQL_HOST’s server supposed to be an interal or external mysql server which can be accessed from the client container.

````shell
kylin.engine.spark-conf.spark.eventLog.dir=maprfs:///WORKING_DIR/spark-history
kylin.engine.spark-conf.spark.history.fs.logDirectory=maprfs:///WORKING_DIR/spark-history
kap.storage.init-spark-at-starting=true
````
config spark event log dir and history fs log dir, it is advised to use a subdirectory under WORKING_DIR

for a complete example for the configurations above:
````shell
$ echo “kylin.env.zookeeper-connect-string=172.33.38.53:5181,172.33.38.55:5181” >> /opt/ke/conf/kylin.properties
$ echo “kylin.env.hdfs-working-dir=maprfs:///maprwork” >> /opt/ke/conf/kylin.properties
$ echo “kylin.metadata.url=kylin@jdbc,url=jdbc:mysql://0.0.0.0:3306/kylin?createDatabaseIfNotExist=true,username=kylin,password=123456,passwordEncrypted=false,maxActive=40,maxIdle=10,driverClassName=com.mysql.jdbc.Driver” >> /opt/ke/conf/kylin.properties
$ echo “kylin.engine.spark-conf.spark.eventLog.dir=maprfs:///maprwork/spark-history” >> /opt/ke/conf/kylin.properties
$ echo “kylin.engine.spark-conf.spark.history.fs.logDirectory=maprfs:///maprwork/spark-history” >> /opt/ke/conf/kylin.properties
$ echo “kap.storage.init-spark-at-starting=true” >> /opt/ke/conf/kylin.properties
````

For now, KE’s configuration is done. We can run a script to generate a sample project for KE’s demo. The script’s location is `/opt/ke/bin/sample.sh`, the script can be run without KE’s service on, just make sure we are under user of mapr when running it.

````shell
$ su mapr
$ bash /opt/ke/bin/sample.sh
````

after that, we could start KE’s service now, we also use user of mapr to start KE

````shell
$ su mapr
$ bash /opt/ke/bin/kylin.sh start
````

KE’s default listen port is 7070, Login the front page by ADMIN/KYLIN (upper case)
