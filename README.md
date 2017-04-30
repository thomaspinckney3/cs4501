# cs4501 Spring 2017
Internet Scale Applications
===========================

This course will provide a survey of methods for building large-scale internet websites and mobile apps. The intent is to build upon prior classes by explaining how theory meets practice. Topics covered will include scaling, security, large team software engineering, etc. There will be a series of cumulative course projects resulting in students building a working marketplace website. Weekly readings from industry and academic sources will complement the weekly lecture.

Prerequisites
--------------

CS3240 (Advanced Software Development) or equivalent experience building non-trivial Python/Django web applications is required. Understanding issues around persistence, databases, concurrency, networking etc along with Linux experience will be extremely helpful in this class.

A course project will be developed using Python, Django, MySQL, and Docker containers. A basic familiarity with HTML and CSS is preferred but not strictly required.

Project Overview
-----------------

You will work in a team of three to four students to build a working marketplace website. Marketplaces such as Airbnb, Uber, DonorsChoose, eBay, Etsy and Watsi are both highly profitable when they succeed and technically challenging to build. It's up to you to decide what kind of marketplace your team will build. Specific instructions for each project will be posted and linked below thoughtout the semester.

Due dates are listed below for each part of the project. These may be revised as the term goes on based on how everyone is doing. You will be graded as a team (unless unusual conditions warrant giving different grades). Grades will be based on: completeness of solution, correctness of solution, and being on time completing each assignment.

Course materials
-----------------

Syllabus and project assignments will be here in GitHub. Feel free to fork and send PRs with corrections, additions or any other changes you think would help fellow/future students.

Lecture slides are in Google drive at https://drive.google.com/folderview?id=0BzWAJQVnIIRYfk9JUmwtbUVKS1pqb0k0Q2ZYU3pPZ3gxV2VnVDctVU51VjFYTTVaR25xR3c&usp=sharing. Note that prior semester's slides can be found here for future lectures, but they may be out of date / subject to update this semester. Generally, the further out a lecture is the more likely that the slides have not been reworked for this semester.

Readings are listed for each week and should be completed BEFORE the week they are assigned so that we can discuss in class.

Course Topics (subject to revision as course progresses)
---------------------------------------------------------

1. Anatomy of the modern internet, websites and mobile apps (Wed 1/18 and Mon 1/23)
  - Trace a web request from browser to server and back
  - Overview of three/four tier app architecture
  - Overview of data center architecture: load balancers, switches, storage, servers
  - Overview of Model View Controller (MVC) design pattern
  - Overview of Docker and course project
  - Project: set up Docker container with Hello World Python/Django web app. https://github.com/thomaspinckney3/cs4501/blob/master/Project1.md
  - Resources: Django Getting Started docs https://docs.djangoproject.com/en/1.8/intro/ and https://docs.docker.com as well as  https://www.udacity.com/course/web-development--cs253 or http://www.codecademy.com/en/tracks/web if needed for HTML and CSS intro
2. Requirements and documentation
  - User stories
  - Product requirement documents
  - Design and architecture documents
3. Service based architecture, part I
  - Loosely coupled systems cooperating via services
  - Errors in the real world: failing gracefully and constantly
  - REST and API design
  - Reading: Service Oriented Architecture at Netflix http://techblog.netflix.com/2012/06/netflix-operations-part-i-going.html , https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/ and http://martinfowler.com/microservices/ . Optional, for more in-depth reading check out http://shop.oreilly.com/product/0636920033158.do
4. Databases
  - CAP theorem / Strong vs eventual consistency / Availability vs consistency
  - Common database use cases -- transactional data, logging, caches, etc
  - Sharding and partitioning
  - Case study: Cassandra
  - Reading: CAP theorem http://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed and Eventual Consistency http://www.allthingsdistributed.com/2008/12/eventually_consistent.html, a counter-argument http://labouseur.com/courses/db/Stonebraker-on-NoSQL-2011.pdf and consistent hashing for sharding http://pdos.csail.mit.edu/papers/chord:sigcomm01/chord_sigcomm.pdf . Optional, Google Spanner http://research.google.com/archive/spanner.html
- Project: Start building site: write user stores, create data models and service layer of project. https://github.com/thomaspinckney3/cs4501/blob/master/Project2.md
5. Service based architecture, part II: Multi-screen development for mobile and desktop
  - Proliferation of channels for consuming apps
  - App and experience logic server side vs client side
  - Responsive web design vs building different apps for different devices
  - Consistent experiences on mobile native, mobile web, desktop web, tablet, email etc
  - Service composition
6. Security
  - SQL injection and XSS
  - Denial of Service
  - Untrusted content (e.g.image exploits in UGC)
  - Stack smashing
  - Phishing
  - Internal IT and employees as attack vector
  - Project: Build view layer. https://github.com/thomaspinckney3/cs4501/blob/master/Project3.md
  - Resources: Bootstrap http://getbootstrap.com/getting-started/ and Django templates https://docs.djangoproject.com/en/1.8/topics/templates/
  - Resource: Django security features: https://docs.djangoproject.com/en/1.8/topics/security/
15. Messaging and queing
  - Integrating online and offline processing / Sync vs async
  - Queueing systems
  - Reading: Kafka at LinkedIn https://engineering.linkedin.com/kafka/kafka-linkedin-current-and-future and
  - Quiz #1
10. Search
  - Basics of information retrieval and search engines
  - Case study: ElasticSearch for site search backend
  - Recommendations and browsing as alternatives to search
  - Reading: Incorporating behavioral data in search http://research.microsoft.com/en-us/um/people/sdumais/SIGIR2006-fp345-Ranking-agichtein.pdf
  - Project: user accounts and user generated content. https://github.com/thomaspinckney3/cs4501/blob/master/Project4.md
8. Users and reputation
  - User generated content (UGC): reviews, wiki posts, comments
  - Graph methods for reputation
  - Spam filtering
  - Case study: Amazon reviews
  - Reading: Analysis of StackOverflow http://www.cs.cmu.edu/~ymovshov/Papers/asonam_2013.pdf
7. Speed
  - Importance of speed for users
  - What determines page load speed
  - Service orchestration and asynchronous processing
  - Content delivery networks
  - Case study: memcached
  - Reading: http://radar.oreilly.com/2009/07/velocity-making-your-site-fast.html
  - Resources: Python futures https://docs.python.org/3.4/library/concurrent.futures.html and Django cache APIs https://docs.djangoproject.com/en/1.8/topics/cache/
  - Project: search. https://github.com/thomaspinckney3/cs4501/blob/master/Project5.md
8. Testing and DevOps
  - Automated testing for services, web pages and mobile apps
  - Continuous integration and deployment
  - Monitoring and alerting
  - Infrastructure as code
  - Reading: Testing lifecycle at LinkedIn https://engineering.linkedin.com/41/testing-lifecycle-linkedin, continuous delivery at Facebook http://www.infoq.com/presentations/Facebook-Release-Process and dated but useful Google talk http://googletesting.blogspot.com/2008/11/clean-code-talks-unit-testing.html
11. TBD
   - Project: Enhancements. https://github.com/thomaspinckney3/cs4501/blob/master/Project6.md
12. Advertising
  - Map of advertising ecosystem
  - Display advertising: targeting, trafficking and optimization
  - Search advertising: advertising as machine learning problem
  - Reading: Ad Auctions (sections I and II in particular) http://users.cms.caltech.edu/~adamw/courses/241/lectures/search2.pdf
13. Parallel batch processing
  - Map / Reduce
  - Hadoop / Spark for data analysis
  - Project. https://github.com/thomaspinckney3/cs4501/blob/master/Project7.md
13. AB-testing and Analytics
  - Understanding the effect of product changes
  - Implementing tracking
  - Conversion funnels
  - AB testing and statistically significant changes
  - Reading: AB testing at Spotify http://www.slideshare.net/alisarrafi3/ab-testing-at-spotify?next_slideshow=1 and http://www.slideshare.net/dj4b1n/ab-testing-pitfalls-and-lessons-learned-at-spotify-31935130 . Spark http://www.cs.berkeley.edu/~matei/papers/2010/hotcloud_spark.pdf . Optionally, can read more at https://play.google.com/store/books/details?id=VfVvAAAAQBAJ&source=productsearch&utm_source=HA_Desktop_US&utm_medium=SEM&utm_campaign=PLA&pcampaignid=MKTAD0930BO1&gl=US&gclid=COGFp6bcwMcCFQikNwodIH0IbA&gclsrc=ds
14. Traffic -- SEO, SEM, Social, Paid Marketing, Email
  - Survey of driving traffic to a site / paid vs organic
  - Overview of SEO
  - Quantitative marketing, attribution models, ROI calculations
  - Case study: BuzzFeed
  - Reading: https://hbr.org/2007/05/viral-marketing-for-the-real-world
15. TBD
  - Quiz #2

Grading
-------

Grades will be computed approximately as follows:
10% class participation (based on discussion in class)
40% two quizzes
50% class project

More detailed project grading guidelines can be found at https://docs.google.com/document/d/1nZrh3uvwQFh3wf3mE8kuH8LL12j5WbLh42-j6AJwr38/edit?usp=sharing

Academic Honor Code
-------------------

All work products that you submit for this class -- whether projects, quizes or anything else -- is assumed to be your own
creation UNLESS you specifically call out that you are including someone else's work, either in spirit or verbatim. Violating
this honor code by presenting work as your own when in fact it is not your own will result in severe disciplinary action
potentially including referral to the University Honor Code comittee who may recommend expulsion.

If there is reason for you to submit other's work in your github repo then you must cite them as the author of
the work. If their work has a copyright notice on it, make sure that copyright notice is perserved and contained in your work
saying "Portions of this work copyright DATE by AUTHOR". If the work has a license agreement associated with it, make sure you 
are complying with the terms of the license agreement. If there is no license or copyright, you must still explicitly list the 
author(s) whose work you are including. This should be in the form of a comment at the top of the source file including their 
work.

Finally, if you are using someone else's work as the substantial basis of your work (even if you're not literally copying it
character by character) then you are still copying their work and you must cite them.

Office Hours
-------------

The teaching staff for this semester are:

  - Tom Pinckney (Lecturer): tp3ks@virginia.edu
  - Ian Zheng (TA): yz9fy@virginia.edu
  - Zakey Faieq (TA): zaf2xk@virginia.edu
  - Christine Tsou (TA): cyt3ea@virginia.edu

There will be one set of office hours for project questions and demos and another time for general class questions. Each project is two weeks. During the first week a project is assigned, the project review hours will be about how to do the project. During the second week that each project is assigned, the project hours will include mandatory demos of your team's progress. If you are unable to make the demo hours, please contact the teaching staff to arrange an alternative time.

  - Office Hours, Project Review, Help, and Demo Hours (led by TAs): 
    - Ian:
      - Monday & Wednesday: 3:30 - 4:30 in Rice 436
    - Zakey:
      - Tuesday: 5:00 - 6:30 in Rice 340
    - Christine:
      - Sunday: 1 - 2 in Rice 340
      - Thursday: 4:45 - 6:15 in Rice 340
    
  - General Q&A (led by Tom): Monday 1-2pm but please email/slack me to reserve a time first.

There is also always help from the teaching staff and your fellow classmates through slack at https://cs4501-isa.slack.com.

Remember, it's always better to ask questions, bring up problems, etc. sooner versus later.
