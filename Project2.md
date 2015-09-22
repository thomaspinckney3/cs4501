Overview
========

In this phase of the project you will build your Django data models
and then build a first version of services for accessing those
models. There are two weeks assigned for completing this work, but if
you wait until the second week you will likely not finish on time.

Django
------

The Django documentation is going to be invaluable and essential for
completing this work. I'm not going to provide step-by-step
instructions for how to build a Django app. I'm assuming you've done
it before and are using the tutorials etc on the Django site. We're
using Django 1.7 which is not the latest version on the Django
website. Make sure you're looking at the correct version!

A note on code layout. I prefer to have my projects layed out like
this:

stuff/
	manage.py
	stuff/
		views.py
		models.py
		migrations/
		settings.py
		wsgi.py
		templates/

This is different the defualt Django layout that is created if run
'startproject' and then 'startapp'. I basically skip the 'startapp'
step and just put everything in the same directory.

It's probably worth your effort to set up the admin interface to make
it easy to update your data. In the real world you wouldn't do this,
but I think it's a reasonable short-cut for a class project. The risk
with this is that it could expose security vulnerabilities so you
wouldn't want to really put it into production.

Models
------

The first step is to create your Django models for your project. You
should already have designed the models in your prior homework. If you
haven't, now is the time to think through what the models are and what
fields will be required in each.

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
  Django. Django supplies its own User model for you with a lot of
  built in common features. For now you can just ignore this unless
  you're feeling adventerous. We'll come back to it later in the
  course. Pictures are complicated by the fact that they tend to be
  large. For now, I'd just ignore doing anything with pictures.

Services
--------

Each API will have it's own url for accessing it. These are listed in
your project's urls.py file. Each one will specify the view that is to
be invoked to handle that url.

You'll then create a Django view for each url. The view may handle
both GET and POST requests. You'll need to consult the Django
documentation for how to do this and for how to properly format a json
response from your view.

Remember, this is a four-tier app we're building. The DB is the fourth
/ bottom tier. This layer of servivces you're building now is the
third tier. It should focus on providing access to creating, looking
up, and updating your models. The second tier, when you build it
later, will consume the third tier services and put them together to
form high level functions in your app like providing search, or a
home/start page/screen, etc. Finally the top tier will be an app that
consumes the second tier and generates HTML for a browser (or for for
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
