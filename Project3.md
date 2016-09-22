Overview
========

In this part of the project you will develop your experience service
API layer and an HTML front-end. This project will build on top of the
work you've done in Project 1 and Project 2.

Architecture
------------

As a reminder, we are building a four-tier web app:

   - HTML front end
   - Experience service APIs
   - Entity / Model APIs (started in Project 2)
   - Database (started in Project 2)

Each layer will run in it's own container and communicate via the
network with the other layers.

An example request will be processed like:

   - A request from a user's browser will go to the HTML front end.
   - The front-end will call the experience service to get the data needed and render the data as HTML.
   - The experience service will call one or more of the entity APIs to get it's data and return it.
   - The entity APIs will call the database to read data and update models.

The experience service app will invoke the entity APIs via HTTP and
receive JSON responses. Similarly, it will provide HTTP/JSON APIs up
to the HTML front-end app. Note, an end-user only ever access the HTML
front-end app. All the other parts are hidden and not publicly
accessible for security reasons.

Recall that there are several reasons for creating an experience service level:

   - Creating a single service call to power each page/screen reduces how many
   service calls a mobile client has to make
   - Moving all the business logic for each screen or page to the expereicen
   service ensures that different mobile and web front-ends have a consistent
   experience for users.

The key point of this is the strict isolation between levels. The only
way the HTML front-end communicates with the rest of your app is
through the expereince service level. In turn, the only way your
experience service level interacts with the database is through the
entity API. This is directly analagous to the data abstraction and
modularity that you learn about applying in an individual program, but
here it is applied to a system of programs.

You should assume that the web front-end and the experience service tiers
are accessible to the world but the entity and database tiers are internal/behind
a firewall. That is, anyone may call your web front-end or experience service but
only your experience service can call your entity and database tiers. This is 
important to think through when it comes to ensuring the security of your application.

This will seem overly confusing and burdensome for a simple
app. However, for a large app with many teams of people working on it
it provides isolation between teams and apps.

Pages
-----

The only required pages for this project are a home page and an item
detail page. The detail page should show details about whatever it is
your marketplace is about -- rides, books, tickets. The home page
should show links to the detail pages -- maybe the newest content, the
most popular content, etc.

Note, these are read-only pages for showing data in the db. To simplify
this project we're ignoring how users or things in your marketplace are
created in the first place. You can test your project using fixtures or by using your entity API to create users and things.
In later projects we'll add the flows for letting users sign up and add
things/content to your app.

If you have time in this project, you should add other useful read-only pages like a user
profile page to show info about buyers / sellers. Later on, we'll add
a search page powered by a real search engine. For now you can add a
simple / dummy search page if you like that just shows all content in
your system.

Don't worry too much about the design of your pages for now. Bootstrap
will provide decent default styling. However, if you want, you can
think about how to improve the design or make it responsive for
smaller screens.

User Stories and Unit Testing
-----------------------------

Before starting to write any code for Project 3, start with developing a set of user stories. 
Each feature that you code from this project onwards should correspond to a user story. Each user story should have a documented set of unit tests that validate the user story is correctly implemented (acceptance criteria). For this project, create at least 5 user stories, but no more than 20. Turn in your user stories as part of your project.

### Unit Testing ###

[Django's unit tests](https://docs.djangoproject.com/en/1.9/topics/testing/overview/) use the Python unit testing framework ```unitest```. To run your unit tests, execute ```python manage.py test``` from the command line. By default, this will run all files in your project named ```test*.py``` where * is a wildcard (ex. test_users.py).

Here are some examples of unit tests for your Django views:

    from django.test import TestCase, Client
    from django.core.urlresolvers import reverse
    from myapp.models import Order, User
    
    class GetOrderDetailsTestCase(TestCase):
      def setUp(self):     #setUp method is called before each test in this class
         pass              #nothing to set up
      def success_response(self):
         response = self.client.get(reverse('all_orders_list', kwargs={'user_id':1}))   #assumes user with id 1 is stored in db
         self.assertContains(response, 'order_list')  #checks that response contains parameter order list & implicitly checks that                                                 #statuscode is 200
      def fails_invalid(self):
         response = self.client.get(reverse('all_orders_list'))
         self.assertEquals(response.status_code, 404)    #user_id not given in url, so error
         
      def tearDown(self):  #tearDown method is called after each test
         pass              #nothing to tear down
      

Note: Django specifies that "Tests that require a database (namely, model tests) will not use your “real” (production) database. Separate, blank databases are created for the tests. Regardless of whether the tests pass or fail, the test databases are destroyed when all the tests have been executed."
If you are creating tests in your experience layer, then a test database will not work because the model layer, not the experience layer, is the one that interacts directly with the database. Therefore, in later projects, if you find you are unable to test a particular function through unit tests (like databse deletions or saves), that is okay.  All read-only data (Project 3) should be testable, though.  

Also see [Django Advanced Testing](https://docs.djangoproject.com/en/1.9/topics/testing/advanced/) if interested!


Implementation
--------------
Part of this section will be repetitions from the previous project, but it will still be beneficial to read along since more detailed explanation is included.

### Container Linking  ###

You will have four Docker containers running -- one for each layer in
your app: one instance of the MySQL container and three instances of
your Django container. In terms of grading, since you can assume we 
have a MySQL container `mysql` running, you only need to start up three containers
in the docker-compose.yml, all of which link to the MySQL container.

Docker assigns a unique IP address to every container running. We'll
use container linking to make sure each container knows the IP address
of the other containers to talk to. In addition, we want to link to a container
that is not created by the compose. This is accomplished via adding
the `external_links` or `links` option to the docker compose file:

```YAML
models:
   image: tp33/django
   external_links:
      - mysql:db
```
   
As a reminder: Notice the difference between `external_links` and `links`. We use `links` to link to a
container created by the docker-compose.yml. On the other hand, `external_links` is used to 
link to a container outside Compose. In this case, since we are linking to a container outside
Compose, we use `external_links`.

That option will create a hostname called `db` and make sure it's
always pointing to the IP address for the container named
`mysql` in `models` container. Thus, your app in this container can always connect to the
host `db` instead of having to know which IP address your MySQL
container is actually running as. This is how you set up your
project's settings.py so far.

Similarly, you'll add another container for your experience service
and link it to your low-level API (notice the change from `external_links` to `links`):

```YAML
exp:
   image: tp33/django
   links:
      - models:models-api
```

In this case we are linking to a container created by docker compose, so
we use `links`. Then the app running in this container can make HTTP requests to the
host model-api in order to conncect to your model-api container.

And finally, your third container for running the HTML front-end will
link to the experience service container:

```YAML
exp:
   image: tp33/django:
   links:
      - exp:exp-api
```

I do a few other things to ease my development:

- I expose each container's port 8000 into my host machine with each container exposed as a different port. This is accomplished via adding docker compose `ports` option, for example:

```YAML
models:
   image: tp33/django:
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
```
   
   exposes the port 8000 in the container (which is the `mod_wsgi-express` default port) 
   to port 8001 on the host machine. In this way you can access your models layer by listen to 
   `http://localhost:8001` using a browser in the host machine.
- I mount the source for each container from my Linux VM so that I can edit code in Linux and have each container pick up the changes immediately. This is accomplished by docker compose `volumes` option:

```YAML
models:
   image: tp33/django:1.2
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - <your_file_path>:/app
```

mounts the file directories inside `<your_file_path>` onto the `/app` directory in the container.

- Docker containers exit when their main process finishes. To prevent containers from immediate exit, I tell each container to run mod_wsgi-express on startup. If I want to interactively log into the container I later run 'docker exec -it name /bin/bash' where name is the container name I want to start a shell in. This is accomplished by docker compose `command` option:
```YAML
models:
   image: tp33/django:
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - <your_file_path>:/app
   command: "mod_wsgi-express start-server --reload-on-changes <project_name>/wsgi.py"
```

Putting that all together into the docker-compose.yml will start my three containers for me (I manually start the mysql container).

A sample docker-compose.yml (you will have to modify the code accordingly to match you configuration):

```YAML
models:
   image: tp33/django:
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - /home/tp/stuff-models:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-models/wsgi.py"
   
exp:
   image: tp33/django:
   links:
      - models:models-api
   ports:
      - "8002:8000"
   volumes:
      - /home/tp/stuff-exp:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-exp/wsgi.py"
   
web:
   image: tp33/django:
   links:
      - exp:exp-api
   ports:
      - "8000:8000"
   volumes:
      - /home/tp/stuff-web:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-web/wsgi.py"
```
   
This results in my running container set looking like (You will have a different tag for the tp33 docker image):

    tp@devel:~$ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
    5d4da12058de        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute 0.0.0.0:8000->8000/tcp   web
    e9f08748b67f        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute 0.0.0.0:8002->8000/tcp   exp
    5ef6412cc321        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute   0.0.0.0:8001->8000/tcp   models
    5b18a2deae1a        mysql:5.7.8         "/entrypoint.sh mysql"   4 weeks ago          Up 40 minutes       3306/tcp                 mysql

- My low level API is running in a contianer called 'models' and is listening on port 8000 (which is exposed as port 8001).
- My experience service API is running in a container called 'exp' and is listening on port 8000 (which is exposed as port 8002).
- My web interface is running in a container called 'web' and is listening on port 8000 (which is exposed as port 8000).

### HTML ###

The HTML front-end app will render data via Django templates to
produce HTML. The external CSS library Boostrap will be used for
styling the HTML.

### Code layout ###

You should be thinking of your app as three separate sub-apps: the
entity API, the experience service API and the HTML
front-end. The best way to do this is to create three separate Django
projects, each in their own directories of one git repository. Note, however,
that only the entity / model API app will be configured to talk to the DB.

Using multiple projects will allow you to have three separate settings.py
files. This will be imporant because you'll want to do things
differently in your HTML front-end since it's the part the public will
be able to access and it's the part that's serving HTML as opposed to
JSON. For example, you'll want to enable things like CSRF-protection
middleware in the HTML front-end while that is not appropriate for the
other tiers of your web app.

### Development process ###

If you're not careful and mehthodical, you'll quickly end up with a
mess. I recommend you do the following:

- design your web interface first -- sketch on a piece of paper what
  info you want to show, where it will be relative to the other
  content etc. -- this will help you to brainstorm and create user stories

- for implementation, start with building the web interface tier first to define what your
  experience services need to return. Then build the experience
  service to provide the data your web interface needs.

- develop and test each tier independently. That means that if you
  make changes to your entity API, test it carefully with unit tests before moving
  on to changes in the experience services tier. You don't want to
  change every tier and then try things and not know why things are
  failing.

- start early because if you wait until a few days before the
  assignment is due you will fail to finish it.

- ask questions to get help from your teammates and from me.

- check your code in as small chunks instead of giant commits. Make
  one logical change, test it, get a teammate to review it, and then
  commit it. If you commit lots of small changes and you later find
  something doesn't work, it's easier to go through the small changes
  to see what broke things as opposed to having to go through a huge
  set of changes all in one commit.

- use Django Fixtures to reproducibly load test data into your databases.

### Calling HTTP/JSON APIs in Python ###

Python requests is a HTTP library with the mission of "HTTP for Humans". It is easier to use than the 
standard python urllib/urllib2 packages. There is no need to encode/decode HTTP content, explicitly deserialize json, etc.
Everything just "works" elegantly! However, since it is not a python 
standard package (and it is not included in the tp33/django image we are using.), you need to either define your own 
DockerFile on top of Professor's tp33/django or install requests on the fly when the container starts up.
Come to office hours if you want to explore the two options!

Here is a example snippet if you are not already familiar with requests. Note how easy it is to fire HTTP requests. 
You can read more about requests here http://docs.python-requests.org/en/master/

      import requests
      
      # make a GET request and parse the returned JSON                                                                                                                     
      print ("About to do the GET...")
      get_req = requests.get('http://jsonplaceholder.typicode.com/posts/1')
      # this is the deserialized json object. It is THAT SIMPLE!
      resp_json = get_req.json()
      print(resp_json)
      
      # make a POST request.
      print ("About to do the POST...")
      post_data = {'title': 'Demo Post', 'body': 'This is a test', 'userId': 1}
      post_req = requests.post('http://jsonplaceholder.typicode.com/posts', data=post_data)
      # again this is the deserialized json object.
      resp_json = post_req.json()
      print(resp_json)

### What to turn in ###

Create a tag in your GitHub and email your TA the tag. Make sure you've tested your code thoroughly and haven't forgotten to comit anything. Also make sure to test from a clean / empty / new database so that you're not accidentally depending on anything already in your system.

You are required to turn in a docker-compose.yml, a set of fixtures, user stories, and unit tests that allow your TA to easily run/test your system and verify that the basic functionality works. Your docker-compose.yml should assume, like in the prior assignment, that there's a mysql container already running and it should be referenced via an external_link. Your fixtures should contain sample test data for viewing the listing page. Include your user stories and which unit tests they correspond to.
