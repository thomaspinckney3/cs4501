Overview
========

In this project you will add the ability for users to search your
site. When they add new listings, the listings will be added to a
search engine. You'll add a search page that users can use to query
the search engine.

Architecture
------------

There are three components to this project:

   - Adding newly created listings to a Kafka queue
   - A search indexer that takes new listings out of Kafka and
     indexing them into Elastic Search (ES)
   - Building a search result page (SRP) that queries ES

The new SRP will be powered by its own new experience service API.

This project will also introduce the first part of your system that
runs code outside of the context of a web request. Step two above,
adding things to ES, will be a backend Python process that constantly
runs pulling new listings out of Kafka and adding them to ES.

### Kafka ###

As you recall from class, Kafka is an open source queing system built
by LinkedIn. We'll be running a Docker image of it built by Spotify
since it combines all the necesary pieces into one easy-to-use
container.

You will be adding things to a Kafka queue (or as Kafka calls them,
topics) in your new listing experience service. Whenever a user
creates a new listing, you'll post the relevent information (title, id
in the db, etc) into the Kafka topic.

We could directly add the new listing to ES instead of going through
Kafka. However, using Kafka gives us two things. First, it's
potentially faster than waiting for ES to index the new
listings. Second, it allows us to ingest LOTS of new listintgs even if
ES can't keep up (or is down). Effecitvely Kafka acts like a buffer
between creating listings and indexing listings.

### Elastic Search ###

ElasticSearch is an open source search engine built on top of the open
source Lucene text indexing and retrieval system. ES provides two APIs
we'll be using: one for adding a document to the search index (in our
case the documents are listings) and another for querying for
documents (how we look up documents that match a user query).

### Search Indexer ###

We're writing a new backend process that will sit in an endless loop,
waiting for new listings to appear in our Kafka topic. When a new
listing appears, the search indexer will call the ES API for adding it
to the search index.

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
header in one place that all pages can share.

Implementation
--------------

You will be adding three new containers to your application:

   - ElasticSearch based on the 'elasticsearch:2.0' image on Dockerhub
   - Kafka based on the 'spotify/kafka' image on Dockerhub
   - The backend search indexer based on my tp33/django:1.2 image on Dockerhub

You can download and run the new Kafka and ES containers like:

    run -d --name kafka --env ADVERTISED_HOST=kafka --env ADVERTISED_PORT=9092 --hostname="kafka" spotify/kafka
    run -d -p 9200:9200 --name es elasticsearch:2.0 -Des.network.host=es

These images may take a few minutes to download as you're pulling down different Java versions for each, dependent apps like Zookeeper, and the main ES and Kafka apps themselves. Still, a lot easier than building and installing all the tools and depencies from source!

Make sure the keep the container names as I used here unless you want
to figure out the nuances of Kafka and it's dependent Zookeeper
configuration :)

And let's start a container to run your search indexer:

    docker run -it --name batch --link kafka:kafka --link es:es tp33/django:1.2
    root@d806ea9af85a:/app#

At this point Docker should have downloaded the 1.2 verison of the container. Notice how easy it is to build a new environment and distribute it to everyone.

And in that container you can try some simple tests of ES:

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
    >>> 

In the example above with ES, we are indexing a JSON document with a title, description and listing id field. ES will by default index whatever fields in whatever documents we give it. The `es.index()` call is specifying that we index the documents using an index called 'listing_index'. Since this index doesn't exist, ES will create it.

Then we call `es.indices.refresh()` on listing_index. Until this is done, ES hasn't actually comitted the changes to the index files and thus queries won't 'see' the new documents. This is a speed optimization ES does allowing many new documents to be added and then the index files only updated once.

Finally, there's an example of calling `es.search()` to query the listing_index for documents that match the query 'macbook air'. We're also specifying that we only want the top 10 results returned. The matches, if any, are returned in the response's `['hits']['hits']` array. Note that the 'id' of each hit matches the id we passed in when indexing the document. We're using the DB assigned primary key id when indexing and so this allows our experience code to quickly look up the corresponding listing from the db by it's primary key at query time.

And test out adding messages to a Kafka queue via a 'SimpleProducer':

    >>> from kafka import SimpleProducer, KafkaClient
    >>> import json
    >>> kafka = KafkaClient('kafka:9092')
    >>> producer = SimpleProducer(kafka)
    >>> some_new_listing = {'title': 'Used MacbookAir 13"', 'description': 'This is a used Macbook Air in great condition', 'id':42}
    >>> producer.send_messages(b'new-listings-topic', json.dumps(some_new_listing).encode('utf-8'))
    [ProduceResponse(topic=b'new-listings-topic', partition=0, error=0, offset=0)]

We're queing up a message via producer.send_messages which takes a message (in this case a JSON doument) and a topic name (in this case 'new-listings-topic'). The returned value shows that the message was queued up succesfully at offset 0 into the queue. Further messages will appeneded at increasing offsets. You may receiver a "LeaderNotAvailable" error the first time you run send_messages for a given topic. This is normal, and should go away on the next try (once the topic is created). If it persists, you might have a more significant error.

And test our receiving messages from Kafka:

    >>> from kafka import KafkaConsumer
    >>> consumer = KafkaConsumer('new-listings-topic', group_id='listing-indexer', bootstrap_servers=['kafka:9092'])
    >>> for message in consumer:
    >>>    new_listing = json.loads((message.value).decode('utf-8'))

Here we're showing an example of a consumer reading messages from the 'new-listings-topic' topic. The consumer is part of the 'listings-indexer' consumer group. Each topic can have multiple groups of consumer reading messages. Each message will be delivered exactly once to SOME member of each group. That is, if there are three clients consuming messages from this topic and all are part of the same group, only one of the three clients will get any given message. This functionality is built to support scaling up the number of consumers. For example, if you had millions of new listings being created per day you might want more than one consumer reading the new listing messages and adding them to ES. However, you'd want to make sure that each new listing was only added to ES once. You can create a new django container and run this script there too to see this behavior in action.

Note, that Kafka is a little picky on starting up. Sometimes you have
to try connecting, sending a message or receiving a message twice for
it to create the topic properly. This is only the case with a new topic. After the topic
has been created and messages sent/received on it, things should work
fine.

The other thing to be aware of is that by default, the first
time a consumer connects it will only receive messages sent AFTER that
point. So in the example above where you use the producer to send a message and then
start a client to read the messages, the client will hang waiting for a new message. To see the client
actually receive a message you'll need to open two shells, run the producer and consumer simultaneously
and then should see the consumer receive messages from the sender.
