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
for students to hire turors, you might have models for users, tutors, tutees,
reviews etc. Users might have a relationships to tutors and tutees (can a user
tutor one subject and receive help in another?). You can also start to spec out
what fields will be in each model (eg first name, email address etc).

Django
------

The Django documentation is going to be invaluable and essential for
completing this work. I'm not going to provide step-by-step
instructions for how to build a Django app. I'm assuming you've done
it before and are using the tutorials etc on the Django site. We're
using Django 1.8.8 which is not the latest version on the Django
website. Make sure you're looking at the correct version!

A note on code layout. I prefer to have my projects laid out like
this:


	stuff/
	├── manage.py
	└── stuff
	    ├── __init__.py
	    ├── main.py
	    ├── models.py
	    ├── settings.py
	    ├── templates
	    │   └── start.html
	    ├── urls.py
	    └── wsgi.py



This is different the defualt Django layout that is created if run
'startproject' and then 'startapp'. I basically skip the 'startapp'
step and just put everything in the same directory.

It's probably worth your effort to set up the admin interface to make
it easy to update your data. In the real world you wouldn't do this,
but I think it's a reasonable short-cut for a class project. The risk
with this is that it could expose security vulnerabilities so you
wouldn't want to really put it into production.

Docker Compose
--------------

One other thing to introduce before we start coding is Docker Compose.
Compose is a tool for defining the topology and running multi-container Docker applications. 
With Compose, you use a Compose file to configure your application’s services.
Recall how we start web, mysql containers one by one in the previous project. This
approach works fine when we have only two containers and the configuration is simple, but
is certainly not scalable. Using Docker Compose, we will be able to stand up our containers
using a single command. Here is a tutorial about how Docker Compose works:

First, install Docker Compose by doing

	$ curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
	$ chmod +x /usr/local/bin/docker-compose


When you've got Compose installed, create a new directory with

	mkdir Compose
	cd Compose

Then go ahead and create a new file, docker-compose.yml, with your favorite text editor. Docker Compose uses these files, called YAML files, to automatically set up or start a group of containers. This is handy, much easier than managing each container individually. You can specify a lot of different options here, but for now we're merely going to create our entity layer and then link it to our mysql database. You need to make sure the mysql container we created from last time is started.

You can tell compose to create a new container by giving it a name and an image to use.

	models:
	    image: tp33/django:1.3
	    external_links:
	      -  mysql:db
	    volumes:
	      - <your_file_path>:/app
	    ports:
	      - "8001:8000"
	    command: bash -c "mod_wsgi-express start-server --reload-on-changes stuff-models/wsgi.py"

Notice the difference between external_links and links. We use links to link to a container created by the docker-compose.yml. On the other hand, external_links is used to link to a container outside Compose. In this case, since we are linking to a container outside Compose, we use external_links. 

Volumes is like the -v tag we use when executing docker run. It mounts the app directory in the container onto the <your_file_path> directory on the host machine(your mac/PC). By specifing that, you can code in your text editor/IDE on your host machine and any change you make will be picked up by the container.

Ports expose the port in your container to the port on your host machine. In this case, we are exposing port 8000 in the container to port 8001 on your host machine. By exposing the ports, you can access you Django app in the browser by going to localhost:8001 if you use a native Docker app or <your_docker_ip>:8001 if you use Docker Machine/Toolbox.

Command specifies the command that will be run when the container starts up. In this case it will start the mod_wsgi server.

Now that we've got this, save docker-compose.yml. Having Compose create and run our container is as simple as running

	docker-compose up

in our compose directory.

	docker-compose rm

will remove all the instance specified by the docker-compose file.

This creates a new container based off the tp33/django:1.3 image, and links it to the mysql container with the hostname of "db". This is the name we'll use when we configure our settings.py file.

You can do a lot more with Docker Compose, including building new images straight from a Dockerfile, configuring ports, and defining shared volumes for containers. These are all things that may be helpful to you later in the course, so keep your compose file updated as you go about your project. There are great examples at https://docs.docker.com/compose/install/ and https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-14-04

Models
------

The first step is to create your Django models for your project.

Generally, there are a few guidelines to consider:

  Remember, each model corresponds to a table in your database. Each
  model is represented in Django by a Python class. Each row in your
  table will correspond to an instance of your model class. Django
  will automatically create the database tables you need.

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

For project 2, try and create a quick and simple pipeline for working with Docker. An objective for project 2 is to show you that setting up to run code is just as important as writing the code. Some of the key concepts you may want to look into are
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

You will not get your services and models exactly right. This is really just
the first draft. As you continue to build higher layers of your app in future assignments you'll
come back and change your models and services.

One important way that Django makes this easy is with database migrations. A migration in Django
is a set of schema changes that are required to move your DB and app from one version to the next.

When you edit your models.py file(s), your DB will not immediately automatically reflect the changes. Instead,
you'll need to use your Django manage.py to generate a set of SQL commands needed to update your DB to match
your new models. Then you can apply these commands to make them actually take affect. Django breaks this into two
stages so that you can check the commands into git on your dev machine and then later apply them to many different db's -- in theory you might have many dev db instances, some testing/qa instances and then prod db instances. See the Django getting started or model documentation for more on migrations and how to use them.

Fixtures
--------
Final thing before we call it a day, you need to add fixtures so that we are provided with initial data when grading your project.

Here's a quick explanation on how to use fixtures to preload data or export your db:

General Steps for creating fixtures:

Dump existing db data using 

	python manage.py dumpdata > db.json

See Django documentation for the various options for dumpdata: https://docs.djangoproject.com/en/1.9/ref/django-admin/#dumpdata

The command saves the output of dumpdata to db.json.

To use your fixture:
Run command 
	python manage.py loaddata db.json

See Django documentation for the various options for loaddata:
https://docs.djangoproject.com/en/1.9/howto/initial-data/

Initial data should now be in db!

To incorporate fixtures into your project submissions, add some data to your database and create a fixture before pushing to github and add “python manage.py loaddata” command to your docker-compose command to load the data before starting your server.

What to turn in
---------------

The teaching staff should be able to run your app entirely by using docker-compose up. You can assume we have a clean mysql database called 'cs4501' with user 'www' with password '$3cureUS' (as we configured in project1). You'll send us the link to your github tag. We'll checkout the code, run docker-compose up and expect things to run.
