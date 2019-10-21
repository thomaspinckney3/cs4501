Overview
========

In this project you will add the ability for users to search your
site. When they add new listings, the listings will be added to a
search engine. You'll add a search page that users can use to query
the search engine using a custom ranking function based on popularity.

Architecture
------------

There are four components to this project:

1. Adding newly created listings to a Kafka queue
2. A search indexer that takes new listings out of Kafka and indexing them into Elastic Search (ES)
3. Extend the web front end and experience services to add a search result page (SRP) that queries ES

This project will also introduce the first part of your system that
runs code outside of the context of a web request. Step two above,
adding things to ES, will be a backend Python worker process that constantly
runs pulling new listings out of Kafka and adding them to ES.

The architecture is illutrated by this image: https://drive.google.com/file/d/0BwxFrbFICisjdUVzQURaTGFzX3c/view?usp=sharing

### User Stories ###

Create new user stories and update old ones to reflect the changes that should be completed by the end of the sprint/project. You should use this as a tool that will help you cement your understanding of how you want your create-listing/add to ES/search flow to work as well as a tool to help you divide up the work among your group members.

### Kafka ###

As you recall from class, Kafka is an open source queuing system built
by LinkedIn. We'll be running a Docker image of it built by Spotify
since it combines Kafka and Kafka's dependency ZooKeeper into one easy-to-use
container.

You will be adding things to a Kafka queue (or as Kafka calls them,
topics) in your new listing experience service. Whenever a user
creates a new listing, you'll post the relevent information (title, id
in the db, etc) into the Kafka topic.

We could directly add the new listing to ES instead of going through
Kafka. However, using Kafka gives us two things. First, it's
potentially faster than waiting for ES to index the new
listings. Second, it allows us to ingest LOTS of new listings even if
ES can't keep up (or is down). Effecitvely Kafka acts like a buffer
between creating listings and indexing listings.

### Elastic Search ###

ElasticSearch is an open source search engine built on top of the open
source Lucene text indexing and retrieval system. ES provides two APIs
we'll be using: one for adding a document to the search index (in our
case the documents are listings) and another for querying for
documents (how we look up documents that match a user query).

Another thing to note is that Elasticsearch 7 offers a new clustering 
methodology, and this may require modifying some system variables. If
the Elasticsearch container prematurely shuts down, please review the
set-up docs [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html).

### Search Indexer ###

We're writing a new backend process that will sit in an endless loop,
waiting for new listings to appear in our Kafka topic. When a new
listing appears, the search indexer will call the ES API for adding it
to the search index. This sits in a separate container called batch.

### Create Listing Changes ###

As noted above, your experience service for creating listings will be
updated to add the new listing to a Kafka topic for new listings.

### Search Result Page ###

A new web front end page, the search result page, will be created. It
will call a new experience service, the search experience service, to
get the results for a user's query.

Ideally, you will also add a common header to your other pages
containing a search box. When someone types a query into the search
box the user will be taken to your SRP to display the results. Whether
you add this header or not, your SRP should have a search box that
allows a user to enter a query.

Note, using Django template inheritance is a good way to define a common
header in one place that all pages can share (search 'Django template extends').

### Custom Ranking Function ###

In addition to performing queries by text matching, you will also add the ability
to perform queries using a custom metric. An example would be having the option to
view "hot" or "relevant" listings by having a scoring function that takes in the popularity
of the item into account.

### Access Log ###

To keep track of the number of views each item has, we will create a log file that keeps track of which users view which item. The access log should be in the form of a file with two columns of values where each row consists of a user-id and an item-id representing an item page view by a logged in user. Similar to the new listing implementation, every time a page view occurs you will need to push the two relevant values (user-id and item-id) to Kafka from the experience layer and have another batch container/script consume the page-view and write/append it to the aforementioned running log file. To get this data into Elasticsearch, you will have a python file within your batch container that periodically parses the access log, counts the number of views for each listing, and updates the view count for each listing in Elasticsearch.


Implementation
--------------

You will be adding three new containers to your application:

   - ElasticSearch based on the 'elasticsearch:7.4.0' image on Dockerhub
   - Kafka based on the 'spotify/kafka' image on Dockerhub. Keep in mind that despite the name
   the image is really Kafka, ZooKeeper and all the configurations to make them work together.
   - The backend search indexer called batch based on tp33/django image on Dockerhub (kafka-python upgraded) whose only job is to run a python script that pulls new listing messages from Kafka and indexes them in ES, a python script that pulls item viewing messages from Kafka and appends them to a log, and a python script that periodically parses the log and updates the view counts in ES.

You can download and run the new Kafka and ES containers like:

```YAML
kafka:
   image: spotify/kafka
   container_name: kafka
   networks:
      - batch_network
      - service_mesh
   environment:
      ADVERTISED_HOST: kafka
      ADVERTISED_PORT: 9092
   hostname: kafka
   
es:
   image: elasticsearch:7.4.0
   container_name: es
   environment:
      - discovery.type=single-node
   networks:
      - service_mesh
      - mod_exp_network
   ports:
      - "9200:9200"

batch:
   image: tp33/django
   container_name: batch
   networks:
      - batch_network
   command: <run python scripts>
```

These images may take a few minutes to download as you're pulling down different Java versions for each, dependent apps like Zookeeper, and the main ES and Kafka apps themselves. Still, a lot easier than building and installing all the tools and depencies from source!

Make sure to keep the container names as I used here unless you want
to figure out the nuances of Kafka and it's dependent Zookeeper
configuration :)

And let's start a container to try out ES and Kafka:

    docker run -it --name batch_test --network <your_batch_network> tp33/django
    root@d806ea9af85a:/app#

And in that container you can try some simple tests of ES:
   
``` PYTHON
root@d806ea9af85a:/app#  python
Python 3.5.0 (default, Oct 29 2015, 07:33:09) 
[GCC 4.9.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from elasticsearch import Elasticsearch
>>> es = Elasticsearch(['es'])
>>> some_new_listing = {'title': 'Used MacbookAir 13"', 'description': 'This is a used Macbook Air in great condition', 'id':42}
>>> es.index(index='listing_index', doc_type='listing', id=some_new_listing['id'], body=some_new_listing)
{'created': True, '_version': 1, '_shards': {'successful': 1, 'total': 2, 'failed': 0}, '_index': 'listing_index', '_id': '42', '_type': 'listing'}
>>> es.indices.refresh(index="listing_index")
>>> es.search(index='listing_index', body={'query': {'query_string': {'query': 'macbook air'}}, 'size': 10})
{'timed_out': False, 'hits': {'total': 1, 'hits': [{'_score': 0.10848885, '_index': 'listing_index', '_source': {'id': 42, 'description': 'This is a used Macbook Air in great condition', 'title': 'Used MacbookAir 13"'}, '_id': '42', '_type': 'listing'}], 'max_score': 0.10848885}, '_shards': {'successful': 5, 'total': 5, 'failed': 0}, 'took': 21}
>>> es.update(index='listing_index', doc_type='listing', id=some_new_listing['id'] , body={ 'script' : 'ctx._source.visits = 0'})
>>> es.update(index='listing_index', doc_type='listing', id=some_new_listing['id'] , body={ 'script' : 'ctx._source.visits += 1'})
>>> es.search(index='listing_index', body={"query": {"function_score": {"query": {"query_string": {"query": "macbook air"}},"field_value_factor": {"field": "visits","modifier": "log1p","missing": 0.1}}}})
{'_shards': {'successful': 1, 'failed': 0, 'total': 1, 'skipped': 0}, 'hits': {'hits': [{'_index': 'listing_index', '_type': 'listing', '_score': 0.2745185, '_source': {'title': 'Used MacbookAir 13"', 'description': 'This is a used Macbook Air in great condition', 'id': 42, 'visits': 1}, '_id': '42'}], 'max_score': 0.2745185, 'total': {'relation': 'eq', 'value': 1}}, 'took': 62, 'timed_out': False}
```
   
In the example above with ES, we are indexing a JSON document with a title, description and listing id field. ES will by default index whatever fields in whatever documents we give it. The `es.index()` call is specifying that we index the documents using an index called 'listing_index'. Since this index doesn't exist, ES will create it.

Then we call `es.indices.refresh()` on listing_index. Until this is done, ES hasn't actually comitted the changes to the index files and thus queries won't 'see' the new documents. This is a speed optimization ES does allowing many new documents to be added and then the index files only updated once.

Next, there's an example of calling `es.search()` to query the listing_index for documents that match the query 'macbook air'. We're also specifying that we only want the top 10 results returned. The matches, if any, are returned in the response's `['hits']['hits']` array. Note that the 'id' of each hit matches the id we passed in when indexing the document. We're using the DB assigned primary key id when indexing and so this allows our experience code to quickly look up the corresponding listing from the db by it's primary key at query time.

Finally, we use `es.update()` to add the `visits` field into the document and increment it. We then call `es.search()` with a custom ranking function. In our example, the `query` and `field_value_factor` both return scores, which are then multiplied by each other to return the overall score. As you can see, the score for the listing retreived by the second search is different from the first. You can see more function examples [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html). 

And test out adding messages to a Kafka queue via 'KafkaProducer':

``` PYTHON
>>> from kafka import KafkaProducer
>>> import json
>>> producer = KafkaProducer(bootstrap_servers='kafka:9092')
>>> some_new_listing = {'title': 'Used MacbookAir 13"', 'description': 'This is a used Macbook Air in great condition', 'id':42}
>>> producer.send('new-listings-topic', json.dumps(some_new_listing).encode('utf-8'))
<kafka.producer.future.FutureRecordMetadata object at 0x7f9df5a71780>
```
   
We're queing up a message via producer.send which takes a message (in this case a JSON doument in bytes) and a topic name (in this case 'new-listings-topic'). The KafkaProducer is in asynchronous mode by default. The returned value shows that the message was queued up asynchronously (The message may NOT be in the queue yet when the value returns).

And test our receiving messages from Kafka (To see messages sent, you need to start another terminal session that connects to the batch_test container. Recall the command `docker exec -it batch_test bash`):
   
``` PYTHON
>>> from kafka import KafkaConsumer
>>> consumer = KafkaConsumer('new-listings-topic', group_id='listing-indexer', bootstrap_servers=['kafka:9092'])
>>> for message in consumer:
...   print(json.loads((message.value).decode('utf-8')))
```

Here we're showing an example of a consumer reading messages from the 'new-listings-topic' topic. The consumer is part of the 'listings-indexer' consumer group. Each topic can have multiple groups of consumer reading messages. Each message will be delivered exactly once to SOME member of each group. That is, if there are three clients consuming messages from this topic and all are part of the same group, only one of the three clients will get any given message. This functionality is built to support scaling up the number of consumers. For example, if you had millions of new listings being created per day you might want more than one consumer reading the new listing messages and adding them to ES. However, you'd want to make sure that each new listing was only added to ES once.

The other thing to be aware of is that by default, the first
time a consumer connects it will only receive messages sent AFTER that
point. So in the example above where you use the producer to send a message and then
start a client to read the messages, the client will hang waiting for a new message. To see the client
actually receive a message you'll need to open two shells, run the producer and consumer simultaneously
and then should see the consumer receive messages from the sender.

Use the code snippets above to implement the python scripts that sits in the batch container. Again, all that script does is just pulling messages from Kafka and index the message in ES!

### GitHub ###

Please turn in a tag to your GitHub repository (e.g. "project5") to your assigned TA when your team has completed the assignment.

