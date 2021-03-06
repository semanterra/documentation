Start Contributing to Open States
=================================

.. note::

    This document is very much a work-in-progress, feel free to `suggest contributions <http://github.com/openstates/documentation>`_.

Scrapers are at the core of what Open States does, each state requires several custom scrapers designed to extract bills, legislators, committees, votes, and events from the state website.  All together there are around 200 scrapers, each one essentially independent, which means that there is always more work to do, but plenty of prior work to learn from.

Code of Conduct
---------------

Open States is a project that can only exist because of the fantastic community that has worked on it over the years.
In the interest of keeping this community a healthy and welcoming place we have a :doc:`code-of-conduct` and encourage you to familiarize yourself with it.

Prerequisites
-------------

This guide assumes a basic familiarity with:
    - using the command line
    - git
    - Python

No worries if you aren't an expert though, we'll walk you through the steps.  And as for Python, if you've written other languages like Javascript or Ruby you'll probably be just fine.  Don't be afraid to :ref:`ask for help <getting-help>` either!

Getting Started
---------------

First thing you will need to do is get a working development environment on your local machine.  We'll do this using Docker.  No worries if you aren't familiar with Docker, you'll barely have to touch it.

**Step 1)** Install Docker (and docker-compose) if not already installed on your local system.

    * On OSX: `Docker for Mac <https://docs.docker.com/docker-for-mac/>`_ is perhaps the easiest way.
    * On Windows: `Docker for Windows <https://docs.docker.com/docker-for-windows/>`_
    * On Linux: Use your package manager of choice or `follow Docker's instructions <https://docs.docker.com/engine/installation/linux/>`_.
    * `Generic instructions from Docker <https://docs.docker.com/compose/install/>`_.

**Step 2)** Ensure that Docker (and docker-compose) are installed locally and check their versions::

    $ docker --version
    Docker version 17.03.0-ce, build 60ccb22
    $ docker-compose --version
    docker-compose version 1.11.2, build dfed245

Of course, your versions may be newer. The minimum required versions for Open States are:

    * 1.9.0 of Docker
    * 1.10.0 of Docker Compose

**Step 3)** We'll fork and clone the main `Open States scraper repository <https://github.com/openstates/openstates>`_.

  * Visit https://github.com/openstates/openstates and click the 'Fork' button.
  * Clone your fork using your tool of choice or the command line::

        $ git clone git@github.com:yourname/openstates.git
        Cloning into 'openstates'...

At this point you'll have a local ``openstates`` directory.  Let's go ahead and look at it::

    $ cd openstates
    $ ls
    AUTHORS            README.rst         experimental       requirements.txt
    Dockerfile         billy_settings.py  manual_data        scripts
    LICENSE            docker-compose.yml openstates         setup.py

There are a few top level text files, some docker files, which we'll come back to shortly, and some directories.  The directory we care about is the one called ``openstates``.::

    $ ls openstates
    __init__.py dc          in          mn          nj          pr          va
    ak          de          ks          mo          nm          ri          vi
    al          fl          ky          ms          nv          sc          vt
    ar          ga          la          mt          ny          sd          wa
    az          hi          ma          nc          oh          tn          wi
    ca          ia          md          nd          ok          tx          wv
    co          id          me          ne          or          ut          wy
    ct          il          mi          nh          pa          utils

This directory is a python module with 50+ subpackages, one for each state.

Let's look inside one::

    $ ls openstates/nc
    __init__.py    bills.py       committees.py  people.py      votes.py

Some will differ a bit, but all will have ``__init__.py``, ``bills.py``, and either ``legislators.py`` or ``people.py``.  These are the NC scrapers that collect these objects.

**Step 4)** Let's finish setting up our environment by creating the database::

    $ docker-compose up database
    
The output of the '... up database...' command should end with::
    NETWORK  [thread1] waiting for connections on port 27017    

If, instead, these commands fail, check that your copy of docker-compose is a recent vintage.

.. note::

    If you encounter an error like::

        database_1        | chown: changing ownership of '/dev/stdout': Permission denied
        database_1        | chown: changing ownership of '/dev/stderr': Permission denied

    It is likely related to `Docker Issue #31243 <https://github.com/docker/docker/issues/31243>`_
    which you can work around by adding ``tty: true`` in your docker-compose's ``database``
    entry.

At this point we have two docker images:

database
    A MongoDB database that we're going to use to store our scraped data.
openstates
    The image that will run our scrapers.

And we're ready to go!

Running Our First Scraper
-------------------------
**Step 5)** Open a new terminal tab in preparation for the '... run openstates ...' command.

**Step 6)** Choose a state. 

**Step 7)** Let's run <your state's> legislator scraper (substitute your state for 'nc' below) ::

    $ docker-compose run --rm openstates nc --legislators --fast

The parameters you pass after ``docker-compose run --rm openstates`` are passed to ``billy-update``.  Here we're saying that we're running NC's scrapers, just want to run the legislators scraper, and that we want to do it in "fast mode."  A full description of ``billy-update`` is available `in the billy docs <http://docs.openstates.org/projects/billy/en/latest/scripts.html#billy-update-state>`_.

So, ``billy-update`` kicks off a full scrape of NC's current legislators.  You'll start seeing things like::

    18:15:16 INFO billy: billy-update abbr=nc
        actions=scrape,import,report
        types=legislators
        sessions=2017
        terms=2017-2018
    18:15:18 INFO scrapelib: GET - http://www.ncga.state.nc.us/gascripts/members/memberListNoPic.pl?sChamber=Senate
    18:15:19 INFO scrapelib: GET - http://www.ncga.state.nc.us/gascripts/members/viewMember.pl?sChamber=Senate&nUserID=392
    18:15:20 INFO billy: Save person John M. Alexander, Jr.
    18:15:21 INFO scrapelib: GET - http://www.ncga.state.nc.us/gascripts/members/viewMember.pl?sChamber=Senate&nUserID=396
    18:15:22 INFO billy: Save person Deanna Ballard
    18:15:22 INFO scrapelib: GET - http://www.ncga.state.nc.us/gascripts/members/viewMember.pl?sChamber=Senate&nUserID=369
    18:15:23 INFO billy: Save person Chad Barefoot

The first thing is billy's *run plan*, what it is going to try to scrape.
This is presented as a sanity check, and each of these values can be controlled by different command line parameters.
In this case we see we're running the *scrape*, *import*, and *report* actions for nc legislators for 2017-2018.  The scraper chose the most recent available session/term for us.

Depending on the scraper you run, this part takes a while.  Some bill scrapers can take hours to run, but most legislator scrapers are a few minutes.

At the end of the scrape you should see a message like::

    18:19:18 INFO billy: Finished importing 169 legislator files.

This means that the data is now in the database.  Congratulations, you just ran your first state scrape!

**Step 8)** To access the data you just fetched, you can connect to the database as follows: ::

    $ docker-compose run --entrypoint mongo database mongodb://database
    
This loads the mongodb shell. You may close the mongo connection with::
    > quit()

You can also view the data in the ``data`` directory of the project root.

.. note::
    It is of course possible that the scrape fails.  If so, there's a good chance that isn't your fault, especially if it starts to run and then errors out.  Scrapers do break, and there's no guarantee North Carolina didn't change their legislator page yesterday, breaking our tutorial here.

    If that's the case and you think the issue is with the scraper, feel free to get in touch with us or `file an issue <https://github.com/openstates/openstates/issues>`_.

Next Steps
----------

At this point you're ready to run scrapers and contribute fixes.

Right now the most important task in front of us is converting scrapers to pupa, see :doc:`pupa-conversion` and consider helping us out today!

.. _getting-help:

Getting Help
------------

Right now the best way to get help is to `join our Slack <https://openstates-slack.herokuapp.com/>`_, plenty of the core team and other contributors are around to answer any questions you may have.
