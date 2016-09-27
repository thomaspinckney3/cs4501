Overview
========

In this phase of the project you will build your Django data models
and then build a first version of services for accessing those
models. There are two weeks assigned for completing this work, but if
you wait until the second week you will likely not finish on time.

Database Design
-------------

You should think about what your data models will be and how they
will relate to each other. For example, if you're building an application
for students to hire tutors, you might have models for users, tutors, tutees,
reviews etc. Users might have a relationships to tutors and tutees (can a user
tutor one subject and receive help in another?). You can also start to spec out
what fields will be in each model (e.g. first name, email address, etc.).

Django
------

The Django documentation is going to be invaluable and essential for
completing this work. I'm not going to provide step-by-step
instructions for how to build a Django app. I'm assuming you've done
it before and are using the tutorials and support provided on the Django site. We're
using Django 1.8.8, which is not the latest version on the Django
website. Make sure you're looking at the correct version!

A note on code layout. It should look something like this. This is generated using
the standard Django `startproject`, `startapp` commands.


	stuff/
	├── manage.py
	├── myapp
	│   ├── __init__.py
	│   ├── admin.py
	│   ├── migrations
	│   │   └── __init__.py
	│   ├── models.py
	│   ├── tests.py
	│   └── views.py
	└── stuff
	    ├── __init__.py
	    ├── settings.py
	    ├── urls.py
	    └── wsgi.py


It's probably worth your effort to set up the admin interface to make
it easy to update your data. In the real world you wouldn't do this,
but I think it's a reasonable short-cut for a class project. The risk
with this is that it could expose security vulnerabilities, so you
wouldn't really want to put it into production.

Docker Compose
--------------

One other thing to introduce before we start coding is Docker Compose. Compose
is a tool for defining the topology and running multi-container Docker
applications.  With Compose, you use a Compose file to configure your
application’s services.  Recall how we start the 'web' and 'mysql' containers
one by one in the previous project. This approach works fine when we have only
two containers (and the configuration is simple), but it is certainly not scalable.
Using Docker Compose, we will be able to start up our containers using a single
command. Here is a tutorial about how Docker Compose works:

First, install Docker Compose if you haven't already done so. You can check if
you have docker-compose on you machine by typing docker-compose in your
terminal.

	$ curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
	$ chmod +x /usr/local/bin/docker-compose


When you've got Compose installed, go ahead and create a new file,
docker-compose.yml, in the project root directory (you need to name the file
exactly as docker-compose.yml). Docker Compose uses these files, called YAML
files, to automatically set up or start a group of containers. This is handy,
much easier than managing each container individually. You can specify a lot of
different options here, but for now we're merely going to create our entity
layer and then link it to our mysql database. You need to make sure the mysql
container we created from last time is started.

You can tell compose to create a new container by giving it a name and an image to use.

	models:
	    image: tp33/django
	    external_links:
	      -  mysql:db
	    volumes:
	      - <project_root_dir>:/app
	    ports:
	      - "8001:8000"
	    command: bash -c "mod_wsgi-express start-server --working-directory <project_root_dir> --reload-on-changes <path_to_wsgi.py>/wsgi.py"

Notice the difference between external_links and links. We use links to link to
a container created by the docker-compose.yml. On the other hand, external_links
is used to link to a container not created by Compose. In this case, since we
are linking to a container "mysql" we created manually, we use external_links.
By specifying "mysql:db" we can refer to the "mysql" container in our "models"
container simply by "db" instead of refer to it by its IP Address.

Volumes is like the -v tag we use when executing docker run in the previous
project. It mounts the app directory in the container onto the <your_file_path>
directory on the host machine(your mac/PC). By specifing that, you can code in
your text editor/IDE on your host machine and any change you make will be picked
up by the container. It is handy in terms of the development workflow.

Ports expose the port in your container to the port on your host machine. In
this case, we are exposing port 8000 in the container to port 8001 on your host
machine. By exposing the ports, you can access your Django app in the browser on
your host machine by going to localhost:8001 if you use a native Docker app or
<your_docker_ip>:8001 if you use Docker Machine/Toolbox.

Command specifies the command that will be run when the container starts up. In
this case it will start the mod_wsgi server.

Also note you NEED to use relatvie paths in your docker compose file or it won't work on our machines. When you use relative path, keep in mind you are addressing path from docker-compose.yml's perpective.

Now that we've got this, save docker-compose.yml. Having Compose create and run our container is as simple as running

	docker-compose up

NOTE that for the command to run successfully, you need to have your mysql
container running.

	docker-compose rm

will remove all the instance specified by the docker-compose file.

One other thing about docker compose. If for some reasons (run makemigrations/migrate etc.) you need to attach to a container created by docker compose, instead of using `docker attach <container_name>`(it will hang), use `docker exec -it <container_name> bash`.

You can do a lot more with Docker Compose, including building new images straight from a Dockerfile, configuring ports, and defining shared volumes for containers. These are all things that may be helpful to you later in the course, so keep your compose file updated as you go about your project. There are great examples at https://docs.docker.com/compose/install/ and https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04

Models
------

The first step is to create your Django models for your project.

Generally, there are a few guidelines to consider:

  Remember, each model corresponds to a table in your database. Each
  model is represented in Django by a Python class. Each row in your
  table will correspond to an instance of your model class. Django
  will automatically create the database tables you need.`

  Every model should include a unique id. For example, if your project
  has things that can be sold, there'll likely be a Items model that
  represents the things to be sold. Each Item should have a unique id
  that you can record, for example, when creating the lists of Items a
  user has bought or sold.

  You need to think through the relationships between
  models. Continuing the example above, a single user may have bought
  many items so there is what is called a "one-to-many" relationship
  between Users and Items. On the other hand, if you have a user has
  written a Review for an Item, the Review will reference the Item id
  being reviewed. There will be exactly one Item that the Review
  references. This is known as a "one-to-one" relation.

  Users and Picture models are generally handled specially in
  Django. However, as we'll discuss in later classes, we're not
  going to use the Django User class. Pictures are complicated by the
  fact that they tend to be large. For now, I'd just ignore doing
  anything with pictures.

Docker
--------

For project 2, try and create a quick and simple pipeline for working with Docker. **An objective for project 2 is to show you that setting up to run code is just as important as writing the code.** Some of the key concepts you may want to look into are
* docker-compose
* docker volumes
* docker port forwarding
* modwsgi --reload-on-change flag

Services
--------

Each API will have its own url for accessing it. These are listed in
your project's urls.py file. Each one will specify the view that is to
be invoked to handle that url. They will look something like this:

    /api/v1/users/43 - GET to return info about user 43, POST to update the user's info.
    /api/v1/things/23 - GET to return info about thing 23, POST to update it
    /api/v1/things/create - POST to create a new thing

You'll then create a Django view for each url. The view may handle
both GET and POST requests. You'll need to consult the Django
documentation for how to do this and for how to properly format a json
response from your view.

The APIs should return JSON results. The POST methods should take either form-encoded
sets of key-value parameters (preferred) or JSON encoded values. For example, a result
from looking up a user might look like:

    {
     'ok':     True,
     'result':
     {
      'username':    'tpinckney',
      'first_name':  'Tom',
      'last_name':   'Pinckney',
      'date_created': 'Feb 12 2016'
     }
    }

Remember, this is a four-tier app we're building. The DB is the fourth
/ bottom tier. This layer of services you're building now is the
third tier. It should focus on providing access to creating, looking
up, and updating your models. The second tier, when you build it
later, will consume the third tier services and put them together to
form high level functions in your app like providing search, or a
home/start page/screen, etc. Finally the top tier will be an app that
consumes the second tier and generates HTML for a browser (or for
a native mobile app if you're so inclined).

Iterative design
----------------

You will not get your services and models exactly right. This is really just the
first draft. As you continue to build higher layers of your app in future
assignments you'll come back and change your models and services.

One important way that Django makes this easy is with database migrations. A
migration in Django is a set of schema changes that are required to move your DB
and app from one version to the next.

When you edit your models.py file(s), your DB will not immediately automatically
reflect the changes. Instead, you'll need to use your Django manage.py to
generate a set of SQL commands needed to update your DB to match your new
models. Then you can apply these commands to make them actually take affect.
Django breaks this into two stages so that you can check the commands into git
on your dev machine and then later apply them to many different db's -- in
theory you might have many dev db instances, some testing/qa instances and then
prod db instances. See the Django getting started or model documentation for
more on migrations and how to use them.

Fixtures
--------
Final thing before we call it a day, you need to add fixtures. You can think of
fixtures as initial data provided to an empty database. It’s sometimes (e.g. when
testing, grading) useful to pre-populate your database with hard-coded data when
you’re first setting up an app so that you can use the app directly. To avoid
hard-coding test data everytime you have a clean database, we use Django
fixtures which does the work for you automatically.

Here's a quick introduction of fixture and a tutorial on how to use fixtures to
preload data or export your db:

General steps for creating fixtures:

Dump existing db data to db.json (JSON file name does not matter) using

	python manage.py dumpdata > db.json

See Django documentation for the various options for dumpdata: https://docs.djangoproject.com/en/1.9/ref/django-admin/#dumpdata

This is an example fixture. You can see it is basically Django model instances
serialized into JSON format.

	[
	  {
	    "model": "myapp.person",
	    "pk": 1,
	    "fields": {
	      "first_name": "John",
	      "last_name": "Lennon"
	    }
	  },
	  {
	    "model": "myapp.person",
	    "pk": 2,
	    "fields": {
	      "first_name": "Paul",
	      "last_name": "McCartney"
	    }
	  }
	]

To use your fixture to pre-populate a **clean** database instance:

Run command

	python manage.py loaddata db.json

The command will deserialize the models instances and loaded them into your
database.

See Django documentation for the various options for loaddata:
https://docs.djangoproject.com/en/1.9/howto/initial-data/

Initial data should now be in db!

To incorporate fixtures into your project submissions, add some data to your
database and create a fixture before pushing to github and add “python manage.py
loaddata” command to your docker-compose command to load the data before
starting your server.

What to turn in
---------------

The teaching staff should be able to run your app entirely by using docker-compose up. You can assume we have a clean mysql database called 'cs4501' with user 'www' of password '$3cureUS' (as we configured in project1) for your models container to hook up to. For this project, we would expect your current iteration to have serveral GET APIs and serveral POST APIs.

Something that need to be executed by the docker-compose.yml (some of them may be repetitions from the previous section):

* run makemigrations (I strongly discourage you from committing your mirgation files. Why? Come to office hours and we can discuss!)
* run migrate
* load fixtures
* start the wsgi server
* (You can do something like `command: bash -c "<command_1> && <command_2> && ..."`)
* expose ports
* link to the mysql db container

Also note that you NEED to use relative path for your compose file to work on our machines (we won't have the same absolute path as yours)! Be extremely careful when you set up the relative path.

You'll send us the link to your github tag. We'll checkout the code, run docker-compose up and expect things to run. Again, we strongly encourage you to take time to demo in the office hours. We want to make sure not only you are writing code that works but also code that is of best practices.
