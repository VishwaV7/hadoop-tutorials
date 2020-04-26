# Sample and Practice code from the training session


# Environment setup 

If you are using it for first time, then please install 

1. Setup your local PC for git
2. Clone the Repo: 

``` 
git clone https://github.com/becloudready/hadoop-tutorials.git

``` 

# Instructions for Module 4
Gitbash
Once cloned the repo, make sure the code works by doing a local map reduce

```
echo "foo foo quux labs foo bar quux" | ./mapper.py | sort -k1,1 | ./reducer.py
bar	1
foo	3
labs	1
quux	2
```

Download the sample file 

```
wget http://www.gutenberg.org/files/5000/5000-8.txt
```

Upload it to HDFS

```
hdfs dfs -mkdir /user/ssh
hdfs dfs -copyFromLocal 5000-8.txt /user/ssh
```

Run the Map reduce job [ Azure HDInsight ]

```
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming-2.7.3.2.6.5.3015-8.jar \
-file /home/sshuser/hadoop-tutorials/Module4/mapper.py    -mapper /home/sshuser/hadoop-tutorials/Module4/mapper.py \
-file /home/sshuser/hadoop-tutorials/Module4/reducer.py   -reducer /home/sshuser/hadoop-tutorials/Module4/reducer.py \
-input /user/ssh/5000-8.txt -output /user/ssh/output

```
Run the MapReduce job [AWS EMR ]

```
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming-2.8.4-amzn-1.jar \
-file /home/hadoop/mapper.py    -mapper /home/hadoop/mapper.py \
-file /home/hadoop/reducer.py   -reducer /home/hadoop/reducer.py \
-input /user/ssh/5000-8.txt -output /user/ssh/output

```

Run the MapReduce job [ Cloudera Sandbox ]

```
hadoop jar /usr/lib/hadoop-0.20-mapreduce/contrib/streaming/hadoop-streaming-2.6.0-mr1-cdh5.12.0.jar -file /home/cloudera/mapper.py    -mapper /home/cloudera/mapper.py -file /home/cloudera/reducer.py   -reducer /home/cloudera/reducer.py -input /user/cloudera/*.py -output /user/cloudera/output
```

# Instructions for Module 5 - Spark 

### WordCount Spark Example:

Upload a large file into HDFS

```
wget http://www.gutenberg.org/files/5000/5000-8.txt
hdfs dfs -mkdir /user/sshuser
hdfs dfs -copyFromLocal 5000-8.txt /user/sshuser
```

Start pyspark interactive session

```
pyspark
```

Initialize the Spark Context

```
text_file = sc.textFile("/user/sshuser/5000-8.txt")
```

Create flatmap variable

```
counts = text_file.flatMap(lambda line: line.split(" ")).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)
```

Save the RDD into File

```
counts.saveAsTextFile("/user/sshuser/output")
```
Output will be stored in the directory

```
hdfs dfs -ls /user/sshuser/output
Found 3 items
-rw-r--r--   1 sshuser supergroup          0 2018-03-17 16:08 /user/sshuser/output/_SUCCESS
-rw-r--r--   1 sshuser supergroup     270851 2018-03-17 16:08 /user/sshuser/output/part-00000
-rw-r--r--   1 sshuser supergroup     269236 2018-03-17 16:08 /user/sshuser/output/part-00001
```

## Spark SQL Example


```
pyspark --packages com.databricks:spark-csv_2.10:1.4.0
df1 = sqlContext.sql("SELECT * FROM default.hivesampletable")
df1.show()
```

Load Databrick CSV parsing library/module

```
df = sqlContext.read.format('com.databricks.spark.csv').options(header='true', inferschema='true').load('/user/sshuser/cars.csv')
df.show()
df.printSchema()
df.select("mpg").show()
df.select(df['mpg'],df['hp']+ 1).show()
df.filter(df['mpg'] > 9).show()
df.groupBy("_c0").count().show()
```

## HomeWork:

1. Take a large text file 10 GB, and run the word count program and compare the performance between MapReduce and Spark
2. Take a CSV file create hive table out of it, compare performance between SparkSQL and Hive queries

# Module 6 

## Pig Examples

Download the data set

wget https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/beginners-guide-to-apache-pig/assets/driver_data.zip

Unzip and upload it to HDFS

## Hbase Examples

Switch to hbase user
```
su - hbase
hbase shell
```
Data Definition language commands in Hbase shell

```
hbase>create 'driver_dangerous_event','events'
hbase>list
```
Download the sample data file -- Run it on Linux Prompt

```
curl -o ~/data.csv https://raw.githubusercontent.com/hortonworks/data-tutorials/d0468e45ad38b7405570e250a39cf998def5af0f/tutorials/hdp/hdp-2.5/introduction-to-apache-hbase-concepts-apache-phoenix-and-new-backup-restore-utility-in-hbase/assets/data.csv

hdfs dfs -put ~/data.csv /tmp
```
Load the CSV file into Hbase -- Run it on Linux Prompt

```
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=,  -Dimporttsv.columns="HBASE_ROW_KEY,events:driverId,events:driverName,events:eventTime,events:eventType,events:latitudeColumn,events:longitudeColumn,events:routeId,events:routeName,events:truckId" driver_dangerous_event hdfs://sandbox-hdp.hortonworks.com:/tmp/data.csv
```
Scan the table -- Run it on Hbase shell

```
scan 'driver_dangerous_event'
```
Insert Commands for hbase

put '<table_name>','row1','<column_family:column_name>','value'

```
put 'driver_dangerous_event','4','events:driverId','78'
put 'driver_dangerous_event','4','events:driverName','Carl'
put 'driver_dangerous_event','4','events:eventTime','2016-09-23 03:25:03.567'
put 'driver_dangerous_event','4','events:eventType','Normal'
put 'driver_dangerous_event','4','events:latitudeColumn','37.484938'
put 'driver_dangerous_event','4','events:longitudeColumn','-119.966284'
put 'driver_dangerous_event','4','events:routeId','845'
put 'driver_dangerous_event','4','events:routeName','Santa Clara to San Diego'
put 'driver_dangerous_event','4','events:truckId','637'
```

Select/Get Commands
get '<table_name>','<row_number>'
get 'table_name', 'row_number', {COLUMN ⇒ 'column_family:column-name '}

```
get 'driver_dangerous_event','1',{COLUMN => 'events:driverName'}
get 'driver_dangerous_event','1',{COLUMNS => ['events:driverName','events:routeId']}
```
## Home work
1. Try pig and hbase examples using car.csv data file


# Module 7

## Sqoop Tutorial

Download Mysql java connector from 

https://dev.mysql.com/downloads/connector/j/

Extract and copy the bin.jar to

```
sudo cp mysql-connector-java-5.1.46-bin.jar /usr/hdp/current/sqoop-client/lib/

```

Install MySQL on Azure Ubuntu box

```
sudo apt install mysql-server
```

It will prompt to choose mysql root password put test

Now you need to modify mysql to run on all ports so that MR jobs can access it from any node.

edit the file /etc/mysql/mysql.conf.d/mysqld.cnf and modify the bind-address to 0.0.0.0

```
bind-address            = 0.0.0.0
```
```
sudo cp my.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
```

Restart Mysql
```
sudo /etc/init.d/mysql restart
```

Now login to mysql and grant priviledges

```
mysql -uroot -ptest

GRANT ALL ON *.* to root@'%' IDENTIFIED BY 'test'; 
GRANT ALL ON *.* to root@'localhost' IDENTIFIED BY 'test'; 
```

Make sure you can login to mysql using -h parameter example

```
mysql -h10.0.0.18 -uroot -ptest
```

Now clone the sample data from github

```
git clone https://github.com/datacharmer/test_db.git
```

Load the data into DB

```
cd test_db
mysql -uroot -ptest < employees.sql
```

Run ingest this data into Hive

```
sqoop import --connect jdbc:mysql://hn0-omni21.w3rnsdgdzaie1pas2ohmnw4jfd.cx.internal.cloudapp.net:3306/employees --username root -P --split-by emp_no --columns emp_no,first_name --table employees  --target-dir /user/sshuser/sq2 --fields-terminated-by "," --hive-import --create-hive-table --hive-table default.employee
```
