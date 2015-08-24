Overview
=========

In this project you will set up a development environment (using
VirtualBox, Ubuntu Linux, Docker, Python, Django and MySQL) and then
create a simple Hello World app in Django. Whew!

There's a lot of Linux and system level stuff in this project. It may seem like you don't need to know all of this, but you do :) Don't just type the commands that apear here. Understand what's going on and only use these instructions as a hint.

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

- Pull down the Docker containers for Django:

		tp@devel:~$ docker pull tp33/django:1.0
		1.0: Pulling from tp33/django
		[...]
		f9d42c108fd8: Pull complete 
		Digest: sha256:90ff75c9817102fe0f5f5e59ff823bd0ea5ad05df24a87bd6def6c18f194da8a
		Status: Downloaded newer image for tp33/django:1.0

- And MySQL:

		tp@devel:~$ docker pull mysql:5.7.8
		5.7.8: Pulling from library/mysql
		[...]
		library/mysql:5.7.8: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
		Digest: sha256:22d2c6e4bff13ccb4b3c156bdaa97e4fbf6f15ee0467233714f51540c64ad6b6
		Status: Downloaded newer image for mysql:5.7.8
		tp@devel:~$ 

- Test it

        tp@devel:~$ docker run -it --name web tp33/django:1.0
    	root@4b6cb96f80f3:/app# python
    	Python 3.4.3 (default, Aug  9 2015, 11:14:27) 
    	[GCC 4.9.2] on linux
    	Type "help", "copyright", "credits" or "license" for more information.
    	>>> import django
    	>>> django.VERSION
    	(1, 7, 10, 'final', 0)
    	>>> 
    	root@4b6cb96f80f3:/app# exit
    	tp@devel:~$ 

- Initialize the MySQL db's (replace XXX with a password of your choice for the root MySQL user):

        tp@devel:~$ docker run --name mysql -d -e MYSQL\_ROOT\_PASSWORD='XXX' mysql:5.7.8
    	79c856338ace5edc9df074e252fb16caedd0ed1b53f64eef613e84301482dd75
    	tp@devel:~$ docker ps -a
    	CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS               NAMES
    	79c856338ace        mysql:5.7.8         "/entrypoint.sh mysql"   8 seconds ago        Up 8 seconds                    3306/tcp            mysql
    	4b6cb96f80f3        tp33/django:1.0     "/bin/bash"              About a minute ago   Exited (0) About a minute ago                       web
    	tp@devel:~$ 
  
- Note the status of your Docker containers. The one named web will be shown as
  exited while the one named mysql will be running.

Congratulations -- you should have a Linux VM with two docker
containers, one for running Django web apps and the other for running
MySQL.

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
container. Let's restart the web container this way:

    tp@devel:~$ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
    f34a25b95ffa        mysql:5.7.8         "/entrypoint.sh mysql"   23 seconds ago      Up 22 seconds               3306/tcp            mysql
    f1e282544b7b        tp33/django:1.0     "/bin/bash"              44 seconds ago      Exited (0) 41 seconds ago                       web
    tp@devel:~$ docker rm web
    web
    tp@devel:~$ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
    f34a25b95ffa        mysql:5.7.8         "/entrypoint.sh mysql"   About a minute ago   Up About a minute   3306/tcp            mysql
    tp@devel:~$ docker run -it --name web --link mysql:db tp33/django:1.0
    root@1c359b81b84f:/app# ping db
    PING db (172.17.0.5): 56 data bytes
    64 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.110 ms
    64 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.152 ms
    ^C--- db ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 0.110/0.131/0.152/0.000 ms
    root@1c359b81b84f:/app# exit
    exit
    tp@devel:~$ 

This tells Docker to create a new container from the tp33/django:1.0
image and create a hosts entry named db that references the IP address
used by the mysql container.

When you exit a container it's disk image persists allowing you to
restart it later. You can re-start your web container with 'docker
start web' and then re-attach an interactive session to it with
'docker attach web'.

If you want to remove a container, use 'docekr rm'. If you want to
remove an image, use 'docker rmi'.

Development Environment
====================

You'll use GitHub for version control. If you don't have a GitHub
account already, go create one. There's good online help for using
GitHub and Git.

Dev container
-------------

You've installed your development environment in the prior steps. Over
time you'll replicate this environment to create testing (QA)
environments and eventually a production (public) environment. Using
modular containers will make this very easy. For now, though, you'll
be doing all your work in the dev environment you just set up.

Your app is built using two containers: a web app container (running
Apache/Django) and a MySQL container. It's highly recommended that you
NOT make any changes to either container. Both are set up for
production use and should be kept as minimal as possible.

However, you will find that using the django/1.0 container tedious for
your development environment. For example, there is no SSH installed
and everytime you restart it you will have to re-install SSH, generate
new SSH keys, register them with GitHub etc.

Instead, we'll make a new container, based off the tp33/django:1.0
container with your customizations in them. Then you can use this
dervivative container for doing development work while still keeping
the django:1.0 container pristine for production work. The short
version of what you'll do is:

- connect to the web instance of django/1.0 you started earlier
- make some changes like installing ssh
- commit / tag the resulting container image so you can easily go back to it

Assuming your web container is still around you'll need to re-start and attach to it:

    tp@devel:~$ docker start web
    tp@devel:~$ docker attach web

Otherwise you'll need to create a new container via 'docker run...'.

Either way, once in use apt-get to install openssh and then generate a key pair:

    root@96e27e313b9d:/app# apt-get install openssh-client
    Reading package lists... Done
    [...]
    root@96e27e313b9d:/app# ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa): 
    Created directory '/root/.ssh'.
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in /root/.ssh/id_rsa.
    Your public key has been saved in /root/.ssh/id_rsa.pub.
    The key fingerprint is:
    f2:89:ff:5e:e4:c1:dc:86:17:b9:3d:0c:5b:6e:2d:96 root@96e27e313b9d
    The key's randomart image is:
    +---[RSA 2048]----+
    |                 |
    |               . |
    |             .o. |
    |           o o*=.|
    |      . S   *.E=o|
    |       + . o =...|
    |      . o   o    |
    |       .   .     |
    |        .oo      |
    +-----------------+
    root@96e27e313b9d:/app# cat ~/.ssh/id_rsa.pub 
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzmRiKXb8Aw+IUUX/EGf3+VXq/yOnJLhaXbOPgOWEABvMU9SYniA2FUDo/IWsRbZ3kp3FQgQWyTdEoGuznUwCtuMJsIxcCCLSw5Q5hbreH6fPfjdkrMRIbOX5JBG6bMhXgpq8GQmYy51M+Q5Do/vkGCcQuRynTMMBeGhLcuy4/XP5dt9gCYdY+LKONapSXKgYRs3nD4vvY7lPvTukNMnCzKKdTb0CAOmFb2x9M/ukgwJfr5ep3PSweiS4z55L/HtLDxY/v2jtsEs4YB19p9hY1iV2wMt+uh5kPI3rZhxjVZ6clEFyXyYPWcHSTWSLRM0naRwn8r4cNoSnANq7Al+KX root@96e27e313b9d
    root@96e27e313b9d:/app# exit

Then back on your Linux VM you can tag the state of this container's image with a name so you can re-start containers using the image:

    tp@devel:~$ docker commit -m "Added SSH and keypair" -a "Tom Pinckney" 96e27e313b9d tp33/django:devel
    19c85f570abd2545e0313ca797a75fc7f9617c352ab20f370a1ab676a6be6086
    tp@devel:~$ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    tp33/django         devel               19c85f570abd        18 seconds ago      472.4 MB
    tp33/django         1.0                 f9d42c108fd8        29 hours ago        454.6 MB
    mysql               5.7.8               3d251620fb60        11 days ago         321.5 MB

And then in the future when you wnat to create a new dev instance that's preconfigured with SSH all you need to do is:

    tp@devel:~$ docker run -it --name web-devel --link mysql:db tp33/django:devel

If you want to make more substantial changes I recommend creating your
own Dockerfile and building a new image. This will be more
reproducible and automated and save you time in the future.

The Dockerfile I used to create tp33/django is at https:/github.com/thomaspinckney3/django-docker

Code layout
-----------

Your Django app will go in /app in this
container.

Create a new Django project with 'django-admin startproject foo' where
foo is the name you want to give your project. Edit the
foo/foo/settings.py file to point to your database. The image already
has the MySQL Pythong Connector installed so use the engine
'mysql.connector.django'. The HOST setting is 'db' as that's what we
told Docker to expose the mysql container's IP address under in this
container (via the --link command when we started this container). Set
USER, PASSWORD, and NAME to 'www', a password of your choice, and a
database name of your choice. You'll set these up in MySQL later.

Note, you're going to be creating / editting your code under your home
directory but running under /app.

    root@4ee80653d2be:/App# cd
    root@4ee80653d2be:~# django-admin startproject stuff
    root@4ee80653d2be:~# cd stuff/
    root@4ee80653d2be:~/stuff# vi stuff/settings.py
    root@4ee80653d2be:~/stuff# grep -A 10 -B 5 DATABASES
    # Database
    # https://docs.djangoproject.com/en/1.7/ref/settings/#databases

    DATABASES = {
              'default': {
              'ENGINE': 'mysql.connector.django',
              'NAME': 'cs4501',
              'USER': 'www',
              'PASSWORD': 'S3cure',
              'HOST': 'db',
              }
          }

    # Internationalization

Commit your newly created Django app up to GitHub. You should create
an ssh key pair ('ssh key-gen') and then upload your public key to
GitHub so you can push/pull changes without typing your password each
time.

In the future, when you want to deploy a test or production container
you can just use 'docker run' to start an instance of tp33/django:1.0,
checkout your code from GitHub, and you're good to go.

Setting up MySQL
----------------

Now it's time to create the user and database in MySQL for your
app. You'll use the MySQL command line client to do
this. Conveniently, that's installed in the MySQL image you've already
got. You'll just need to start a NEW container running that image and
use it to connect to the first MySQL container image you're already
running.

    tp@devel:~$ docker run -it --name mysql-cmdline --link mysql:db mysql:5.7.8 bash
    # mysql -uroot -p'!Secure' -h db
    mysql: [Warning] Using a password on the command line interface can be insecure.
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 2
    Server version: 5.7.8-rc MySQL Community Server (GPL)

    Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> 


You're logged in as root now and should create a non-root user that
your app will use. It's always best to use the least priviledges
possible so we'll only give this new user access to the tables
required for this app and no other privlidges.

    mysql> create user 'www'@'%' identified by 'S3ecure';
    Query OK, 0 rows affected (0.01 sec)

    mysql> create database cs4501 character set utf8;
    Query OK, 1 row affected (0.00 sec)

    mysql> grant all on cs4501.* to 'www'@'%';
    Query OK, 0 rows affected (0.00 sec)

    mysql> 

This will create a new MySQL user named www and a new databse named
cs4501 (replace foo with the name of your project). www's password is
'S3cure' (replace with something else for your project). Finally, this
gives www access to all tables in cs4501.

You can keep this container around for easy access to the MySQL
command line.

Make sure your Django settings.py DATABASES configuration matches this
user, password and database name.

Trying it all out
-----------------

If everything is working correctly, you can now go back to your web
container and try out some Django commands:

    root@4ee80653d2be:~/stuff# python manage.py migrate
    Operations to perform:
    Apply all migrations: contenttypes, sessions, auth, admin
    Running migrations:
    Applying contenttypes.0001_initial... OK
    Applying auth.0001_initial... OK
    Applying admin.0001_initial... OK
    Applying sessions.0001_initial... OK
    root@4ee80653d2be:~/stuff#

If the DATABASES section is not set up correctly, or if the database
itself has a problem, you'll get an error here.

Now try running Apache. Your web container has Apacke and mod_wsgi
installed already. Note, we're running from under /app like the prod containers will and not under your home directory.

You should be able to run

    root@399bd9c4a2a5:/app# cd /app
    root@399bd9c4a2a5:/app# git clone https://github.com:thomaspinckney3/stuff.git
    Cloning into 'stuff'...
    remote: Counting objects: 8, done.
    remote: Compressing objects: 100% (7/7), done.
    remote: Total 8 (delta 0), reused 8 (delta 0), pack-reused 0
    Receiving objects: 100% (8/8), done.
    Checking connectivity... done.
    root@399bd9c4a2a5:/app# mod_wsgi-express start-server --working-directory /app/stuff --url-alias /static stuff/static stuff/stuff/wsgi.py &
    [1] 73
    Server URL         : http://localhost:8000/
    Server Root        : /tmp/mod_wsgi-localhost:8000:0
    Server Conf        : /tmp/mod_wsgi-localhost:8000:0/httpd.conf
    Error Log File     : /tmp/mod_wsgi-localhost:8000:0/error_log (warn)
    Request Capacity   : 5 (1 process * 5 threads)
    Request Timeout    : 60 (seconds)
    Queue Backlog      : 100 (connections)
    Queue Timeout      : 45 (seconds)
    Server Capacity    : 20 (event/worker), 20 (prefork)
    Server Backlog     : 500 (connections)
    Locale Setting     : en_US.UTF-8

And test it out:

    root@399bd9c4a2a5:/app/stuff# curl http://localhost:8000/

and get back the stock Django "It worked" page. If you get
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
