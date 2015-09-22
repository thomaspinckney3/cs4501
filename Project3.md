Overview
========

In this part of the project you will develop your experience service
API layer and an HTML front-end. This project will build on top of the
work you've done in Project 1 and Project 2.

Architecture
------------

As a reminder, we are building a four-tier web app:

   - HTML front end
   - Experience service APIs
   - Low level / Model APIs
   - Database

Each layer will run in it's own container and communicate via the
network with the other layers.

An example request will be processed like:

   - A request from a user's browser will go to the HTML front end.
   - The front-end will call thge experience service to get the data needed and render the data as HTML.
   - The experience service will call one or more of the low-level APIs to get it's data and return it.
   - The low-level APIs will call the database to read data and update models.

The experience service app will invoke the low-level APIs via HTTP and
receive JSON responses. Similarly, it will provide HTTP/JSON APIs up
to the HTML front-end app. Note, an end-user only ever access the HTML
front-end app. All the other parts are hidden and not publicly
accessible for security reasons.

The key point of this is the strict isolation between levels. The only
way the HTML front-end communicates with the rest of your app is
through the expereince service level. In turn, the only way your
experience service level interacts with the database is through the
low-level API. This is directly analagous to the data abstraction and
modularity that you learn about applying in an individual program, but
here it is applied to a system of programs.

This will seem overly confusing and burdensome for a simple
app. However, for a large app with many teams of people working on it
it provides isolation between teams and apps.

Pages
-----

The only required pages for your site are a home page and an item
detail page. The detail page should show details about whatever it is
your marketplace is about -- rides, books, tickets. The home page
should show links to the detail pages -- maybe the newest content, the
most popular content, etc.

If you have time, you should add other useful pages like a user
profile page to show info about buyers / sellers. Later on, we'll add
a search page powered by a real search engine. For now you can add a
simple / dummy search page if you like that just shows all content in
your system.

Don't worry too much about the design of your pages for now. Bootstrap
will provide decent default styling. However, if you want, you can
think about how to improve the design or make it responsive for
smaller screens.

Implementation
--------------

### Container Linking ###

You will have four Docker containers running -- one for each layer in
your app: one instance of the MySQL container and three instances of
your Django container.

Docker assigns a unique IP address to every container running. We'll
use container linking to make sure each container knows the IP address
of the other containers to talk to. This is accomplished via adding
the --link command line to the docker run command:

    docker run -it --name model-api --link mysql:db tp33k/django:1.0

That command will create a hostname called 'db' and make sure it's
always pointing to the IP address for the container named
'mysql'. Thus, your app in this container can always connect to the
host 'db' instead of having to know which IP address your MySQL
container is actually running as. This is how you set up your
project's settings.py so far.

Similarly, you'll run another container for your experience service
and link it to your low-level API:

    docker run --it --name exp-api --link model-api:model-api tp33k/django:1.0

Then the app running in this container can make HTTP requests to the
host model-api in order to conncect to your model-api container.

And finally, your third container for running the HTML front-end will
link to the experience service container:

    docker run --it --name html-fe --link exp-api:exp-api tp33k/django:1.0

### HTML ###

The HTML front-end app will render data via Django templates to
produce HTML. The external CSS library Boostrap will be used for
styling the HTML.

### Code layout ###

You should be thinking of your app as three separate sub-apps: the
low-level / model API, the experience service API and the HTML
front-end. The best way to do this is to create _three separate Django
apps_.

Using multiple apps will allow you to have three separate settings.py
files. This will be imporant because you'll want to do things
differently in your HTML front-end since it's the part the public will
be able to access and it's the part that's serving HTML as opposed to
JSON. For example, you'll want to enable things like CSRF-protection
middleware in the HTML front-end while that is not appropriate for the
other tiers of your web app.

### Development process ###

If you're not careful and mehthodical, you'll quickly end up with a
mess. I recommend you do the following:

- design your web interface first -- sketch on a piece of paper what
  info you want to show, where it will be relative to the other
  content etc.

- start with building the web interface tier first to define what your
  experience services need to return. Then build the experience
  service to provide the data your web interface needs.

- develop and test each tier independently. That means that if you
  make changes to your low-level API, test it carefully before moving
  on to changes in the experience services tier. You don't want to
  change every tier and then try things and not know why things are
  failing.

- make sure to carefully start your containers each time with the
  right --link and --name arguments.

- start early because if you wait until a few days before the
  assignment is due you will fail to finish it.

- ask questions to get help from your teammates and from me.

- check your code in as small chunks instead of giant commits. Make
  one logical change, test it, get a teammate to review it, and then
  commit it. If you commit lots of small changes and you later find
  something doesn't work, it's easier to go through the small changes
  to see what broke things as opposed to having to go through a huge
  set of changes all in one commit.

### Calling HTTP/JSON APIs in Python ###

    import urllib.request
    import urllib.parse
    import json

    # make a GET request and parse the returned JSON                                                                                                                                                           # note, no timeouts, error handling or all the other things needed to do this for real                                                                                                                      
    print ("About to do the GET...")
    req = urllib.request.Request('http://jsonplaceholder.typicode.com/posts/1')
    resp_json = urllib.request.urlopen(req).read().decode('utf-8')
    resp = json.loads(resp_json)
    print(resp)

    # make a POST request.                                                                                                                                                                                     # we urlencode the dictionary of values we're passing up and then make the POST request                                                                                                                    # again, no error handling                                                                                                                                                                                  
    print ("About to do the POST...")
    post_data = {'title': 'Demo Post', 'body': 'This is a test', 'userId': 1}
    post_encoded = urllib.parse.urlencode(post_data).encode('utf-8')
    req = urllib.request.Request('http://jsonplaceholder.typicode.com/posts', data=post_encoded, method='POST')
    resp_json = urllib.request.urlopen(req).read().decode('utf-8')
    resp = json.loads(resp_json)
    print(resp)
