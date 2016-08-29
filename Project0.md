Overview
--------

This project is set up to introduce to you a technology called Docker.
Docker will be used throughout the semester and we hope this project will familiarize you with the technology.
If you intend to take this course, we encourage you to finish (at least look at it) before the semester starts.
This will be helpful for you in terms of the first project where we do some more complicated stuff with Docker.

Docker First Encounter
----------------------
Docker is a system for managing Linux containers. You can read more about them elsewhere(like here http://www.docker.com/what-docker), but containers are like a virtual machine but lighter weight(all containers share the same kernel). You can define what kind of environment you need, install apps, and then easily clone/start as many instances of that container as you like -- on your dev machine or on public hosting providers like Amazon AWS.

You will start to appreciate the beauty of Docker/containers and the idea of Infrastructure as Software more and more as we progress in the semester. For now, you can think of Docker as a solution to managing environment that you software runs in. Docker encapsulates the installed apps (like the specific version of Django), files (the code you wrote for you web app), and database setup, etc., so that you can you can esily replicate your software along with its environment on any machine. You can think of Docker as a physical machines, but configured in the form of software. With containers, forget about the pain of environment setup. If the code works on my machine, it works on yours too.

In this preliminary project, you will install Docker and make sure it works. That's it. However it is important that you actually do the project.

Install Docker
--------------
Install Docker following the instructions Ubuntu: https://docs.docker.com/installation/ubuntulinux/ Mac: https://docs.docker.com/docker-for-mac/ Windows: https://docs.docker.com/docker-for-windows/ More OS options available: https://docs.docker.com/engine/installation/

Docker commands
---------------
Now that you get your docker working, it's time for us to go over some docker commands.

###docker ps -a
This command is used to see the status of containers you created. If you follow the project description, the command will give you something like
```
	zeizyy@zeizyy-thinkpad:~/cs4501_ta$ docker ps -a
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
	ad3b71d8998e        tp33/django:1.2     "/bin/bash"              6 hours ago         Exited (0) About a minute ago                       web
```

###docker run IMAGE
This command is used to create and run a container. (Like we just did in the instruction.)

Some useful options (for now)
- --name: specify the name of the container. The container will be named by a hash if no name is specified.
- -it: connected to the container in an interactive command line.
- any commands specified after IMAGE will be interpreted as command to be executed inside the container when it is created.

###docker start CONTAINER
This is used to start a stopped container.

###docker stop CONTAINER
This is used to stop a running container.

Final Comment
-------------
Docker is an amazing software we will use throughout this course. This is a heavily project-based course and you cannot learn a lot unless you do the project yourself. Hope you guys find this course useful and enjoy the course!
