Overview
========

In this project you will add the ability for users to create accounts,
login, and to create new listings in your market place. As part of
this, you will extend your APIs to support authentication.

Architecture
------------

There are two discrete components to this project:

   - Adding user account support
   - Adding the ability for users to create new listings

Inevitably, you will end up having to make changes to the particular
low-level model API's that you have created. Don't be overly
sentimental about what you may have already created in Project 2. Now
that you understand what your application needs, you have a much
better understanding of what your APIs need to support. It's natural
to refactor and rewrite like this as you iteratively learn more about
what you're trying to build.

### User Accounts ###

You will _not_ be using Django's built in authentication framework in
your project. Instead, you will be implementing it's functionaliy on
your own. This is because we want to support both cookie based
authentication in the web front end as well as service based
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
the client (web front end, mobile app, etc) and invoke the user model
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
call. However, this would require our web front end or mobile client
to store the user's password for the lifetime of the session. This is
considered poor security practice as it increases the risk that this
information could be stolen. If a password was stolen, it could
potentially lead to accounts at other sites being comproposed if the
user has the same password accross sites. So intead, we store the
passwords in the client only long enough to convert it into a
authenticator which can only be used on this site and for a limited
time.

#### Web front end ####

Your web front end will be responsible for:

   - Create account page: reading new user info on the create account
     page and passing it on to the create account experience
     service. In the real world this page would be protected by
     SSL/TLS to prevent anyone from intercepting the new user's
     passowrd as it's being sent from the browser to the web front
     end.

   - Login page: reading user login info on a login page and passing
     it to a login experience service that would check the
     username/password. The web front end receives an authenticator
     back from the experience service if the username/password matches
     some registered user. The web front end then needs a place to
     remember this authenticator across future web requests from the
     browser client. We will do this by storing the authenticator into
     an HTTP cookie that is returned from the login process. The
     browser will pass back this cookie on all subsequent requests and
     then our web front end can in turn pass this authenticator back
     into each experience service call.

   - Logout page: logging out the user by removing the cookie storing
     the authenticator. This means that future requests to the web
     front end will not have the authenticator and so future web
     requests will no longer be processed using the user's identity.

#### Authenticators ####

The idea of an authenticator is to create a weaker more limited
version of the authentication information represented by a
username/password.

We will implement them as an authenticator model with fields for
user_id, authenticator, and date_created. Authenticator will be the
primary key.

When a user logs in, we create a new random authenticator and add it
along with the logged in user's id to this table.

When a model API wants to check that an authenticator is valid, it
will look up the authenticator in this table. If a match is found, we
can assume the request is being made by the specified user_id. If a
match is not found, then we assume the request is invalid.

When a user logs out we should delete their
authenticator. Periodically, we can delete authenticators that are
more than a certain amount of time old. This prevents a stolen
authenticator from being used indefinitely.

Since the web front end stores authenticators in browser cookies, the
user could tamper with the authenticor or even forge a new one. Also,
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

You will extend your Project 3 app with the following:

   - Extending your web front end to have create account, login,
     logout, and create listing pages along with associated HTML
     templates for rendering.

   - Extending your experience services to have a create account,
     logout, login and create listing service.

   - Exnteding your model APIs to allow creating and authenticating
     users and creating listings.

   - Verifying correct authentication information in the create
     listing model API call so that only authenticated users can
     create new listings.

### Django forms ###

Django forms are a mechanism to validate user input. You will be
adding two forms to your project: one for creating users and one for
creating listings. You will create one Django form for each of these
use cases in your web frontend.

### Password management ###

Django supplies well tested code for hashing and checking
passwords. You should use this code rather than trying to create your
own versions. Use `django.contrib.auth.hashers` module's
`make_password` and `check_password`. My sameple
Project 2 code shows an example of using `make_password`.

Good further reading on salting, hashing and attempts to crack
passwords at https://crackstation.net/hashing-security.htm

Creating the random authenticators can be done by generating a random
string as such:

    import os
    import base64

    authenticator = base64.b64encode(os.urandom(32)).decode('utf-8')

This will generate 256 bits of randomness which is pretty hard for an
attacker to guess.

And then storing it into a cookie named auth in the web front end:

    resp = login_exp_api (username, password)
    authenticator = resp['authenticator']
    response = render(...)
    response.set_cookie("auth", authenticator)

And reading it back in a future web request:

    auth = request.COOKIES['auth']
    resp = create_listing_exp_api(auth, ...)

As a side note, it's not great form to invent this authenticator
generating and checking logic but there's no good Django library
available and in the name of simplicity I'm trying to avoid using more
third party libraries. Best practice would be to use a third party to
take care of coding errors, considering things like timing attacks,
using sufficient randomness, etc.

### Cookie management ###

See the Django documentation for setting, reading and clearing
cookies. Roughly, you can use `HttpResponse`'s
`set_cookie` method to set a cookie and
`HttpRequest`'s `COOKIES` dictionary to look up
cookies passed in by the browser.



