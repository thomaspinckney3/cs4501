Overview
========

This project is semi grabbag-style. There are six different topics that is related to improving the website you build. You must complete Continuous Integration(#3), Integration Testing(#4) and Load Balancing(#6). In addition, you will need to choose one topic from the remaining three which are Hosting on DigitalOcean(#1), Caching with Redis(#2) and Performance Testing(#5).
The detailed topic descriptions are listed as follows:

1. Hosting on DigitalOcean
    
      Email me (Professor Pinckney) for an invite if you you'd like to do this.
    
2. Caching with Redis
    
      Install the Python Redis client https://pypi.python.org/pypi/redis with pip and
      start a Redis docker image such as https://hub.docker.com/_/redis/ .
      
      You can't use Django's caching interfaces to configure whole page caching with Redis without using additonal packages (Look into https://github.com/niwinz/django-redis if you want to set Redis as Django's caching backend). Instead just directly call the Redis python client to store pages and later look them up. Think about how and when
      to invalidate the cache'd content (after a certain amount of time? when the DB changes? something else?).
    
3. Continuous Integration
    
      Use a continuous integration (CI) tool such as Travis CI to automate builds of your marketplace project. Your CI build should also automatically run unit tests or integration tests that your team has written and verify that they all pass.
    
4. Integration tests
    
      Write integration tests using Selenium http://www.seleniumhq.org to test your web front end.
    
5. Performance testing
    
      Measure how fast your app will scale. This is a good project to do in combination with #1 above
      to measure how fast it runs on your laptop vs a DigitalOcean VM (it may not be as much different
      as you expect!). I recommend using JMeter http://jmeter.apache.org which is also available via
      a Docker image at https://hub.docker.com/r/hauptmedia/jmeter/ .
    
6. Load balancing
    
      A popular open source load-balancer is [HAProxy](https://en.wikipedia.org/wiki/HAProxy).
      The official Docker image can be found [here](https://hub.docker.com/_/haproxy/).
      The official image, however, does not work right out of the box. As the Docker page describes, you must write a
      configuration file that specifies how the load balancer should behave (i.e. specify where it should forward requests,
      how extensive its logs should be, etc.), then you must build a new image that inherits from the official docker image but
      also incorporates this file.
      
      HaProxy has extensive [documentation](http://cbonte.github.io/haproxy-dconv/1.6/intro.html) and great general information
      about load-balancers and proxies in general.
      
      - For this application, the config file needs 4 sections that can be roughly described like so:
        * global: settings which apply to all sections
        * default: default settings which apply to the proxies below it, i.e. logging settings
        * frontend: settings concerning HAProxy's listening for connections from clients
        * backend: settings concerning the servers where HAPoxy sends/forwards incoming connections
            
      More details about the configuration file/how to make it, can be found in the documentation and online.
      A good debugging tool and method to verify that your load balancer is working is to look at the load balancer's 
      server logs. The logs don't just show up in the terminal though - a separate server must be set up to receive logs.
      [papertrail](https://papertrailapp.com/) is a great cloud-hosted free log manager/server - you can arrange to
      have the server logs be sent to a papertrail account by following the directions [here](http://help.papertrailapp.com/kb/configuration/haproxy/)
      
      
      
Deliverable
===========

Please turn in via email a write up including:

1. describing what you did

2. updated docker-compose.yml if you are running any new or changed containers
 
2. link to any code in github you wrote for this (especially for caching)

3. include any sample output (especially for the load balancer, continuous integration, performance test examples)

4. include any command lines (such as for running load tests, unit tests or load balancers)

5. for hosting on DigitalOcean, include a link to your app or service working there
