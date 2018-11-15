Overview
========

This project is semi grabbag-style. There are six different topics that are related to improving the website you build.

You *must* complete Continuous Integration, Integration Testing and Load Balancing.

In addition, you will need to choose one topic from the remaining three which are Hosting on DigitalOcean, Caching with Redis and Performance Testing.
The detailed topic descriptions are listed as follows:

# Required Topics
- Continuous Integration

    Use a continuous integration (CI) tool such as Travis CI to automate builds of your marketplace project. Your CI build should also automatically run unit tests or integration tests that your team has written and verify that they all pass.

- Integration tests

    Write integration tests using [Selenium](http://www.seleniumhq.org) to test your web front end.
    To create Selenium tests in python, you'll need to install the selenium package from PyPi. This package will enable to use Selenium's python API to interact with the Selenium WebDriver. 
      
   From [Selenium's documentation](http://www.seleniumhq.org/docs/03_webdriver.jsp#introducing-webdriver):
      ` WebDriver’s goal is to supply a well-designed object-oriented API that provides improved support for modern advanced web-app testing problems. Selenium-WebDriver makes direct calls to the browser using each browser’s native support for automation. How these direct calls are made, and the features they support depends on the browser you are using. ` 
      
    For this class, we only expect you to create tests for Google Chrome. To interface with Google Chrome, the Selenium WebDriver needs to use ChromeDriver ([info](http://www.seleniumhq.org/docs/03_webdriver.jsp#chromedriver))
      
     In the past, we've had students download the [chrome driver](https://sites.google.com/a/chromium.org/chromedriver/home) locally. But many people had problems with their python code not finding the local driver in their PATH. As a result, we've transitioned to a new solution using, you guessed it, containers! Selenium has created a [docker container image](https://hub.docker.com/r/selenium/standalone-chrome/) with Google Chrome, ChromeDriver, and the Selenium WebDriver already installed. The Selenium WebDriver in this container can be accessed via the 'Selenium Server' that is also included in the container (and is started upon container instantiation). From Selenium's documentation: 
      `You may, or may not, need the Selenium Server, depending on how you intend to use Selenium-WebDriver. If your browser and tests will all run on the same machine, and your tests only use the WebDriver API, then you do not need to run the Selenium-Server; WebDriver will run the browser directly... use the Selenium-Server... (if) you want to connect to a remote machine... `

    To set up containers for Selenium and for the test script, your docker-compose should include a section that looks something like this:
    ```
    selenium-chrome:
      image: selenium/standalone-chrome
      container_name: selenium-chrome
      links:
        - web:web
      ports:
        - "4444:4444"

    selenium-test:
      image: tp33/django
      container_name: selenium-test
      links:
        - selenium-chrome:selenium-chrome
        - web:web
      volumes:
        - ./app/selenium:/app
      command: bash -c "pip install selenium==<some version number> && python <your_selenium_test_script>.py"
      ```
    
 See [Selenium's Python documentation](http://selenium-python.readthedocs.io/getting-started.html#using-selenium-with-remote-webdriver) on how to connect to a remote WebDriver from your python code.

- Load balancing
    
     A popular open source load-balancer is [HAProxy](https://en.wikipedia.org/wiki/HAProxy).
      The official Docker image can be found [here](https://hub.docker.com/_/haproxy/).
      You must write a configuration file that specifies how the load balancer should behave (i.e. specify where it should forward requests, how extensive its logs should be, etc.), then you must build a new image that inherits from the official docker image but also incorporates this file.
      
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
# Optional Topics (Choose One)

- Hosting on DigitalOcean
    
     Email me (Professor Pinckney) for an invite if you you'd like to do this. Do this within the first week this project is assigned so that I have time to get invites to everyone that needs one.


- Caching with Redis
   
     Install the Python Redis client https://pypi.python.org/pypi/redis with pip and
      start a Redis docker image such as https://hub.docker.com/_/redis/ .
      
     You can't use Django's caching interfaces to configure whole page caching with Redis without using additonal packages (Look into https://github.com/niwinz/django-redis if you want to set Redis as Django's caching backend). Instead just directly call the Redis python client to store pages and later look them up. Think about how and when
      to invalidate the cache'd content (after a certain amount of time? when the DB changes? something else?).
   
    

    
- Performance testing
    
     Measure how fast your app will scale. This is a good project to do in combination with #1 above
     to measure how fast it runs on your laptop vs a DigitalOcean VM (it may not be as much different
      as you expect!). I recommend using JMeter http://jmeter.apache.org which is also available via
      a Docker image at https://hub.docker.com/r/hauptmedia/jmeter/ .
      
      
      
Deliverable
===========

Please turn in via email a write up including:

1. describing what you did

2. updated docker-compose.yml if you are running any new or changed containers
 
2. GitHub link to any code or config files required to make the project work

3. include any sample output (especially for the load balancer, continuous integration, performance test examples)

4. include any command lines (such as for running load tests, unit tests or load balancers)

5. for hosting on DigitalOcean, include a link to your app or service working there
