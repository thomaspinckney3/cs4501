Overview
========

In this project you will build a recommendation system that generates accurate recommendations based on a recurring map/reduce job on Apache Spark. Your Spark job will take a web site access log as input and as output produce data that will be used by the recommendation system.

Co-views
--------

One type of recommendation that a website may wish to produce is something like "People who looked at this also looked at ... ". That is, for every item on your site you would like to be able to produce a list of other items that people also browsed. These are sometimes called co-viewed items. The idea being that if a user looks at two different items they're probably similar in some important way so they should be associated with each other. Then in the future, when another user looks at one of those items the website can recommend they look at the other item as well.

Conceptually computing co-viewed items is straightforward: look at all the items that a user clicks on and put them all into the same co-view set. However, this has two problems. First, an item that a user looks at today probably shouldn't be considered a co-view of an item a user looks at tomorrow -- the user may be looking for very different things tomorrow so the co-viewed items wouldn't necesarily be similar. We should only associate two items as co-views if they happen in relatively quick succession and as part of the same search session. The second problem is that some users might randomly click on completely unrelated things in quick succession. We should therefore only associate two items as co-views if a critical mass of different people all click on the same pair of items.

In our project we'll ignore the first problem (clicks in quick succession) but we will address the second problem (critical mass of different people co-clicking).

A pseudocode map-reduce style algorithm for computing co-views is something like:

  1. Read data in as pairs of (user_id, item_id clicked on by the user)
  2. Group data into (user_id, list of item ids they clicked on)
  3. Transform into (user_id, (item1, item2) where item1 and item2 are pairs of items the user clicked on
  4. Transform into ((item1, item2), list of user1, user2 etc) where users are all the ones who co-clicked (item1, item2)
  5. Transform into ((item1, item2), count of distinct users who co-clicked (item1, item2)
  6. Filter out any results where less than 3 users co-clicked the same pair of items

Implementation
==============

Using the Access Log
------------------------

In Project 5 you created an access log file to record which users viewed which items. This will serve as the input to your Spark map/reduce job. You can use Docker volumes to allow Spark to see the log file. Whenever you decide to mount a volume, follow the general security principle of least privilege by making sure that containers don't needlessly have access to important files they don't need.

Spark Setup
-----------

Here is a Docker compose example for a two node spark cluster (one master and one worker node):
```
spark-master:
  image: gettyimages/spark
  command: bin/spark-class org.apache.spark.deploy.master.Master -h spark-master
  container_name: spark-master
  hostname: spark-master
  environment:
    MASTER: spark://spark-master:7077
    SPARK_CONF_DIR: /conf
  expose:
    - 7001
    - 7002
    - 7003
    - 7004
    - 7005
    - 7006
    - 7077
    - 6066
  ports:
    - 4040:4040
    - 6066:6066
    - 7077:7077
    - 8080:8080
  volumes:
    - ./data:/tmp/data

spark-worker:
  image: gettyimages/spark
  command: bin/spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077
  container_name: spark-worker
  hostname: spark-worker
  environment:
    SPARK_CONF_DIR: /conf
    SPARK_WORKER_CORES: 2
    SPARK_WORKER_MEMORY: 512m
    SPARK_WORKER_PORT: 8881
    SPARK_WORKER_WEBUI_PORT: 8081
  links:
    - spark-master
  expose:
    - 7012
    - 7013
    - 7014
    - 7015
    - 7016
    - 8881
  ports:
    - 8081:8081
  volumes:
    - ./data:/tmp/data
```

Here's a sample Spark program. It will read in a file consisting of two values -- a user id and a page id. The program will then calculate the frequency of each page id. 

```
from pyspark import SparkContext

sc = SparkContext("spark://spark-master:7077", "PopularItems")

data = sc.textFile("/tmp/data/access.log", 2)     # each worker loads a piece of the data file

pairs = data.map(lambda line: line.split("\t"))   # tell each worker to split each line of it's partition
pages = pairs.map(lambda pair: (pair[1], 1))      # re-layout the data to ignore the user id
count = pages.reduceByKey(lambda x,y: int(x)+int(y))        # shuffle the data so that each key is only on one worker
                                                  # and then reduce all the values by adding them together

output = count.collect()                          # bring the data back to the master node so we can print it out
for page_id, count in output:
    print ("page_id %s count %d" % (page_id, count))
print ("Popular items done")

sc.stop()
```

which reads a file like this of tab separated values:

```
tp      4
bob     5
tp      4
hank    3
```

The Pyspark program and it's data file will need to be present in the Spark containers. This is accomplished by using Docker volumes to make a data directory under wherever you run ```docker-compose up```. Put these files in that directory and then spark-master will be able to access your Pyspark program and spark-worker will be able to access the data file.

Submit and run your Pyspark job to your Spark cluster like this:
```
docker exec -it spark-master bin/spark-submit --master spark://spark-master:7077 --total-executor-cores 2 --executor-memory 512m /tmp/data/hello.py
```

When you run your Spark program, Spark will generate A LOT of logging output. You'll have to look carefully for your program's output in the middle of it all. Also, if something goes wrong there may be many stack traces -- look for the first one to get an idea of what went wrong.

If all works then somewhere in the midst of all the Spark logging you'll see the output pages and their counts. Congratulations! You've run your first Spark job.

Take a moment and consider what has just happened. Spark distributed your program across however many workers were available, ran the map command, then shuffled the data for the reduce, and collected the results for you to see. While it was a lot of overhead to count four lines in a text file, this same program would have worked on a 1,000 node cluster processing a terrabyte of text files.

Spark Programming
-----------------
Spark calls the data objects it manipulates RDDs. The first RDD in the sample program above is created by the call to sc.textFile. Each RDD is actually partitioned across the workers in your Spark cluster. However, when programming you just reference the RDD as a single object. Spark handles the distributed computing behind the scenes.

RDD's support map/reduce style operations like we've discussed in class. For example, in the sample program the line
```
pairs = data.map(lambda line: line.split("\t"))
```
applies a map operation to the ```data``` RDD to produce a new RDD called ```pairs```. Every row of text in ```data``` is transformed into a row containing a Python pair/tuple in ```pairs```. The next line
```
pages = pairs.map(lambda pair: (pair[1], 1))
```
applies a map operation to each row in ```pairs``` to produce a new RDD called ```pages``` where each row is a pair of page id and the number 1. Finally, the line
```
count = pages.reduceByKey(lambda x,y: x+y)
```
does a reduce operation. This groups all the rows with the same key onto the same worker and then reduces all the values for those keys by summing.

Spark supports some other useful opreations on RDDs:

  1. groupByKey() - take all rows of the form (K,V) and group them into a single row of form (K, list of different Vs)
  2. distinct() - filter out all duplicate rows in an RDD to produce a new RDD with distinct rows
  3. flatMap() - like map, but returns multiple rows for when you want produce a new RDD with multiple output rows for each input row
  4. filter() - remove certain rows from an RDD
  
The Spark programming guide at http://spark.apache.org/docs/latest/programming-guide.html#basics is a good reference.

Debugging Spark Programs
------------------------

Debugging Spark programs is somewhat more difficult for two reasons:

  1. Your code is running across multiple containers so if you print something it will go to the stdout of whatever container it is running in. This makes it hard to find and piece together.
  2. Spark logs a ton of debug info on it's own so your output can get lost along the way.
  
I highly recommend that you write and debug your program by printing each RDD after every step. The sample program shows how to print an RDD:
```
output = count.collect()                          # bring the data back to the master node so we can print it out
for page_id, count in output:
    print ("page_id %s count %d" % (page_id, count))
print ("Popular items done")
```

The ```collect``` call is what's important -- it collects all the data from the RDD it is called on and brings it back to the spark master. So then when the subsequent print operations run on it you know that you have all the data and that the output is appearing on the spark-master's stdout where you should see it in the terminal that you ran ```bin/spark-submit``` command.

It is also very useful to write comments for each line describing what the format of the RDD is. Since these RDDs are pairs, I always write down what the first and second value in the pair is to help me keep track.

If you have an error in your program, Spark will print an exception (or many exceptions). You'll have to read carefully through the logging output to find a reference to what line or part of your program actually failed.


Creating a Recommendation System
--------------------------------

With Spark's output of popular co-views, all that's left is figuring out how to store the information efficiently in a manner such that it would be easily accessible by a recommendation service. For this, we will create another table in our database with only two columns. The first column will contain item-id's, and the second column will contain variable length strings that will contain a comma separated "list" of all item-ids that were co-viewed 3 or more times with the item in the first column. This format is not difficult to implement and is a natural choice for a recommendation service, which need only have an item-id to retrieve from the database a list of "recommended" item-ids for that item. So an example recommendation table could look like this:

```
item_id  | recommended_items
---------|------------------
1        |  5, 7, 9,
2        |  3, 4
3        |  4, 2, 1, 5,
```

You will need to create a new model for your recommendations and create additional services in your models api application for creating, updating and reading the recommendations. Your Spark job will have to call these services to populate the recommendations at the end of the Spark job.

Once you have correctly populated the database with Sparks output for the recommendation service, you can now move on to implementing the experience service and web front end updates. In every item detail page, you must now display a few of the recommended items there are any. The experience service should retrieve these using the models api services and your web front end should render them into the page's HTML.


Automated Spark Job
-------------------
The last step is to automate the Spark job so that it executes on regular intervals to keep the recommendations up to date and accurate. You should create a simple bash script that triggers the job around every minute or two so that you have time to test the effect of co-views on an item's recommendations. This script will be separate from docker compose and should be started only after you've already 'docker-composed up'. 

Since we don't have a real stream of data, you can just run the Spark job on the entirety of the same single access log file every time, the idea being that as the log is appended to and expanded over time, the generated recommendations are keeping up right alongside it (hence why you should erase the recommendations table every time the job is run)


What to turn in
================

You should include in your submission:

  1. Your source code in github
  2. User stories and working unit tests
  3. Input file that was generated by your web app that is/will be used by Spark
  4. Output file that contains the output produced by one Spark job on the input file from (2). 
  5. Bash script that automates dependency installation and Spark job scheduling
  6. Anything else not mentioned in this repo that was necessary to make everything in project 7 work (i.e. had to install an extra package).
  
  For parts 3-5, please indicate in your submission email where in the repo these files are (don't attach it).
  
We should be able to get everything up and running with just two commands: docker-compose up and then running the bash script.
