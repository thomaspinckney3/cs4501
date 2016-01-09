Overview
========

This project is set up to introduce to you a technology called Docker.
Docker will be used throughout the semester and that is the reason we set up this preliminary project.
If you intend to take this course, we encourage you to finish (at least look at it) before the school starts.
This will be helpful for you in terms of the first project where we do some more complicated stuff with Docker.

Docker First Encounter
----------------------
Docker is a system for managing Linux containers. You can read more about them elsewhere, but containers are like a virtual machine but lighter weight(they share the kernel and memory). You can define what kind of environment you need, install apps, and then easily clone/start as many instances of that container as you like -- on your dev machine or on public hosting providers like Amazon AWS.

You will start to appreciate the beauty of Docker/containers and the idea of Infrastructure as Software more and more as we progress in the semester. For now, you can think of Docker as a solution to manage environment that you software runs in. Docker encapsulates the installed apps(like the specific version of Django), files(the code you wrote for you web app), and Database setup etc so that you can you can esily replicate your software along with its environment on any machine. You can think of Docker as a physical machines but configured in the form of software. With containers, forget about the pain of environment setup. If the code works on my machine, it works on yours too.

In this preliminary project, you will install Docker and make sure it works.That's it. However it is important that you do it since it is helpful for the first project where we will do something more complicated with the containers.

Install Docker
--------------
### Linux User
Docker is supported natively on Linux, so good new if you are using some version of Linux like Ubuntu or Debian. Using a native Linux machine will save you so much trouble in terms of networking, memory issue and GUI interface etc. If you don't have a Linux machine, I (Ian Zheng) strongly suggest you installing one on your machine so that you can dual boot with you Mac or Window OS.
- The installation is easy. Just follow the instructions@
  https://docs.docker.com/installation/ubuntulinux/

### Mac users
Unfortunately, Docker is not yet supported on Mac OS. If you are a Mac user, you can either get access to a Linux machine or use our old friend from CS2150, Virtual box.
- Install VirtualBox from https://www.virtualbox.org

- Download the Ubuntu Server 15.04 (or newer) ISO from
  http://www.ubuntu.com/download/server (The Server version is not like the virtual machine you use during 2150 since it does not have a GUI. We are thinking of wrrting up a instruction on Linux shell so that you can be more familiar with that. Stay turned.)

- Start VirtualBox, create a new VM (1gig RAM, 12gigs disk and 
  'bridged' networking), selecting the Ubuntu ISO under Storage for
  the optical drive.

- Start the VirtualBox VM which should boot form the Ubuntu ISO. Go
  through the install process. Make sure to install the SSH server so
  you can log into your VM. 

- Install Docker following the instructions
  https://docs.docker.com/installation/ubuntulinux/
