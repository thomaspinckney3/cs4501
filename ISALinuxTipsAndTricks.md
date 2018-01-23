
# Good Ideas and Tips:

 - If you’re running Docker directly on your Linux laptop, set your user to be part of the docker group. This will save you from having to do “sudo docker whatever” and let you just do “docker whatever”
	 - eg, usermod -a -G docker tony
 - Use the python library Requests for your web calls. It’s much nicer to work with than the built in urllib library 
     - eg, pip install requests
- Start mod_wsgi-express with the flag --reload-on-changes to avoid having to force mod wsgi to reload your files when you make changes

## Django Commands:

 - To migrate your database: python manage.py migrate
 - To make migrations: python manage.py makemigrations
 - To run the development server: python manage.py runserver
	 - You may pass in 0.0.0.0:ABCD to expose the server to outside
   connections. It’ll serve on local at the port you specify with ABCD.
   Note, you can docker inspect to get a containers IP, let’s call it
   123.456.789.000 and then runserver and access the site with 123.456.789.000:ABCD in your browser. This is handy for when mod wsgi is being problematic.
 - To start python with access to django stuff: python manage.py shell

## Docker Commands:

 - To get a shell inside a docker container (must be running): docker
   exec -it <container name> /bin/bash
 - To view handy info about a container: docker inspect <container name>
 - To view a log of what happened in a container (especially why it
   died): docker logs <container name>
 - To view all docker containers: docker ps -a
 - To remove a docker container (must be stopped): docker rm <container
   name>
## Linux Commands:

 - When you’re not sure how a command works, ‘man cmd’ to look up the
   manual page for cmd. Man pages may not be installed inside your
   containers but the should be on your linux host system.
 - To list files in a directory: ls
 - To change directories: cd
 - To show the current directory you’re in: pwd
 - To see what processes are running: top is a great option, but not
   always available inside containers. ps is another option, albeit less
   interactive.
 - To search for a string: grep
 - To copy a file: cp
 - To move or rename a file: mv
 - To remove a file: rm <filename>, use rm -rf <folder name> to remove a
   folder. **BE CAREFUL WITH RM -RF**
 - To show what user you’re logged in as: whoami
 - To run a command as root (the system administrator account) prefix
   the command with sudo: sudo apt-get install docker-engine
 - To paginate text being displayed: less (or more though less is more)
 - To display a file: cat <filename>

### Putting it all together:
 - Search through all the python files in a directory looking for the
   word “urlopen”: grep urlopen *.py |less
 - Copy all the files in one directory into another: cp src/* dest
