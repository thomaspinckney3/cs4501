Overview
=========

In this project you will set up a development environment (using
VirtualBox, Ubuntu Linux, Docker, Python, Django and MySQL) and then
create a simple Hello World app in Django. Whew!

Linux
-----

You will need some basic knowledge of Linux and sys admin
knowledge. You'll be doing basic things like install new software,
running SSH, looking at Apache log files, etc. While you don't need to
be a shell scripting god you should also be able to get the gist of a
script if need be.

VirtualBox
----------

If you're using a Mac or Windows machine, you'll need to install
VirtualBox and Ubuntu Linux in a VM. You can't do the projects
natively on anything other than Linux due to the use of Docker.

Containers
----------

Docker is a system for managing Linux containers. You can read more
about them elsewhere, but they're like a virtual machione but lighter
weight. They're one component of treating infrastructure as
software. You can define what kind of environment you need, install
apps, and then easily clone/start as many instances of that container
as you like -- on your dev machine or on public hosting providers like
Amazon AWS.

You'll be using two containers initially -- one that contains an
Apache/Django installation and another that contains MySQL. 

Django
------

You should already be familiar with Python and Django. If not, go read more
here https://docs.djangoproject.com/en/1.8/

MySQL
-----

The database used in this class will be MySQL. It's a very standard
(at this point) SQL database. You can read more at
http://dev.mysql.com/doc/refman/5.7/en/index.html


Install everything
===================

- Install VirtualBox from https://www.virtualbox.org

- Download the Ubuntu Server 15.04 ISO to install from
  http://www.ubuntu.com/download/server

- Start VirtualBox, create a new VM (1gig RAM and 8gigs disk is fine),
  selecting the Ubuntu ISO under Storage for the optical drive.

- Start the VirtualBox VM which should boot form the Ubuntu ISO. Go
  through the install process. Make sure to install the SSH server so
  you can log into your VM.

- Install Docker following the instructions
  https://docs.docker.com/installation/ubuntulinux/

- Pull down the Docker containers for Django and MySQL:
  - docker pull tp33/django:1.0
  - docker pull mysql:5.7.8

- Test it
  - docker run -it --name web tp33/django:1.0
  - (inside your Docker container now)
    - python
    - import django
    - django.VERSION
    - (you should see that you're using version 1.7.X)
    - Ctrl-D (to exit python)
    - exit (to exit your Docker container and return to your Linux VM shell)

- Initialize the MySQL db's
  - pick a password for your admin MySQL account (replace below where it says XXX)
  - docker run --name mysql -d -e MYSQL_ROOT_PASSWORD='XXX' mysql:5.7.8 
  
- Note the status of your Docker containers. The one named web will be shown as exited while the one named mysql will be running:
  - docker ps -a

Congratulations -- you should have a Linux VM with two docker
containers, one for running Django web apps and the other for running
MySQL.

Development Environment
====================

You've installed your development environment in the prior steps. Over
time you'll replicate this environment to create testing (QA)
environments and eventually a production (public) environment. Using
modular containers will make this very easy. For now, though, you'll
be doing all your work in the dev environment you just set up.

Your app is built using two containers: a web app container (running
Apache/Django) and a MySQL container. It's highly recommended that you
NOT make any changes (like installing new packages) beyond writing
code in your web app container. If you want to make other changes I
recommend creating your own Dockerfile and building a new image. This
will be more reproducible and automated and save you time in the
future.

You'll use GitHub for version control. If you don't have a GitHub
account already, go create one. There's good online help for using
GitHub and Git.

Working with containers
-----------------------

Run 'docker ps -a' and you'll see that you have two containers (the
ones you started above when testing your install) named web and
mysql. These are your dev environment containers. You'll do your
coding and initial testing in these containers. Run 'docker rm web' to
clean up the (stopped) web container you started earlier when testing
your install. Go ahead and stop your mysql container with 'docker stop
mysql'.

Docker provides a simple way for containers to talk to each other via
private networks. When you start a container you can tell Docker to
create an /etc/hosts entry for another running container. We'll do
this so that the web container can easily connect to the mysql
container. Let's restart the web container this way: 'docker run -it
--link mysql:db tp33/django:1.0'. This tells Docker to create a new
container from the tp33/django:1.0 image and create a hosts entry
named db that references the IP address used by the mysql container.

When you exit a container it's disk image persists allowing you to
restart it later. You can re-start your web container with 'docker
start web' and then re-attach an interactive session to it with
'docker attach web'.

Code layout
-----------

Your Django app will go in /app in this
container.

Create a new Django project with 'django-admin startproject foo' where
foo is the name you want to give your project.

Edit the foo/foo/settings.py file to point to your database. The image
already has the MySQL Pythong Connector installed so use the engine
'mysql.connector.django'. The HOST setting is 'db' as that's what we
told Docker to expose the mysql container's IP address under in this
container (via the --link command when we started this container). Set
USER, PASSWORD, and NAME to 'www', a password of your choice, and a
database name of your choice. You'll set these up in MySQL later.

Commit your newly created Django app up to GitHub. You should create
an ssh key pair ('ssh key-gen') and then upload your public key to
GitHub so you can push/pull changes without typing your password each
time.

Setting up MySQL
----------------

Now it's time to create the user and database in MySQL for your
app. You'll use the MySQL command line client to do
this. Conveniently, that's installed in the MySQL image you've already
got. You'll just need to start a NEW container running that image and
use it to connect to the first MySQL container image you're already
running.

Run 'docker run -it --name mysql-cmdline --link mysql:db mysql:5.7.8 sh' to:
- start a new container running the mysql:5.7.8 image (docker run)
- create a /etc/hosts entry called db that has the IP address of your other MySQL instance (the --link mysql:db)
- run an interactive shell (the -it and the sh command at the end)
- name it for easy future reference (--name mysql-cmdline)

Now inside this new container, run the mysql command line client and
tell the client to connect to your mysql server container: 'mysql
-uroot -pXXX -h db' where XXX is the MySQL server password you set
when you first ran the container with MYSQL_ROOT_PASSWORD. If all goes
well you should get a MySQL prompt.

You're logged in as root now and should create a non-root user that
your app will use. It's always best to use the least priviledges
possible so we'll only give this new user access to the tables
required for this app and no other privlidges.

	 create user 'www'@'localhost' identified by 'baz';
	 create database foo character set utf8;
	 grant all on foo.* to 'www'@'localhost';

This will create a new MySQL user named www and a new databse named
foo (replace foo with the name of your project). www's password is baz
(replace with something else for your project). Finally, this gives
www access to all tables in foo.

You can keep this container around for easy access to the MySQL
command line.

Make sure your Django settings.py DATABASES configuration matches this
user, password and database name.

Trying it all out
-----------------

If everything is working correctly, you can now go back to your web
container and try out some Django commands:

	  cd /app/foo
	  python manage.py migrate

If the DATABASES section is not set up correctly, or if the database
itself has a problem, you'll get an error here.

Now try running Apache. Your web container has Apacke and mod_wsgi
installed already. You should be able to run

	  cd /app/foo/foo
	  mod_wsgi-express start-server wsgi.py &
	  curl http://localhost:8000/

And get a 404 response from Django saying page-not-found. If you get
any other sort of response then something's not right.

mod_wsgi-express is a handy tool which generates an Apache conf file
for you and then starts Apache using that conf file. It prints out
some paths to where the conf file is and where the error log is. You
can read more at
http://blog.dscpl.com.au/2015/04/introducing-modwsgi-express.html

The error log is going to critial in helping you debug problems. When
something doesn't work, go look at the end of the error log. Changes
are there will be an exception and stack trace from Python there
telling you what went wrong.

Hello World
===========

For the final part of this project you're going to write a very simple
Django app. This assumes you're already familiar with the basics of
Django development. If not, now is a good time to review the Django
tutorial and docs. We're using version 1.7.X.

Create a very simple home page for your app. You should add an entry
to your urls.py that points to a view that you create in a new file
named home.py. You can return whatever you want, but whatever it is
should be rendered by a Django template.

Later on in the course we're going to be using the Bootstrap CSS
library. If you want to get fancy, you can include this CSS to pretty
up your page a bit.