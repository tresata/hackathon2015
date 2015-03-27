hackathon 2015
==============

Welcome to hackathon 2015!

## Machines

We have a Hadoop cluster with one master and four slaves. The slaves have sixteen cores, 15 1TB data drives, and 128GB of RAM. You will have ssh access to the slaves.

Please spread yourselves out across the machines.

## Getting Started

You can obtain a username and login information from one of the Tresata representatives.

Edit your hosts file (/etc/hosts) and add a line to associate the ip to the hackathon hostname:

    <ip-address>  hackaton-master.datachambers.com

Ssh into a server where you can access the retail data stored on the hackathon Hadoop cluster.

    > ssh <username>@<ip-address>

and enter the password you were given.

We made Hive, Spark, and pySpark command-line interfaces available, and included a tool to compile and run simple Scalding scripts on-the-fly.

## Hive

Give Hive a whirl and run a sample query:

    > hive

Try pasting the following query into the hive command-line interface:

    hive> show tables;
    OK
    hackathon
    Time taken: 0.034 seconds, Fetched: 1 row(s)

    hive> select * from hackathon limit 10;

This will return all the fields for the first ten items in the 'hackathon' table.

## Spark

Now give the Spark-shell a test:

    > spark-shell --num-executors 4 --executor-cores 4 --executor-memory 4G

Read in the data and run a simple query that calculates the number of purchases for each upc in the sample data:

    val dataRDD = sc.textFile("/data/sample")
    val upcs = dataRDD.map(_.split("\\|")(12))
    val upcCounts = upcs.map(upc => (upc, 1)).reduceByKey((a, b) => a + b)
    upcCounts.take(10)

## pySpark

You can also do the same query using a python version of the Spark shell.

    > pyspark --num-executors 4 --executor-cores 4 --executor-memory 4G

    dataRDD = sc.textFile("/data/sample")
    upcs = dataRDD.map(lambda line: line.split('|')[12])
    upcCounts = upcs.map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)
    upcCounts.take(10)


## Scalding

In addition to the Hive and Spark shells, we're also packaging Eval-tool, a tool to compile and run Scalding scripts without having to create a project. If you create a file called test.scala with the following contents:

    import com.twitter.scalding._
    import com.tresata.scalding.Dsl._
    import com.tresata.scalding.util.ScaldingUtil

    (args: Args) => {
      new Job(args) {
        ScaldingUtil.sourceFromArg(args("input"))
          .groupBy('upc) { _.size }
          .write(ScaldingUtil.sourceFromArg(args("output")))
      }
    }

you can run a query on the data set sample from the command-line:

    > eval-tool test.scala --hdfs --input bsv%/data/sample --output bsv%upc_counts

This will generate a bar-separated file called 'upc_counts' in your HDFS home directory, containing the upc numbers along with their total counts.

To access your HDFS location, you need to use hadoop fs commands (some reference: http://www.folkstalk.com/2013/09/hadoop-fs-shell-command-example-tutorial.html). For example, to take a look at your home directory on HDFS, use

    > hadoop fs -ls

or

    > hadoop fs -ls /user/username

## Resource Manager
http://hackaton-master.datachambers.com:8088
