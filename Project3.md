Overview
========

In this part of the project you will develop your experience service
API layer and an HTML front-end. This project will build on top of the
work you've done in Project 1 and Project 2.

Architecture
------------

As a reminder, we are building a four-tier web app. In decreasing order, these
are

   - the HTML front end
   - the experience service APIs
   - the Entity / Model APIs (started in Project 2)
   - the database (started in Project 2)

Each layer will run in its own container and communicate via the
network with the other layers.

An example request will be processed as follows:

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
   service calls a mobile client has to make.
   - Moving all the business logic for each screen or page to the experience
   service ensures that different mobile and web front-ends have a consistent
   experience for users.

The key point of this is the strict isolation between levels. The only
way the HTML front-end communicates with the rest of your app is
through the experience service level. In turn, the only way your
experience service level interacts with the database is through the
entity API. This is directly analogous to the data abstraction and
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
your marketplace is about: rides, books, tickets---whatever relevant
models your app uses. The home page should show links to the detail
pages -- maybe the newest content, the most popular content, etc.

Note: These are read-only pages for showing data in the db. To simplify
this project we're ignoring how users or things in your marketplace are
created in the first place. You can test your project using fixtures or by using
your entity API to create users and things.
In later projects we'll add the flows for letting users sign up and add
things/content to your app.

If you have time in this project, you should add other useful read-only pages
like a user
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
Each feature that you code from this project onwards should correspond to a user story. Each user story should have a documented set of unit tests that validate the user story is correctly implemented (acceptance criteria). For this project, create at least 5 user stories, but no more than 20. Turn in your user stories as part of your project via your [repository's Wiki page](https://help.github.com/articles/about-github-wikis/).

### Unit Testing ###

[Django's unit tests](https://docs.djangoproject.com/en/2.1/topics/testing/overview/) use the Python unit testing framework ```unitest```. To run your unit tests, execute ```python manage.py test``` from the command line. By default, this will run all files in your project named ```test*.py``` where * is a wildcard (ex. test_users.py).

Django creates and destroys a temporary test database every test run. To be able to grant the user access to the database, we will need to grant permission for user 'www' like in Project 1. Permission can be granted without the database existing yet.

```mysql> grant all on test_cs4501.* to 'www'@'%';```

Here are some examples of unit tests for your Django views:

```Python
    from django.test import TestCase, Client
    from django.urls import reverse
    from myapp.models import Order, User

    class GetOrderDetailsTestCase(TestCase):
        #setUp method is called before each test in this class
        def setUp(self):
            pass #nothing to set up

        def success_response(self):
            #assumes user with id 1 is stored in db
            response = self.client.get(reverse('all_orders_list', kwargs={'user_id':1}))

            #checks that response contains parameter order list & implicitly
            # checks that the HTTP status code is 200
            self.assertContains(response, 'order_list')

        #user_id not given in url, so error
        def fails_invalid(self):
            response = self.client.get(reverse('all_orders_list'))
            self.assertEquals(response.status_code, 404)

        #tearDown method is called after each test
        def tearDown(self):
            pass #nothing to tear down
```

Note: Django specifies that "Tests that require a database (namely, model tests) will not use your “real” (production) database. Separate, blank databases are created for the tests. Regardless of whether the tests pass or fail, the test databases are destroyed when all the tests have been executed."
If you are creating tests in your experience layer, then a test database will not work because the model layer, not the experience layer, is the one that interacts directly with the database. Therefore, in later projects, if you find you are unable to test a particular function through unit tests (like database deletions or saves), that is okay. All read-only data (Project 3) should be testable, as well as the API CRUD functionalities built in Project 2. Therefore, unit tests in the models layer is required for this project.

Also see [Django Advanced Testing](https://docs.djangoproject.com/en/2.1/topics/testing/advanced/) if interested!


Implementation
--------------

### Container Linking ###

You will have four Docker containers running -- one for each layer in
your app: one instance of the MySQL container and three instances of
your Django container. Keep in mind that in order to execute commands like
`python manage.py test`, you must be inside the container (not host). For
example, running `docker exec -it models bash` opens a bash terminal inside
the container named "models".

In terms of grading, since you can assume we have a MySQL container
`mysql` running, you only need to start up three containers in the
docker-compose.yml, all of which link to the MySQL container.

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
container created by the `docker-compose.yml`. On the other hand, `external_links` is used to
link to a container outside `docker-compose`. In this case, since we are linking to a container outside
`docker-compose`, we use `external_links`.

That option will create a hostname called `db` and make sure it's
always pointing to the IP address for the container named
`mysql` in `models` container. Thus, your app in this container can always connect to the
host `db` instead of having to know which IP address your MySQL
container is actually running as. This is how you set up your
project's `settings.py` so far.

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
host model-api in order to connect to your model-api container by making a request like so:

`req = urllib.request.Request('http://models-api:8000/')`

Note: You can only use the url 'http://models-api:8000' in code run in a container that links to the model container using 'models-api' like in the example above. If you are testing outside the container (like in a browser), you will have to use 'http://localhost:port'.

And finally, your third container for running the HTML front-end will
link to the experience service container:

```YAML
web:
   image: tp33/django
   links:
      - exp:exp-api
```

I do a few other things to ease my development:

- I expose each container's port 8000 into my Linux VM with each container exposed as a different port. This is accomplished via adding docker compose `ports` option, for example:

```YAML
models:
   image: tp33/django
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
```

   exposes the port 8000 in the container (which is the `mod_wsgi-express` default port)
   to port 8001 on the host machine. In this way you can access your models layer by listen to
   `http://localhost:8001` using a browser in the host machine. The exposure of ports and why it is necessary is a critical concept to understand, if you need more help understanding it, please come to office hours.

- I mount the source for each container from my Linux VM so that I can edit code in Linux and have each container pick up the changes immediately (update the last modified time on wsgi.py in the top of your Django app to tell Apache to reload your app -- can do this via 'touch wsgi.py'). This is accomplished by docker compose `volumes` option:

```YAML
models:
   image: tp33/django
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - <your_file_path>:/app
```

mounts the file directories inside `<your_file_path>` onto the `/app` directory in the container.

- Docker containers exit when their main process finishes. To prevent containers
  from immediate exit, I tell each container to run mod_wsgi-express on startup.
  If I want to interactively log into the container I later run `docker exec -it
  name /bin/bash` where name is the container name I want to start a shell in.
  This is accomplished by docker compose `command` option:
```YAML
models:
   image: tp33/django
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - <your_file_path>:/app
   command: "mod_wsgi-express start-server --reload-on-changes <project_name>/wsgi.py"
```

Putting that all together into the docker-compose.yml will start my three containers for me (I manually start the mysql container).

A sample docker-compose.yml (you will have to modify the code accordingly to match your configuration):

```YAML
models:
   image: tp33/django
   external_links:
      - mysql:db
   ports:
      - "8001:8000"
   volumes:
      - /home/tp/stuff-models:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-models/wsgi.py"

exp:
   image: tp33/django
   links:
      - models:models-api
   ports:
      - "8002:8000"
   volumes:
      - /home/tp/stuff-exp:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-exp/wsgi.py"

web:
   image: tp33/django
   links:
      - exp:exp-api
   ports:
      - "8000:8000"
   volumes:
      - /home/tp/stuff-web:/app
   command: "mod_wsgi-express start-server --reload-on-changes stuff-web/wsgi.py"
```

This results in my running container set looking like:

    tp@devel:~$ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
    5d4da12058de        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute 0.0.0.0:8000->8000/tcp   web
    e9f08748b67f        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute 0.0.0.0:8002->8000/tcp   exp
    5ef6412cc321        tp33/django:1.2     "mod_wsgi-express sta"   About a minute ago   Up About a minute   0.0.0.0:8001->8000/tcp   models
    5b18a2deae1a        mysql:5.7.8         "/entrypoint.sh mysql"   4 weeks ago          Up 40 minutes       3306/tcp                 mysql

- My low level API is running in a container called 'models' and is listening on port 8000 (which is exposed as port 8001).
- My experience service API is running in a container called 'exp' and is listening on port 8000 (which is exposed as port 8002).
- My web interface is running in a container called 'web' and is listening on port 8000 (which is exposed as port 8000).

### HTML ###

The HTML front-end app will render data via Django templates to
produce HTML. The external CSS library Bootstrap will be used for
styling the HTML.

### Code layout ###

You should be thinking of your app as three separate sub-apps: the
entity API, the experience service API and the HTML
front-end. The best way to do this is to create three separate Django
projects, each in their own directories of one git repository. Note, however,
that only the entity/model API app will be configured to talk to the DB.

Using multiple projects will allow you to have three separate settings.py
files. This will be important because you'll want to do things
differently in your HTML front-end since it's the part the public will
be able to access and it's the part that's serving HTML as opposed to
JSON. For example, you'll want to enable things like CSRF-protection
middleware in the HTML front-end while that is not appropriate for the
other tiers of your web app.

### Development process ###

If you're not careful and methodical, you'll quickly end up with a
mess. I recommend you do the following:

- Design your web interface first -- sketch on a piece of paper what
  info you want to show, where it will be relative to the other
  content, etc. -- this will help you to brainstorm and create user stories

- For implementation, start with building the web interface tier first to define what your
  experience services need to return. Then build the experience
  service to provide the data your web interface needs.

- Develop and test each tier independently. That means that if you
  make changes to your entity API, test it carefully using unit tests before moving
  content on to changes in the experience services tier. You don't want to
  change every tier and then try things and not know why things are
  failing.

- Start early because if you wait until a few days before the
  assignment is due you will fail to finish it.

- Ask questions to get help from your teammates and from me.

- Check your code in to your GitHub repository in small chunks instead
  of giant commits. Make one logical change, test it, get a teammate to review it, and then
  commit it. If you commit lots of small changes and you later find
  something doesn't work, it's easier to go through the small changes
  to see what broke things as opposed to having to go through a huge
  set of changes all in one commit.

- Use Django Fixtures to reproducibly load test data into your databases.

### Calling HTTP/JSON APIs in Python ###

```Python
    import urllib.request
    import urllib.parse
    import json

    # make a GET request and parse the returned JSON
    # note, no timeouts, error handling or all the other things needed to do this for real
    print ("About to perform the GET request...")

    req = urllib.request.Request('http://placeholder.com/v1/api/posts/1')

    resp_json = urllib.request.urlopen(req).read().decode('utf-8')
    resp = json.loads(resp_json)

    print(resp)

    # make a POST request.
    # we urlencode the dictionary of values we're passing up and then make the POST request
    # again, no error handling

    print ("About to perform the POST request...")

    post_data = {'title': 'Demo Post', 'body': 'This is a test', 'userId': 1}

    post_encoded = urllib.parse.urlencode(post_data).encode('utf-8')

    req = urllib.request.Request('http://placeholder.com/v1/api/posts/create', data=post_encoded, method='POST')
    resp_json = urllib.request.urlopen(req).read().decode('utf-8')

    resp = json.loads(resp_json)
    print(resp)
```

### What to turn in ###

once you are ready to submit, create a Github Release, and email Zakey (zaf2xk) the link to the release - please make the subject of the email "ISA Project 3 Submission". Make sure you've tested
your code thoroughly and haven't forgotten to commit anything. Also make sure to
test your full stack from a clean/empty/new database so that you're not
accidentally depending on anything already in your system.

You are required to turn in a `docker-compose.yml`, a set of fixtures, user
stories, and unit tests that allow your TA to easily run/test your system and
verify that the basic functionality works. Your `docker-compose.yml` should
assume, like in the prior assignment, that there's a `mysql` container already
running and it should be referenced via an `external_link`. Your fixtures should
contain sample test data for viewing the listing page. Include your user stories in the github wiki
and make sure your unit tests attempt to correspond to the user stories.
