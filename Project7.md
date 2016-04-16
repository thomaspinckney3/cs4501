
Docker compose example for a two node spark cluster (one master and one worker node):
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
  links:
    - kafka
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
    - kafka
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

Sample Pyspark program:
```
from pyspark import SparkContext

sc = SparkContext("spark://spark-master:7077", "PopularItems")
data = sc.textFile("/tmp/data/access.log", 2)

pairs = data.map(lambda line: line.split("\t"))
pages = pairs.map(lambda pair: (pair[1], 1))
count = pages.reduceByKey(lambda x,y: x+y)

output = count.collect()
for page_id, count in output:
    print ("page_id %s count %d" % (page_id, count))
print ("Popular items done")

sc.stop()
```

which reads a file liek this of tab separated values:
```
tp      4
bob     5
tp      4
hank    3
```

And then you can submit your Pyspark job to your Spark cluster like this:
```
docker exec -it spark-master bin/spark-submit --master spark://spark-master:7077 --total-executor-cores 2 --executor-memory 512m /tmp/data/hello.py
```
