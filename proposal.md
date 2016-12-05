# PyCon Proposal


:Title: Lobbing the Holy Hand Grenade: Fixing Python packaging to nullify the 
dangerous rabbit-den that lurks within the sunset of Python 2 support 
:Duration: 30/45 minutes
:Level: Intermediate
:Categories: General, Python 2, Python 3, Packaging, Best Practices.

:Authors: Matthias Bussonnier, Michael Pacer, Thomas Kluyver, MinRK


Summary
=======


> "Four shalt thou not count, neither shalt thou count two, excepting that thou
then proceedeth to three."

Python 3 has been around for more than eight years, and much of the Python
ecosystem is now available both on Python 2 and Python 3, often using a single
code base. There comes a time in the life of such libraries when the authors
consider stopping support for Python 2 in a new major version.

While it may seem simple to cease support for Python 2 – if you did not want
wreak havoc for users who wish to stay on Python 2 – you just made your life
much more difficult. Put simply, up until recently it was not possible to tag
a release as Python 3 only, today it is possible. 

Like any maintainer of a widely used library, we want to ensure that users
still using Python 2 can still have a functioning library, even after
development proceeds in a way that does not support Python 2. One way to ensure
that is to allow easy installation older versions; another way is to have a
parallel "Long Term Support" variant that a different community of developers
supports going forward. At the same time, it is not reasonable to expect users
to explicitly pin maximal version dependencies in all of their projects and
track those that they depend on (including those that they were not actively
developing), when all they wish is to avoid having a Python 2 incompatibility
issue.

In order to solve this problem, and thereby make both users' and maintainers'
lives easier, we ventured into the rabbit-hole called Packaging.

Though we set off with a singular quest, our tale roves through many lands.
We'll narrate the story of our amending PEPs through discussions with the
autonomous collective, our journeys alongside the knights of setuptools,
our efforts in building the ramparts of the pypa/Warehouse Castle, battles with the
dragons of Pip, and errands in the "land of no unit tests" otherwise known as PyPI
legacy.

By the end of the above tale, the careful beholder will be aware of the
hazards of migrating their libraries to require Python 3, and of strategies for
overcoming these issues.


Description
===========

This talk will have 3 main parts:

In the first part, we will describe our quest, focusing on movements toward Python 3 and the concerns that arise for library developers who wish to end support for Python 2. 

In the second part, we will describe several ways to approach the transition
itself, complete with a discussion of the merits and demerits of each approach.
In particular, we will focus on our new solution that avoids many of the
pitfalls of earlier solutions. 

In the third part, we describe the work needed in order to make to implement
the newest solution and how it avoids unleashing rabbits on unsuspecting Python 2 users.

Audience
========

Library authors, in particular library authors interested in releasing a
version of their library that requires Python 3 without breaking Python 2 users'
systems.

Users and developers of Python 2 libraries, who want to make sure their systems
don't upgrade to incompatible versions of once-compatible packages.

User and developers of Python 3 libraries who care about Python 2 users and
developers.


Objective
=========

 - Make users and developers aware of the [Python 3
   statement](http://www.python3statement.org/) as a resource to inform users
   and developers that some packages are migrating to require Python 3, and how
   to do it without breaking users' systems.

 - Make users aware that they should be using pip 9+ and setuptools 24.3+ to avoid
   problems in their Python 2 installation.

 - Make developers aware of the recent changes in Python Packaging
   (`python_requres` metadata) and how to make use of it.

 - The traps not to fall into if you plan to release a Python 3 only package
   version.


Detailed Abstract
=================

1. Intro The Python 3 statement
-------------------------------

A growing number of libraries and library authors have announced that they are
planning on stopping support for Python 2, no later than 2020, if not already.
It is also known that a number of users will not migrate their Python 3 by that
day.

Currently releasing a Python-3 only version of the library while ensuring that
Python 2 user will still install the last Python 2 compatible version not
upgrade by mistake is hard.

Ensuring regardless of Python version in use, library upgrade is as seamless as
possible is crucial. Having long-time user system broken during an upgrade can
only reinforce the feeling that "Python Packaging is broken" and the wrong
impression that "Python 3 zealots are introducing incompatibilities only to
disrupt the Python 2 ecosystem"


2. The gotchas and old solutions
--------------------------------

We'll dive into various solutions that could be or have to be used (as of mid
2016) to release a python 3 only package. While it is possible to tag wheels as
being Python 3 only, a Python 2 installation with pip `<9.0` will consider the
latest `.tar.gz` of a package are the "most recent" and compatible with Python
2, thus breaking user system after upgrade, – at best – on install, at worst
during runtime.

Several workaround are possible, from deploying a meta-package with conditional
requirement, uploading wheel only packages or relaying on
little-known-and-not-really-a-feature of oldest pip versions.


It's easier now !
    - Use setuptools > 24.3, this allows you to set the `python_requires` medatada.
    - Add a `python_requires` to your `setup.py`
    - Use (and have your users use pip 9.0+ it understand `python_requires`

Of course there will always be case where one of the above requirement will not
be true on user machine.

Regardless of the failures it is _critical_ to provide users with the right
error messages _and solution_. Providing _early_
warnings to regular library users that things are about to change is helpful to
reduce the dissatisfaction from users and flow of bug report to your dev team
post-transition.


3. Under the Hood – Updating the Python Packaging stack
-------------------------------------------------------

While Pep 5xx describe a mechanism by which a package can be tagged as Python 3
only this currently does not work for various reasons. Indeed for a Python 3 only
package to be installed 2 critical pieces of software need to understand this metadata:


A) The package manger

For a package to install only on compatible python versions, the package need
to have access to the compatibility information. Beyond some (previously
mentioned "features", and alternate package manager (yum, apt, conda), pip <9.0
does not understand the `requires_python` metadata. It is _prefereable_ for the
Package manager to get this information _before_ downloading and (trying to)
install packages. Understanding how pip does get this information from the
package repository and how pip makes its decision is useful for any team with
internals deployments, and to be able to understand errors users can encounter.

Pip does parse what is called a `simple repository` format, lo list the
available files  for a given package. Information about the package are
extracted from its _name_, and since pip 9.0 from the `data-`attributes
provided in the HTML. This now allow pip to not even consider downloading a
file and trying to install it is the current Python version is not compatible.


B) The Package index.

While the package index (aka PyPI), does store some of this informations, is
quite not what is needed. Plus there was no way to query this information, nor
to expose it for consumption for pip.

The current Package distribution infrastructure is a (complex) beast, it is
interesting to dive into it, see the current status, and what changes can come
in a near, or further future.

PyPI legacy and warehouse are both sharing a database. Any changes is tricky as
PyPI legacy has close to no tests at all. Fetching the `require_python` set on
the `release` table of PyPI (using a Join) was a bottle neck, so we had to
change the table, though PyPI-legacy was doing unexpected operations on the
database... Let's have a look on how the package information is stored and see
what we did.

Despite many legends about the current state of python packaging, contributing
to such infrastructure is less scary than usually depicted, and should be more
friendly with recent changes.


Outline
=======

  1. Intro (4min):

    - Who am I/Who are we
    - History of Python releases/ End of life.
    - The Python 3 statement.
        - Some library will migrate to Python 3 only by 2020.
        - We don't want it to be a hassle for Python 2 users.
        - We might want to have a Python 2 LTS.
    - We <3 users of Python <3

  2. The Problems, and solutions (12 min)

    - How to upgrade a library to Python 3-only ? Current (2016) state:
      - no way to tag the requires_python metadata.
      - Release wheel only: Downstream distributor are sad
      - multiple Tar.Gz a removed "Feature".
      - Fake Meta package: can also break update.
    - Use new setuptools
      - you can now set the requires_python. How does it work ?
      - Still users might not have the right setuptools.
        They will install a non-workign version.
      - Fail early at runtime. With a clear error message.
    - PIP < 9.0  does not understood this medatata item.
      - Fail early at setup.

  3. Under the hood (7min)

    - The `requires_python` metadata
        - Original pep
        - new Pep
        - pypi `/simple/` repository URL
    - PIP:
        - Read info from the `simple-repository`
    - Warehouse vs PyPI legacy
        - Warehouse 100% coverage, docker based, easy to contribute.
        - PyPI legacy. Here be dragon. (much less dragons now)
        - Share a single DB. Bottle neck that needed to refactor for speed.
    - You too can contribute to Python packaging now

  4. TL;DL (too long didn't listen), AKA conclusion.

    - use PIP 9
    - upgrade setuptools
    - Question and Contribution to gotchas, read and contribute to
      python3statement practicalities section.





Ideas,

why you would like (or not) make the transition
Warning pep 440 !
Does not apply to Python 3 only _new_ libraries.

Who are the affected people:

 - Everyone ! As long as you use more than 2 minor versions of Python in you
   life. We _believe_ the most common use case is user of Python 2 that will
   `pip install (--upgrade?)` a library which goes from "single-source" to only
   Py3.



 -
