Overview
========

The following are various workflows you may find useful to refer to throughout the
projects. You may need to make modifications to the provided code snippets 
to fit your project's needs.

Continuous Integration (CI)
---------------------------

[Continuous Integration (CI)](https://en.wikipedia.org/wiki/Continuous_integration)
is a practice used by dev ops to frequently merge local code changes and automate 
successful builds of a project code base (e.g. code base needs to pass all unit 
and integration tests before deploying). CI is "continuous" as dev ops typically
merge local code changes multiple times per day.

[Travis CI](https://travis-ci.org/) is a hosted free tool for open source projects 
that we can integrate with GitHub to run unit tests and integration tests on every push. 
A build in Travis CI is `passing` if your script exits with status code `0` (so
you can implement arbitrary testing environments). 

To use Travis CI, connect your GitHub account to it (simply click "sign in with
GitHub" on the [homepage](https://travis-ci.org/)) and place `.travis.yml`, 
a YAML description of your project's testing environment, in your project's root
directory.

An example `.travis.yml` configuration for the course project is as follows:
```YAML
sudo: required
dist: trusty # google-cloud-engine mod of ubuntu 14.04.3 LTS

# if specific versions of docker or docker-compose are needed, see travis-CI docs
services:
  - docker # default is docker 1.8.2 and docker-compose 1.4.2 (as of Oct. 2016)

install: # prepare mySQL database
  - docker pull mysql:5.7.14
  - mkdir db
  - >
    docker run --name mysql -d -e MYSQL_ROOT_PASSWORD='$3cureUS'
    -v `pwd`/db:/var/lib/mysql mysql:5.7.14
  - sleep 10 # need to give time for mysql to start
  - >
    docker run -it --name mysql-cmd --rm --link mysql:db mysql:5.7.14
    mysql -uroot -p'$3cureUS' -h db -v -e
    "CREATE DATABASE cs4501 CHARACTER SET utf8;
    CREATE DATABASE test_cs4501 CHARACTER SET utf8;
    CREATE USER 'www'@'%' IDENTIFIED BY '\$3cureUS';
    GRANT ALL PRIVILEGES ON *.* TO 'www'@'%';"

before_script:
  - docker-compose up -d

script:
  - >
    docker exec -it models bash -c
    "pip install -r requirements.txt &&
    python manage.py test --noinput" # run Django unit tests on models
  # can put other tests (e.g. integrations) here too with exp, web containers

after_script:
  - docker-compose stop
  - docker stop mysql
  - docker rm `docker ps -a -q`
  - docker rmi `docker images -q`
  - rm -rf db
```

Automated Database Creation
---------------------------

[GNU make](https://www.gnu.org/software/make/) is a useful tool for compiling a 
project with a set of file dependencies. We can write a `Makefile` to specify build 
targets, such as automatically creating a `mysql` database with `docker`:

```Makefile
# put 'db/' in your .gitignore file, so git will ignore your database directory.
database: clean
	sudo docker pull mysql:5.7.14
	mkdir db
	sudo docker run --name mysql -d -e MYSQL_ROOT_PASSWORD='$$3cureUS' -v `pwd`/db:/var/lib/mysql mysql:5.7.14
	sleep 20 # need to give time for mysql to start...
	sudo docker run -it --name mysql-cmd --rm --link mysql:db mysql:5.7.14 \
          mysql -uroot -p'$$3cureUS' -h db -e \
          "CREATE DATABASE cs4501 CHARACTER SET utf8; \
          CREATE DATABASE test_cs4501 CHARACTER SET utf8; \
          CREATE USER 'www'@'%' IDENTIFIED BY '\$$3cureUS'; \
          GRANT ALL PRIVILEGES ON *.* TO 'www'@'%';"

  # if you encounter ERROR 2003 (HY000): Can't connect to MySQL server on 'db' (111):
  # increase the sleep time. this *likely* occured because the db docker container 
  # did not have enough time to complete setting up a mysql community server.

clean:
	@echo "cleaning..." 
	@sudo docker stop mysql > /dev/null 2>&1 && \
	 sudo docker rm mysql > /dev/null 2>&1 ||:
	@sudo rm -rf db > /dev/null 2>&1 ||:
```

Running `make database` will build the target. `make clean` will remove the
database (useful if when you are using fixtures and need to reset your
database).


HTTP requests and JSON responses in Python
------------------------------------------
```Python
import urllib.request
import urllib.parse
import json

# make a GET request and parse the returned JSON
# # note, no timeouts, error handling or all the other things needed to do this
# for real                                                                                                                      
print ("About to do the GET...")
req = urllib.request.Request('http://jsonplaceholder.typicode.com/posts/1')
resp_json = urllib.request.urlopen(req).read().decode('utf-8')
resp = json.loads(resp_json)
print(resp)

# make a POST request.
# # we urlencode the dictionary of values we're passing up and then make the
# POST request
# # again, no error handling                                                                                                                                                                                  
print ("About to do the POST...")
post_data = {'title': 'Demo Post', 'body': 'This is a test', 'userId': 1}
post_encoded = urllib.parse.urlencode(post_data).encode('utf-8')
req = urllib.request.Request('http://jsonplaceholder.typicode.com/posts',
data=post_encoded, method='POST')
resp_json = urllib.request.urlopen(req).read().decode('utf-8')
resp = json.loads(resp_json)
print(resp)
```

Django Unit Tests 
-----------------
```Python
from django.test import TestCase, Client
from django.core.urlresolvers import reverse
from myapp.models import Order, User

class GetOrderDetailsTestCase(TestCase):
  def setUp(self):     #setUp method is called before each test in this class
     pass              #nothing to set up
  def success_response(self):
     response = self.client.get(reverse('all_orders_list',
kwargs={'user_id':1}))   #assumes user with id 1 is stored in db
     self.assertContains(response, 'order_list')  #checks that response contains
parameter order list & implicitly checks that
#statuscode is 200
  def fails_invalid(self):
     response = self.client.get(reverse('all_orders_list'))
     self.assertEquals(response.status_code, 404)    #user_id not given in url,
so error

  def tearDown(self):  #tearDown method is called after each test
     pass              #nothing to tear down
```
