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

You will be adding things to a Kafka queue (or as Kafak calls them,
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

We're writing a new backend process that will sit in and endless loop,
waiting for new listings to appear in our Kafka topic. When a new
listing appears, the search indexer will call the ES API for adding it
to the search index.

### Other changes ###

As noted above, your experience service for creating listings will be
updated to add the new listing to a Kafka topic for new listings.

A new web front end page, the search result page, will be created. It
will call a new experience service, the search experience service, to
get the results of a user's query.

Ideally, you will also add a common header to your other pages
containing a search box. When someone types a query into the search
box the user will be taken to your SRP to display the results. Whether
you add this header or not, your SRP should have a search box that
allows a user to enter a query.

Implementation
--------------

You will be adding three new containers to your application:

   - ElasticSearch based on the 'elasticsearch:2.0' image on Dockerhub
   - Kafka based on the 'spotify/kafka' image on Dockerhub
   - The backend search indexer based on my tp33/django:1.1 image on Dockerhub

You will also need to update the image you're using for your models
api, experience service api, and web front end api to the 1.1 version
of my tp33/django container. I've updated it to update the versions of
a few things and more importantly to include the python Kafka and ES
client libraries. Since it's only your experience service app that is
talking to Kafka and ES, you could only upgrade your experience
service container to 1.1. However, it's easier to just run the same
version everywhere.

You can download and run the new Kafka and ES containers like:

    run -d --name kafka --env ADVERTISED_HOST=kafka --env ADVERTISED_PORT=9092 spotify/kafka
    run -d -p 9200:9200 --name es elasticsearch:2.0 -Des.network.host=es

Make sure the keep the container names as I used here unless you want
to figure out the nuances of Kafka and it's dependent Zookeeper
configuration :)

And let's start a container to run your search indexer:

    docker run -it --name batch --link kafka:kafka --link es:es tp33/django:1.1

And test out calling ES:

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

And test out adding messages to a Kafka queue:

    >>> from kafka import SimpleProducer, KafkaClient
    >>> import json
    >>> kafka = KafkaClient('kafka:9092')
    >>> producer = SimpleProducer(kafka)
    >>> some_new_listing = {'title': 'Used MacbookAir 13"', 'description': 'This is a used Macbook Air in great condition', 'id':42}
    >>> producer.send_messages(b'new-listings-topic', json.dumps(some_new_listing).encode('utf-8'))
    [ProduceResponse(topic=b'new-listings-topic', partition=0, error=0, offset=0)]

And test our receiving messages from Kafka:

    >>> from kafka import KafkaConsumer
    >>> consumer = KafkaConsumer('new-listings-topic', group_id='listing-indexer', bootstrap_servers=['kafka:9092'])
    >>> for message in consumer:
    >>>    new_listing = json.loads(message.value)

Note, that Kafka is a little picky on starting up. Sometimes you have
to try connceting, sending a message or receiving a message twice for
it to work. This is only the case with a new topic. After the topic
has been created and messages sent/received on it, things should work
fine. The other thing to be aware of is that by default, the first
time a consumer connects it will only receive messages sent AFTER that
point.
