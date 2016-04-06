Overview
--------

This project is set up to introduce to you a technology called Docker.
Docker will be used throughout the semester and we hope this project will familiarize you with the technology.
If you intend to take this course, we encourage you to finish (at least look at it) before the semester starts.
This will be helpful for you in terms of the first project where we do some more complicated stuff with Docker.

Docker First Encounter
----------------------
Docker is a system for managing Linux containers. You can read more about them elsewhere(like here http://www.docker.com/what-docker), but containers are like a virtual machine but lighter weight(all containers share the same kernel). You can define what kind of environment you need, install apps, and then easily clone/start as many instances of that container as you like -- on your dev machine or on public hosting providers like Amazon AWS.

You will start to appreciate the beauty of Docker/containers and the idea of Infrastructure as Software more and more as we progress in the semester. For now, you can think of Docker as a solution to managing environment that you software runs in. Docker encapsulates the installed apps(like the specific version of Django), files(the code you wrote for you web app), and database setup etc so that you can you can esily replicate your software along with its environment on any machine. You can think of Docker as a physical machines but configured in the form of software. With containers, forget about the pain of environment setup. If the code works on my machine, it works on yours too.

In this preliminary project, you will install Docker and make sure it works. That's it. However it is important that you actually do the project.

Install Docker
--------------
### Linux User
Docker is only supported on Linux, so good news if you are using some version of Linux like Ubuntu or Debian. Using a native Linux machine will save you so much trouble in terms of networking, memory issue and GUI interface etc throughout the semester. If you don't have a Linux machine, I (Ian Zheng) strongly suggest you installing Linux on your machine to dual boot with you Mac or Window OS.
- The installation for Linux is easy. Just follow the instructions@
  https://docs.docker.com/installation/ubuntulinux/

### Mac users
Unfortunately, Docker is not yet supported on Mac OS. If you are a Mac user, you can either get access to a Linux machine or use our old friend from CS2150, Virtual box. The steps are as follows,
- Install VirtualBox from https://www.virtualbox.org

- Download the Ubuntu Server 15.04 (or newer) ISO from
  http://www.ubuntu.com/download/server (The Server version is not like the virtual machine you use during 2150 since it does not have a GUI. We are thinking of wrrting up a instruction on Linux shell so that you can be more familiar with that. Stay turned.)

- Start VirtualBox, create a new VM (1gig RAM, 12gigs disk and 
  'bridged' networking), selecting the Ubuntu ISO under Storage for
  the optical drive.

- If you will be working on campus, you will need to register the network MAC address for your VM with UVa. If you don't, you won't be able to connect to the wireless network. You can find the MAC address under the Settings->Network tab in VirtualBox. Go to https://netreg.itc.virginia.edu to register it. 

- Start the VirtualBox VM which should boot form the Ubuntu ISO. Go
  through the install process. Make sure to install the SSH server so
  you can log into your VM. 

- Install Docker following the instructions
  https://docs.docker.com/installation/ubuntulinux/

- Note, that if you move between runing your VM at home outside the UVa network and the UVa network, you will likely have to reboot your VM each time you switch between networks. This is a simple way to pick up the new IP address and name servers that each network requires.

### Windows users
We strongly recommend using a Linux machine or a Mac running VirtualBox rather than Windows. Unfortunately, none of the course staff have extensive knowledge of Windows systems and we are unable to provide substantial help on getting VirtualBox and Windows working well together.

Play with Docker
----------------
After Docker is installed, we can test it.
- Pull down the Docker containers for Django:
```
	tp@devel:~$ docker pull tp33/django:1.2
	1.2: Pulling from tp33/django
	[...]
	f9d42c108fd8: Pull complete 
	Digest: sha256:90ff75c9817102fe0f5f5e9ff823bd0ea5ad05df24a87bd6def6c18f194da8a
	Status: Downloaded newer image for tp33/django:1.2
```
Docker works in a simiar way to Github. When you type
```
    docker pull tp33/django:1.2
```
it pulls the specific container 'django' from user 'tp33' with tag '1.2' to your local machine. The container has Django version 1.7.10 installed.

You can then initialize a container.
```
	tp@devel:~$ docker run -it --name web tp33/django:1.2
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
```

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
This command is used to create and run a container.(like we just did in the instruction.)

Some useful options (for now)
- --name: specify the name of the container. The container will be named by a hash if no name is specified.
- -it: connected to the container in an interactive command line.
- any commands specified after IMAGE will be interpreted as command to be executed inside the container when it is created.

###docker start CONTAINER
This is used to start a STOPPED container.

###docker stop CONTAINER
This is used to stop a running container.

Final Comment
-------------
Docker is an amazing software we will use throughout this course. This is a heavily project-based course and you cannot learn a lot unless you do the project yourself. Hope you guys find this course useful and enjoy the course!
