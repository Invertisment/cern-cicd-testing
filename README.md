Investigation
---------------------

* Scope of query conversion to SQLAlchemy (if self-contained DB solution will be chosen)
  * dbLayer/querying: 2 trivial queries -- `get_record_requests, get_record_entries`
  * `logs/dropbox_querying`: 13 queries, non trivial, requires database knowledge. `get_user_logs, get_user_search_logs, search_tag_logs ...`

* Add personal preferences to github readme
  * Would not choose single central database (Oracle) with multiple schemas because the cleanup would be harder and it is a single point of failure.
  * Docker solution requires some investigation for Windows platform and if it works out then this would be the primary solution with self-contained database. Also this would mean longer set up time for a new developer (can be done via script).

* SLC6 feasibility inside Docker on Windows (Docker + Windows + devicemapper driver)
  * Docker for Windows requires 64bit Windows 10 Pro with Hyper-V available https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows
  * Windows 10 Home lacks some features https://docs.docker.com/docker-for-windows/faqs/#why-is-windows-10-home-not-supported
  * Previous Windows versions (prior to 10) lack features too
  * Some antiviruses may interfere with Docker
  * Workaround: Any OS can be used with Vagrant or VirtualBox to run SLC6 itself or Docker inside of it.

CI engine
---------------------
There are a couple of CI tools available in CERN: GitLab and Jenkins.
Gitlab is mostly suited for small scale development and Jenkins is preferred for larger scale applications.
As we can see in the information in [Jenkins docs](https://jenkinsdocs.web.cern.ch/) -- it should be used when Gitlab is not sufficient to fullfil the project's needs.

:golf:Gitlab CI
* Pros:
  * Integrated with GitLab -- set up is very easy
* Cons:
  * Only one build per project
  * Can't use non-Gitlab for project source hosting

Jenkins
* Pros:
  * More popular, more tutorials and information
  * Multiple builds for project
  * Can be integrated with GitLab
  * Highly configurable steps and workflows
  * Suitable when user wants something specific
* Cons:
  * Requires knowledge and time to maintain

----------------------------
Testing engine
----------------------------

:golf:Use Selenium
* Pros:
  * We don't know the needs for testing so we may choose the technology later
* Cons:
  * Something could be limited


Selenium alternatives:

https://github.com/Blazemeter/taurus
Built on selenium
* Pros:
  * improves dev experience
* Cons:
  * Uses YAML to create config.
  * YAML keywords require knowledge for configuration

https://www.ranorex.com/
Built on selenium
* Pros:
* Cons:
  * Paid
  * Specifies on GUI recognition -- Too much for us, we need a simple solution first
  * Click recording

http://www.telerik.com/teststudio
* Pros:
* Cons:
  * Paid

----------------------------
Testing database
----------------------------
Sometimes application may write data into the cmsDbAccess and eventually to the main DB.
We may want to isolate test runs that two overlapping runs would not break each other.
For that we could create an isolated database or use the same one.
Some database queries (10 or more) use Oracle SQL directly and not SQLAlchemy library.
This means that database cannot be swapped easily and we are bound to use Oracle technology.
Some proposals for this problem:

Have everything as for now, don't change any queries and don't create any new databases:
* Pros:
  * No rewriting of the queries
* Cons:
  * We must use the dev database or a dedicated one
  * If we use a single database tests should check only read-only flows
  * Tests work only online

Have most things as for now and try to place multiple users into the same database:
* Pros:
  * Single instance of Oracle DB
* Cons:
  * We must use the dev database or a dedicated one
  * Table name prefixes solution would require a small change for all SQL statements
  * Different schemas solution would require a small change for all SQL statements
  * Tests work only online
  * If there is a network outage during the test then data will persist in the database. Periodical cleaning would be a workaround.

Have most things as for now and don't change any queries but create databases as we need them:
* Pros:
  * No rewriting of the queries
* Cons:
  * TODO: How long does an instance of Oracle DB start up?
  * TODO: Can Oracle be run locally?
  * If there are new developers, how many databases do we need?
  * If there is a network outage during the test then data will persist in the database. Periodical cleaning would be an option.

:golf:Change queries into SQLAlchemy format:
* Pros:
  * Any free and lightweight database can be used (Most of the system can work normally with sqlite)
* Cons:
  * Need to rewrite some (10 or more) of the queries for Browser and DbAccess
  * Need to change configuration logic for the applications

----------------------------
Testing platform
----------------------------
Each developer must have a convenient way to develop a new test. This means that if tests are run only per commit the developer loses some of her power. If we want to preserve this we must run the tests not only per-commit but also locally when developer wants to do it.

Proposals:

Run web and DB remotely, test from local machine
* Pros:
  * Less changes to the codebase
  * Easier to maintain
  * Development interactivity
  * No virtualization
  * Can run in interactive and non-interactive mode
* Cons:
  * DB port and web are visible for the whole CERN network (a possible security issue)
  * Web can be opened by anyone from CERN network -- tests are fragile and easy to break
  * Local user must have selenium (or other testing library) binary file in local device

Run Web and DB remotely, test from local machine + encrypted SSH port forwarding
* Pros:
  * Nobody sees our webpage and database except us
  * Unintentional test break is hard
  * // from upper one:
  * Development interactivity
  * Less changes to the codebase
  * Easier to maintain
  * No virtualization
  * Can run in interactive and non-interactive mode
* Cons:
  * Harder to set up on both sides
  * Encrypted connection set up needs root for testing vm, possibly even every time a connection must be made
  * // from upper one:
  * Local user must have selenium (or other testing library) binary file in local device

Run directly on openstack SLC6 without exposing any ports
* Pros:
  * No additional local libraries
  * Can run only remotely
  * Direct DB access (DB is hosted inside of SLC6 VM)
  * No virtualization
  * Tests can run per-commit
* Cons:
  * Non interactive development => new tests must be developed using other solution than this one

Run directly on SLC6 on openstack without exposing any ports but with X11 support
* Pros:
  * No additional local libraries
  * Can run in interactive and non-interactive mode
  * No virtualization
* Cons:
  * X11 support for ssh can only be enabled with root user
  * User must enable X11 support when he wants to develop new tests

Force the user to use SLC6 on his computer
* Pros:
  * Can run in interactive and non-interactive mode
  * Same environment as on VM/Prod, only locally
  * Development can happen without internet
  * Remote test execution happens almost in the same way as development
* Cons:
  * Environment set up takes longer for each developer

:golf:Run SLC6 instance from docker or VM
* Pros:
  * Can be run CI server and on VM
  * Same environment as on VM/Prod
  * Less time to set up (everything is preset)
  * Everything is self-contained
* Cons:
  * Docker or VM manager are additional tools who need support
  * Docker image or VM file need to be maintained

---------------------
CD engine
---------------------
If we do not switch from puppet deployment script we will need to edit it.
If we would switch to Jenkins for cmsDbBrowser that would mean to remove puppet code of deployment. And this may not be intended by my project definition.

:golf:No (or as little as possible) changes
* Pros:
  * Less time to config
* Cons:
  * Gitlab or Jenkins will not be used to deploy the code of cmsDbBrowser
  * Puppet will get dirty and fat in the long term

Change to Jenkins or Gitlab
* Pros:
  * Deploy on successful tests
* Cons:
  * Configuration is needed
  * Recreation of a working solution

---------------------
Database migration
---------------------
When new database changes occur inevitably we will have to migrate the testing database to the new layout

Recompile DB binary on schema change (Even on adding new tables)
* Pros:
  * No time setting up the test data
  * Only production queries and models have to be updated
  * Retaining of old test data must be done using SQL (oracle or sqlite) statements and may become too hard to update
* Cons:
  * When schema and code changes we have to update the binary of the database (or multiple DBs)

:golf:Create a new DB before each test (DB creation works right now using SQLAlchemy)
* Pros:
  * Easier to maintain in the long term
  * Only code (models + queries) have to be changed to make E2E tests work again
  * It is more durable when changes arise.
  * Creation queries must be written
* Cons:
  * Requires more time (or reuse of the same insertion scripts) while creating the tests from the beginning
  * Tests contain a helper that has some insertion queries

