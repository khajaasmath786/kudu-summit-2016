# kudu-summit-2016
Kudu and Spark latest and greatest features as shown at FCE Summit 2016

# Comments

Writing to kudu: Docs on DataFrames describe saving to persistent tables typically being done with an api call like saveAsTable(). Kudu presently has a kuduContext method called writeRows() that will write to a kudu table.

# Code snippets

## StructType schemas

We often need to create StructType schemas if we want to define a new Kudu table.  Now, to do this, we can do a quick and dirty approach as the following

```java
hiveContext.sql("create table person (name string, age int, color string)")
val emptyDataFrame = hiveContext.sql("select * from person limit 0")
val schema         = emptyDataFrame.schema
```

Piggy back on `hiveContext` to create a person table, run a select on that table with 0 rows so you get an empty data frame, and then from there, you can fetch the schema.

A more complete way of doing this, so that you not only specify column name and type, but provide nullability, may be to do the following:

```java
import  org.apache.spark.sql.types._

val schema = StructType(StructField("name" , StringType , false) ::
                        StructField("age"  , IntegerType, true) ::
                        StructField("color", StringType , true) :: Nil)
```

# Kudu install notes - using Cloudera manager

Always refer to latest documentation.

http://www.cloudera.com/documentation/betas/kudu/0-9-0/topics/kudu_installation.html


CSD location is here: http://archive.cloudera.com/beta/kudu/csd/
Impala_kudu parcels: http://archive.cloudera.com/beta/impala-kudu/parcels/latest/


## Shortcut Commands

```
# As root on CM host
ssh -i ~/pemkeys/fce.pem ec2-user@10.7...
sudo su -
cd /opt/cloudera/csd
yum -y install wget
wget http://archive.cloudera.com/beta/kudu/csd/KUDU-0.9.1.jar
cd /opt/cloudera/parcel-repo/
wget http://archive.cloudera.com/beta/impala-kudu/parcels/latest/IMPALA_KUDU-2.7.0-1.cdh5.9.0.p0.23-el7.parcel.sha1
wget http://archive.cloudera.com/beta/impala-kudu/parcels/latest/IMPALA_KUDU-2.7.0-1.cdh5.9.0.p0.23-el7.parcel
chown cloudera-scm:cloudera-scm IMPALA_KUDU*
mv IMPALA_KUDU-2.7.0-1.cdh5.9.0.p0.23-el7.parcel.sha1 IMPALA_KUDU-2.7.0-1.cdh5.9.0.p0.23-el7.parcel.sha

service cloudera-scm-server restart

# In CM,
# Parcels->Kudu->Download, Distribute, Activate
# Actions->Add Service->Kudu

/data0/kudu/wal
/data0/kudu/data
/data1/kudu/data

# Click Parcels, IMPALA_KUDU, Distribute, Activate
# Actions->Add Service->Impala
Impala Service Environment Advanced Configuration Snippet (Safety Valve)
IMPALA_KUDU=1

# Start up all the services
# As hdfs user
hdfs dfs -mkdir /user/ec2-user
hdfs dfs -chown ec2-user:ec2-user /user/ec2-user
hdfs dfs -ls /user/

# As ec2-user
impala-shell -i 10.13.5.108:21000
select if(version() like '%KUDU%', "all set to go!", "check your configs") as s;
```

## rsync to servers
Copy files from dev environment to gateway nodes with rsync.

```
rsync -varlte "ssh -i /Users/mladen/pemkeys/fce.pem" * ec2-user@10.13.5.78:kudu-summit-2016
```

## Impala samples

```
impala-shell -i 10.13.5.108:21000 -f util/spark_kudu_tbl.sql
impala-shell -i 10.13.5.108:21000 -q "select * from spark_kudu_tbl"
```

## Submit job

In the primary directory, submit a spark job

```
spark-submit --class com.cloudera.summit.kudu.KuduSparkSample --master yarn-client --num-executors 6 --executor-cores 2 --executor-memory 1G --jars jars/kudu-client-0.9.1.jar,jars/kudu-spark_2.10-0.9.1.jar target/KuduSparkSample-1.0.jar
```
