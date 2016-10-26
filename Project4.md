Overview
========

In this project you will add the ability for users to create accounts,
login, and create new listings in your marketplace. As part of
this, you will extend your APIs to support authentication.

Architecture
------------

There are two discrete components to this project:

1. Adding user account support
2. Adding the ability for users to create new listings

Inevitably, you will end up having to make changes to the particular
low-level model APIs that you have created. Don't be overly
sentimental about what you may have already created in Project 2. Now
that you understand what your application needs, you have a much
better understanding of what your APIs need to support. It's natural
to refactor and rewrite like this as you iteratively learn more about
what you're trying to build.

### User Accounts ###

You will _not_ be using Django's built in authentication framework in
your project. Instead, you will be implementing it's functionality on
your own. This is because we want to support both cookie-based
authentication in the web front-end as well as service-based
authentication in your experience services. The latter is because we
want the experience service layer to be easily callable from native
mobile apps.

#### Model and API changes ####

You will create a new model for users (if you do not already have one)
and associated APIs for creating a user and authenticating a user. You
will also create a new model and associated APIs for authenticators
(see below). Finally, you will create experience service calls for
creating users, logging users in and logging them out.

Our login experience service will take a username and password from
the client (web front-end, mobile app, etc) and invoke the user model
login API. This model API will check the password is correct and if
so, return an _authenticator_ back to the experience service. The
experience service will then return the authenticator back to the
client.

This authenticator will be passed back into future experience service
calls to prove the user's identity. The experience services will in
turn pass this authenticator into model API calls as proof that the
user is authenticated. Some experience services will treat the
authenticator as optional (such as for the home page service) while
others will require it (the create listing service).

We could theoretically pass the username and password into every
experience service call and in turn into every model API
call. However, this would require our web front-end or mobile client
to store the user's password for the lifetime of the session. This is
considered poor security practice as it increases the risk that this
information could be stolen. If a password was stolen, it could
potentially lead to accounts at other sites being compromised if the
user has the same password across sites. So instead, we store the
passwords in the client only long enough to convert it into a
authenticator which can only be used on this site and for a limited
time.

#### Web front-end ####

Your web front-end will be responsible for:

   - __Create account page__: reading new user info on the create account
     page and passing it on to the create account experience
     service. In the real world this page would be protected by
     SSL/TLS to prevent anyone from intercepting the new user's
     password as it's being sent from the browser to the web front
     end.

   - __Login page__: reading user login info on a login page and passing
     it to a login experience service that would check the
     username/password. The web front-end receives an authenticator
     back from the experience service if the username/password matches
     some registered user. The web front-end then needs a place to
     remember this authenticator across future web requests from the
     browser client. We will do this by storing the authenticator into
     an HTTP cookie that is returned from the login process. The
     browser will pass back this cookie on all subsequent requests and
     then our web front-end can in turn pass this authenticator back
     into each experience service call.

   - __Logout page__: logging out the user by removing the cookie storing
     the authenticator. This means that future requests to the web
     front-end will not have the authenticator and so future web
     requests will no longer be processed using the user's identity.

#### Authenticators ####

The idea of an authenticator is to create a weaker, more limited
version of the authentication information represented by a
username/password.

We will implement them as an `Authenticator` model with fields for
`user_id`, `authenticator`, and `date_created`. `authenticator` will be the
primary key.

When a user logs in, we create a new random authenticator and add it
along with the logged in user's id to this table.

When a model API wants to check that an authenticator is valid, it
will look up the authenticator in this table. If a match is found, we
can assume the request is being made by the specified `user_id`. If a
match is not found, then we assume the request is invalid.

When a user logs out we should delete their
authenticator. Periodically, we can delete authenticators that are
more than a certain amount of time old. This prevents a stolen
authenticator from being used indefinitely.

Since the web front-end stores authenticators in browser cookies, the
user could tamper with the authenticator or even forge a new one. Also,
in the real world, the experience services would have to be exposed
outside the firewall so that mobile clients could call them. Thus
anyone could make an experience service call and pass it a potentially
forged authenticator.

The solution we use is to make authenticators hard to forge and easy
to detect modifications. Each authenticator is a large random number
which serves as a primary key for the authenticator model. Thus, for
an attacker to forge an authenticator they would have to guess the
random number associated with the user id. We can make the
authenticator sufficiently big to make it sufficiently unlikely that
anyone can guess it.

In practice, mobile clients would also use SSL/TLS to secure their
communication with the experience services to make sure that no one
intercepted the authenticators they pass in API calls.

### Creating Listings ###

You will add new a new listing model(s), model API(s), and experience
service(s) to support adding new listings. Only logged in users should
be able to create listings.

Implementation
--------------

You will extend your Project 3 app by implementing the following changes:

- Extend your web front-end to have "create account", "login",
 "logout", and "create new listing" pages, along with associated HTML
 templates for rendering.

- Extend your experience services to have a "create account",
 "logout", "login" and "create new listing" service.

- Extend your model APIs to allow for creating and authenticating
 users, as well as creating new listings.

    - Verify that the given authentication information is correct
         in any "create new listing" API calls. This will allow only
         authenticated users to create new listings.

### Django forms ###

Django forms are a mechanism to validate user input. You will be
adding two forms to your project: one for creating users and one for
creating listings. Your web front-end will use one Django form for
each of these use cases.

You can (and should!) pass each form into the template rendering that page,
so that the form can be rendered by Django rather than you recreating
the HTML for each form element. This will help keep your template and
form in sync, since you only need to change the form definition.

### Password management ###

Django supplies well-tested code for hashing and checking
passwords. You should use this code rather than trying to create your
own versions. Use the `django.contrib.auth.hashers` module's
`make_password` and `check_password`. My
[sample Project 2 code](https://github.com/thomaspinckney3/stuff-models/blob/master/stuff/main.py)
shows an example of using `make_password`. Read more about password hashing in
the official Django documentation:

<https://docs.djangoproject.com/en/1.8/topics/auth/passwords/#module-django.contrib.auth.hashers>

There is additional reading on salting, hashing, and password-cracking at:

<https://crackstation.net/hashing-security.htm>

Creating the random authenticators can be done by generating a random
string as such:

```PYTHON
import os
import hmac

# import django settings file
import settings

authenticator = hmac.new(
        key = settings.SECRET_KEY.encode('utf-8'),
        msg = os.urandom(32),
        digestmod = 'sha256',
    ).hexdigest()
```

This will generate a 256-bit random sequence, which should be pretty hard for an
attacker to guess. It's also [pretty
unlikely](http://stackoverflow.com/questions/4014090/is-it-safe-to-ignore-the-possibility-of-sha-collisions-in-practice) to collide with a previously
generated authenticator that may already be in the database. To be 100% safe, however,
you should check that the newly generated authenticator does not
in fact exist in the database.

As a side note, it's not best security practice to invent this authenticator
generating and checking logic ourselves. However, there is no good Django library
available currently to accomplish our goal, and in the name of simplicity we
will try to avoid using more third party libraries. The best practice would be
to use a open-source (and thus peer-reviewable) third party library to take care
of coding errors, other security concerns, use of sufficient
randomness, speed, and so on.

#### Web front-end authentication checking skeleton code ####

Here is some pseudo-Python illustrating how the web front-end might
implement the `login` and `create_listing` views.

```PYTHON
"""
exp_srvc_errors.py: where I put some HTTP error status codes that the experience
service can return.
"""
import exp_srvc_errors

def login(request):
    # If we received a GET request instead of a POST request
    if request.method == 'GET':
        # display the login form page
        next = request.GET.get('next') or reverse('home')
        return render('login.html', ...)

    # Creates a new instance of our login_form and gives it our POST data
    f = login_form(request.POST)

    # Check if the form instance is invalid
    if not f.is_valid():
      # Form was bad -- send them back to login page and show them an error
      return render('login.html', ...)

    # Sanitize username and password fields
    username = f.cleaned_data['username']
    password = f.cleaned_data['password']

    # Get next page
    next = f.cleaned_data.get('next') or reverse('home')

    # Send validated information to our experience layer
    resp = login_exp_api(username, password)

    # Check if the experience layer said they gave us incorrect information
    if not resp or not resp['ok']:
      # Couldn't log them in, send them back to login page with error
      return render('login.html', ...)

    """ If we made it here, we can log them in. """
    # Set their login cookie and redirect to back to wherever they came from
    authenticator = resp['resp']['authenticator']

    response = HttpResponseRedirect(next)
    response.set_cookie("auth", authenticator)

    return response

def create_listing(request):

    # Try to get the authenticator cookie
    auth = request.COOKIES.get('auth')

    # If the authenticator cookie wasn't set...
    if not auth:
      # Handle user not logged in while trying to create a listing
      return HttpResponseRedirect(reverse("login") + "?next=" + reverse("create_listing")

    # If we received a GET request instead of a POST request...
    if request.method == 'GET':
        # Return to form page
        return render("create_listing.html", ...)

    # Otherwise, create a new form instance with our POST data
    f = create_listing_form(request.POST)

    # ...

    # Send validated information to our experience layer
    resp = create_listing_exp_api(auth, ...)

    # Check if the experience layer said they gave us incorrect information
    if resp and not resp['ok']:
        if resp['error'] == exp_srvc_errors.E_UNKNOWN_AUTH:
            # Experience layer reports that the user had an invalid authenticator --
            #   treat like user not logged in
            return HttpResponseRedirect(reverse("login") + "?next=" + reverse("create_listing")

    # ...

    return render("create_listing_success.html", ...)
```

Note, the pattern of passing a `GET` field `next` into the login
page in order to specify where the user should be redirected upon successful login.
The `create_listing` view can use this when it finds that the current user is not
logged in. They can be redirected to the login page with next set to the URL
for the `create_listing` view. Then when they complete logging in, the login view
will redirect them back to `create_listing`. The URL will look something like this:

    http://localhost:8000/account/login/?next=CreateListing

Also notice the pattern of each view handling both rendering via `GET` and
`POST`
requests. The form for logging in or creating a listing will initially render
via a `GET` and then be `POST`'ed upon submission. The `POST` handler will either error out
and return the same rendered form (plus some helpful errors), or it will succeed and return
the user to wherever they're supposed to go next.

### Some Django best practices (Optional) ###

#### Python decorator ####
Suppose later your site has more services like `create_list` that requires the user
to be authenticated. One option is to rewrite the authenticating code for each
of the views. This violates the software engineering principle of DRY (Don't Repeat Yourself).

Before you begin to worry: Python has the idea of nested functions, which in turn powers the idea of a _decorator_.
You can think of decorator as an on-the-fly modification to a function. In this case you may consider
creating a decorator to authenticate the user. It should look something like as follows:

```PYTHON
def login_required(f):
    def wrap(request, *args, **kwargs):

        # try authenticating the user
        user = _validate(request)

        # authentication failed
        if not user:
            # redirect the user to the login page
            return HttpResponseRedirect(reverse('login')+'?next='+current_url)
        else:
            return f(request, *args, **kwargs)
    return wrap
```

Python provides syntactic sugar to attach a decorator to a function as follows:

```PYTHON
@login_required
def create_listing(request):
   # [...]
```

You then just need to prepend `@login_required` to each view that needs authentication. Come to the office
hours if you want to know more about decorators.

#### Slightly advanced Django forms ####
Django ModelForms provide nice predefined behaviors in a majority of use cases.

However, there are some cases in which you want to customize the default behaviors. In these cases, you probably
want to look into overriding the \<field\>_clean() method, which provides additional customized checks for a
model field (e.g. if the email is a UVa email), and overriding the `is_valid()` method, which provides addition customized
checks for a model as a whole. You can also do things like append customized error messages to a field and render that
message when the form does not validate. Come to the office hours if you want to know more about Django forms.


What to turn in
----------------

Please turn in a tag to your GitHub repository (e.g. "project4") to your assigned TA when your team has completed the assignment.

Some additional reminders:
  - As before, create data fixtures so that course staff can start your project with necessary database tables filled in with test data.
  - Provide user stories and unit tests accordingly.
  - We expect `docker-compose up` to work out of box! Make sure to test your
    code with a clean database (that will be loaded wth fixtures at runtime).

