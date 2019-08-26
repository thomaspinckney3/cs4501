Overview
=========

In this project you will set up a development environment (using
Docker, Python, Django and MySQL) and then create a simple Hello World
app in Django. Whew!

There's a lot of Linux and system level stuff in this project. It may
seem like you don't need to know all of this, but you do :). Don't just
type the commands that appear here. Understand what's going on and only
use these instructions as a hint.

This is probably the most involved project in the course. If you get
stuck, don't get demoralized. Ask the instructor or the TA's for help!

Linux
-----

You will need some basic knowledge of Linux and sys admin
knowledge. You'll be doing basic things like installing new software,
running SSH, looking at Apache log files, etc. While you don't need to
be a shell scripting god, you should also be able to get the gist of a
script if need be.


Containers
----------

Docker is a system for managing Linux containers. You can read more
about them elsewhere, but they're like virtual machines but lighter
weight. They're one component of treating infrastructure as
software. You can define what kind of environment you need, install
apps, define networking topologies, and then easily clone/start as
many instances of that container as you like—on your dev machine or
on public hosting providers like Amazon AWS or Digital Ocean.

You'll be using two containers initially: one that has Apache and Django
pre-installed and built to work together easily, and another that contains MySQL.

The system for containers we're using requires Linux. If you're
already running Linux on your laptop then you will likely need to just
install Docker. If you're using a Mac or Windows, you will need to
install Docker for Mac or Docker for Windows. 

Note, if you're using Windows, you must be using Windows Pro or Windows Education (both of which support Hyper-V based virtual machines) in order to use Docker for Windows. Docker Home will not work. UVa has free access to Windows Education so that's the recommended upgrade if you're using Windows Home. 

And lastly, if for some reason you cannot get
any of those to work, you could install VirtualBox and run Docker from within a Linux virtual machine

Note, when you're using the Docker for Mac or Docker for Windows
packages you're still using Linux. These packages install a
lightweight virtual machine and Linux installation. They manage all of
this for you so you don't even notice that you're running Linux on
your Mac or Windows machine.

Django
------

You should already be familiar with Python and Django. If not, go read more
here: https://docs.djangoproject.com/en/2.2. We highly recommend that you
work through the example application in the Django tutorial if this is your
first experience with Django.

MySQL
-----

The database used in this class will be MySQL. It's a very common SQL
database. You can read more at
http://dev.mysql.com/doc/refman/8.0/en/index.html.

You will not need detailed knowledge of SQL for most of the class, but you
should be familiar with the concepts involved of tables, queries,
updates etc.


Install everything
===================

- Install Docker following the instructions:
    - Ubuntu: https://docs.docker.com/installation/ubuntulinux/
    - Mac: https://docs.docker.com/docker-for-mac/
    - Windows: https://docs.docker.com/docker-for-windows/
    - More OS options available: https://docs.docker.com/engine/installation/

- Pull down the [Docker container image](https://hub.docker.com/r/tp33/django/~/dockerfile/) containing the Apache/Django installations:

        tp@devel:~$ docker pull tp33/django
        latest: Pulling from tp33/django
        [...]
        f9d42c108fd8: Pull complete
        Digest: sha256:77257e2e5894f2258b6cbb90d91d7a530ed1e6c5c194f119f7d6c979610bff72
        Status: Downloaded newer image for tp33/django:latest

- And now pull a container image with MySQL installed:

        tp@devel:~$ docker pull mysql:latest
        5.7.23: Pulling from library/mysql
        [...]
        Digest: sha256:e25e2768e910223db3095c1560aa2255371986b24fbebf4b015bae3cc60b9b34
        Status: Downloaded newer image for mysql:8.0.4
        tp@devel:~$

- Start up the container with Apache/Django using the image you just pulled:

        tp@devel:~$ mkdir -p ~/cs4501/app
        tp@devel:~$ docker run -it --name web -v ~/cs4501/app:/app tp33/django
    	root@4b6cb96f80f3:/app# python
        Python 3.5.4 (default, Aug 17 2018, 09:30:08)
        [GCC 4.9.2] on linux
        Type "help", "copyright", "credits" or "license" for more information.
        >>> import django
        >>> django.VERSION
        (2, 2, 4, 'final', 0)
        >>>
        root@4b6cb96f80f3:/app# exit
        tp@devel:~$

- Initialize the MySQL db container (NOTE: You MUST use the password specified here!):
        
        tp@devel:~$ mkdir ~/cs4501/db
        tp@devel:~$ docker run --name mysql -d -e MYSQL_ROOT_PASSWORD='$3cureUS' -v ~/cs4501/db:/var/lib/mysql  mysql:latest
        249e7f18b7679879197b49199de97a2a9f6705d99a7510086f51e30d830ca108
    - If you are running this on a Linux Subsystem in Windows 10 add --innodb_use_native_aio=0 to the end of the docker run command

- Note the status of your Docker containers. The one named *web* will be shown as
  exited while the one named *mysql* will be running.

        tp@devel:~$ docker ps -a
        CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS                 NAMES
        249e7f18b767        mysql:8.0.4        "docker-entrypoint.s…"   8 seconds ago        Up 8 seconds                    3306/tcp, 33060/tcp   mysql
        4b6cb96f80f3        tp33/django		    "/bin/bash"              About a minute ago   Exited (0) About a minute ago                         web
        tp@devel:~$

Congratulations—you now have two containers running, one for
running Django web apps and the other for running MySQL.

Setting up MySQL
----------------

Now it's time to create the user and database in MySQL for your
app. You'll use the MySQL command line client to do
this. Conveniently, that's installed in the MySQL image you've already
got. You'll just need to start a NEW container running that image and
use the MySQL client in that container to connect to the first MySQL container image you're already
running.

    tp@devel:~$ docker run -it --name mysql-cmdline --link mysql:db mysql:latest bash
    root@31617162d0de:/# mysql -uroot -p'$3cureUS' -h db
    mysql: [Warning] Using a password on the command line interface can be insecure.
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 2
    Server version: 8.0.4 MySQL Community Server (GPL)

    Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql>

You're logged in as root now and should create a non-root user that
your app will use. It's always best to use the least privileges
possible, so we'll only give this new user access to the tables
required for this app and no other privileges.

    mysql> create user 'www'@'%' identified with mysql_native_password by '$3cureUS';
    Query OK, 0 rows affected (0.01 sec)

    mysql> create database cs4501 character set utf8;
    Query OK, 1 row affected (0.00 sec)

    mysql> grant all on cs4501.* to 'www'@'%';
    Query OK, 0 rows affected (0.00 sec)

    mysql> ^DBye

(hit Ctrl-D at the mysql> prompt to exit the MySQL shell).

This will create a new MySQL user named `www` and a new database named
`cs4501`. The user `www`'s password is
`'$3cureUS'`. Finally, the user `www` is given access to all tables in the database `cs4501`.

You can keep this container around for easy access to the MySQL
command line.

Working with containers
-----------------------

Run `docker ps -a` and you'll see that you have two containers (the
ones you started above when testing your install) named *web* and
*mysql*. These are your dev environment containers. You'll do your
coding and initial testing in these containers. Run `docker rm web` to
clean up the (stopped) *web* container you started earlier when testing
your install. Go ahead and stop your *mysql* container with `docker stop
mysql`.

Docker provides a simple way for containers to talk to each other via
private networks. When you start a container you can tell Docker to
create an `/etc/hosts` entry for another running container. We'll do
this using the `--link` flag so that the *web* container can easily connect to the *mysql*
container. Let's restart the *web* container this way:

    tp@devel:~$ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                 NAMES
    249e7f18b767        mysql:8.0.4        "docker-entrypoint.s…"   23 seconds ago      Up 22 seconds               3306/tcp, 33060/tcp   mysql
    f1e282544b7b        tp33/django         "/bin/bash"              44 seconds ago      Exited (0) 41 seconds ago                         web
    tp@devel:~$ docker rm web
    web
    tp@devel:~$ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                 NAMES
    249e7f18b767        mysql:8.0.4        "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp, 33060/tcp   mysql
    tp@devel:~$ docker run -it --name web -p 8000:8000 --link mysql:db -v ~/cs4501/app:/app tp33/django
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

This tells Docker to create a new container from the tp33/django
image and create a hosts entry named db that references the IP address
used by the *mysql* container. It also exposes port 8000 on the container
to port 8000 on the host machine (so you can see the app on your browser
just like you could any other web page). The concepts of port exposure and container
networking has historically been often misunderstood by students. It's highly recommended
to come to office hours if you do not fully understand this topic—it will save you a lot of time later on.

When you exit a container, its disk image persists, allowing you to
restart it later. You can re-start your *web* container with `docker
start web` and then re-attach an interactive session to it with
`docker attach web`.

If you want to remove a container, use `docker rm`. If you want to
remove an image, use `docker rmi`. Once you remove a container, any
changes that you made inside of it are gone.

The way that you can persist data between container invocations in a way
that can be viewed on your host machine as well is to
store the data outside the container and then make the data accessible
once the container starts. The simplest way to do this is to mount a
directory from outside the containers to a point inside the container.

The `-v` argument to `docker run` does exactly this. It takes two paths,
seperated by a colon. The first path is a directory on your
Mac/Windows/Linux system and the second path is a directory inside the
container. Any data stored or changed in this directory in the
container will really be getting stored outside the container and so
will persist beyond the container's lifetime.

Notice that when we started the *web* and *mysql* containers above we used
the `-v` argument to tell docker that we wanted to make data in our host
system be mounted into the containers. We said that `/app` in the web
container should access data stored in the host's `~/cs4501/app` and the
*mysql* container should store its `/var/lib/mysql` data under the host's
`~/cs4501/db`.

Go ahead and try creating a file on your host in `~/cs4501/app` and then
verify that you can access that file inside your *web* container.

Knowing `curl` as a command line HTTP tool is important, but developing a
Django project by looking at raw HTML from the terminal is not the
best development experience. To see those HTML scripts in a browser on
your host machine, you need to expose a port inside the container to a port
on your host machine. The `-p` argument to the `docker run` command does that.
It exposes port 8000 (the 8000 after colon) inside the container to port 8000
on your host machine (the 8000 before the colon). Then the Django app can be
reached on the host machine at http://localhost:8000. Note for students who are
using Docker Toolbox. The Toolbox does not automatically forward the Docker
host to localhost. You can find the Docker host IP at the beginning of the Docker
Quickstart Terminal (under the whale logo).

Development Environment
====================

Git
---

You'll use GitHub for version control for this class. If you don't have a GitHub
account already, go create one. There's good online help for using
GitHub and Git.

Creating a repo for Project 1 is not required since it is an individual project and
it won't be checked/needed for project submission. However, if you are not experienced
with Git/Github, it is highly recommended that you start to familiarize yourself with
it now. You should be familiar with using a basic [Git Workflow](https://guides.github.com/introduction/flow/),
and consider using it when you start Project 2 with your group. You should not be pushing pycache files,
database files, or database migration files on Github.
Learn to use `.gitignore` to enforce this, otherwise nasty merge conflicts and other issues will pop up later on.

Dev process
-------------

Generally, you will be editing your code in the host directory
`~/cs4501/app`, testing it inside your container and then when it's
complete you will check it into github from your host. This allows you
to edit using whatever IDE / editor you prefer on your Mac/Windows
laptop but run your code with a carefully configured system specified
inside your *web* container.

Making your own containers
---------------------------

You can create your own Docker images. The easiest way to do this is
to start with an existing container and make changes to it. For
example, maybe there are some Python libraries that you want to use
and which I didn't install in tp33/django. You could build your own
container that inherits from my container and then adds your Python
packages.

Each container's image is built according to instructions stored in a
Dockerfile. This is the list of commands required to install software
and configure your image. You can read more about the syntax of
Dockeriles on the Docker website.

The Dockerfile I used to create tp33/django is at
https:/github.com/thomaspinckney3/django-docker.

If you are interested in learning more about this, come to office hours.

Code layout
-----------

Your Django app will go in `/app` in the *web* container, which is mounted
from `~/cs4501/app` in your host environment.

Inside your web container, create a new Django project with
`django-admin startproject foo` where *foo* is the name you want to give
your project.

On your host environment, edit the `~/cs4501/app/foo/foo/settings.py` file to point
to your database.

The tp33/django image already has mysqlclient installed
so use the engine `'django.db.backends.mysql'`. The HOST setting is `'db'`
as that's what we tell Docker to expose the *mysql* container's IP
address under in this container (via the `--link` command when we
started this container). Set USER, PASSWORD, and NAME to `'www'`,
`'$3cureUS'`, and `'cs4501'`. You must use these values in order to hook
into our grading systems later.

Again, note, you're creating / editing your code outside the
container in the host but running inside the container under `/app`.

Inside your *web* container:

    root@4ee80653d2be:/app# django-admin startproject foo

(but replace *foo* with the name of your project)

Back on your Mac/Windows shell:

    tp@devel:~$ cd ~/cs4501/app/foo/foo
    tp@devel:~/cs4501/app/foo/foo# vi settings.py
    tp@devel:~/cs4501/app/foo/foo# grep -A 10 -B 5 DATABASES settings.py
    # Database
    # https://docs.djangoproject.com/en/2.1/ref/settings/#databases

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'cs4501',
            'USER': 'www',
            'PASSWORD': '$3cureUS',
            'HOST': 'db',
        }
    }

    # Internationalization

Finally, commit and push your newly created Django app up to GitHub.

You should create an ssh key pair (`ssh key-gen`) and then upload your
public key to GitHub so you can push/pull changes without typing your
password each time. (Not required)

In the future, when you want to deploy a test or production container
you can just use `docker run` to start an instance of tp33/django,
checkout your code from GitHub, and you're good to go.

Trying it all out
-----------------

If everything is working correctly, you can now go back to your web
container and try out some Django commands.

Attach to your *web* container

    tp@devel:~$ docker exec -it web bash

Then apply database migrations:

    root@4ee80653d2be:~/app# cd foo
    root@4ee80653d2be:~/app/foo# python manage.py migrate
    Operations to perform:
    Apply all migrations: contenttypes, sessions, auth, admin
    Running migrations:
    Applying contenttypes.0001_initial... OK
    Applying auth.0001_initial... OK
    Applying admin.0001_initial... OK
    Applying sessions.0001_initial... OK
    root@4ee80653d2be:~/app/foo#

If the DATABASES section is not set up correctly, or if the database
itself has a problem, you'll get an error here.

Now try running Apache. Your *web* container has Apache and mod_wsgi
installed already. Again, note 'foo' is my project name and you
will need to replace that with your project's name.

Inside your *web* container:

    root@399bd9c4a2a5:/app# mod_wsgi-express start-server --reload-on-changes --working-directory /app/foo /app/foo/foo/wsgi.py &
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

    root@399bd9c4a2a5:/app/foo# curl http://localhost:8000/

You should get back the stock Django "It worked" page. If you get any
other sort of response then something's not right.

`mod_wsgi-express` is a handy tool which generates an Apache httpd.conf file
for you and then starts Apache using that httpd.conf file. It prints out
some paths to where the conf file is and where the error log is. You
can read more at
http://blog.dscpl.com.au/2015/04/introducing-modwsgi-express.html.

`curl` is another handy tool. It is a command line tool for issuing
HTTP GET or POST commands. It's great for debugging web apps.

If you have trouble and need to debug your code there are two ways:
either use the Django development server or look in the Apache error
logs.  The Django development server prints exceptions to standard out
so it's easy to see when something is wrong. If you're using Apache,
the `mod_wsgi-express` command above printed out the path to the error
log—it will be something like
`/tmp/mod_wsgi-localhost:8000:0/error_log`. Look there for exceptions or
other errors.

Hello World
===========

Next you're going to write a very simple
Django app. This assumes you're already familiar with the basics of
Django development. If not, now is a good time to review the Django
tutorial and docs.

Create a very simple home page for your app. You should add an entry
to your `urls.py` that points to a view that you create in a new file
named `home.py`. You can return whatever you want, but whatever it is
should be rendered in an html file using the `render` keyword.

Later on in the course we're going to be using the Bootstrap CSS
library. If you want to get fancy, you can include this CSS to pretty
up your page a bit.

What to turn in
---------------

Turn in screenshots of `docker ps` showing your running containers
along with a screenshot of your "Hello World" output from curl.
