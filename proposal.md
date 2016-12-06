# PyCon Proposal


:Title: Lobbing the Holy Hand Grenade: Fixing Python packaging to nullify the
dangerous rabbit-den that lurks within the sunset of Python 2 support

Alt :  Ending Py2/py3 compatibility in a user friendly manner /// No enough mention of py2/py3

:Duration: 30/45 minutes
:Level: Intermediate
:Categories: General, Python 2, Python 3, Packaging, Best Practices.

:Authors: Matthias Bussonnier, Michael Pacer, Thomas Kluyver, MinRK


Summary
=======


> "Four shalt thou not count, neither count thou two, excepting that thou
then proceed to three."

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
that is to allow easy installation of older versions; another way is to have a
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
autonomous collective, our journeys alongside the knights of setuptools, our
efforts in building the ramparts of the pypa/Warehouse Castle, battles with the
dragons of Pip, and errands in the "land of no unit tests" otherwise known as
PyPI legacy.

By the end of the above tale, the careful beholder will be aware of the
hazards of migrating their libraries to require Python 3, and of strategies for
overcoming these issues.


Description
===========

This talk will have 3 main parts:

In the first part, we will describe our quest, focusing on movements toward
Python 3 and the concerns that arise for library developers who wish to end
support for Python 2.

In the second part, we will describe several ways to approach the transition
itself, complete with a discussion of the merits and demerits of each approach.
In particular, we will end with a focus on our new solution that avoids many of
the pitfalls of earlier solutions.

In the third part, we describe the work needed in order to make to implement
the newest solution and how it avoids unleashing rabbits on unsuspecting Python
2 users.

Audience
========

Library authors, in particular library authors interested in releasing a
version of their library that requires Python 3 without breaking Python 2 users'
systems.

Users and developers of Python 2 libraries, who want to make sure their systems
don't upgrade to incompatible versions of once-compatible packages.

User and developers of Python 2 and 3 libraries who wish to transition to a
Python 3 only codebase, but who want to ensure a path for Python 2 users to
continue to use older versions of the software or "Long Term Support" versions
of the software.

User and developers of Python 3 libraries who care about Python 2 users and
developers.

Objective
=========

 - Make users and developers aware of the [Python 3
   statement](http://www.python3statement.org/) as a resource to inform users
   and developers that some packages are migrating to require Python 3, and how
   to do it without breaking users' systems.

 - Make users and community leaders aware that they should be using pip 9+ and
   setuptools 24.3+ to avoid problems in their Python 2 installation.

 - Make developers aware of the recent changes in Python Packaging
   (`python_requres` metadata) and how to make use of it.

 - The traps to avoid if you plan to release a Python 3 only version of your
   package.


Detailed Abstract
=================

1. Intro The Python 3 statement and ipythoN
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
2016) to release a Python 3 only package. While it is possible to tag wheels as
being Python 3 only, a Python 2 installation with pip `<9.0` will consider the
latest `.tar.gz` of a package as the "most recent version". It will treat it as
compatible with Python 2, thereby breaking users' systems after they upgrade
(without any warning being given). Worse, if even one  dependency upgrades in
this manner, that will still be enough to break users' systems.

Several workarounds are possible. This includes (META: we should give examples
for each)

Do Nothing
~~~~~~~~~~

Do nothing. Just release your new package that uses Python 3 only feature.

It's super easy. You just need to release.

The drawbacks is that some users (dependees' maintainers) will chase you to the
end of the world with a chainsaw because you broke their system. You likely
don't want that.

Example: Nikolas

Change your package name
~~~~~~~~~~~~~~~~~~~~~~~~

Of course instead of releasing a new version you can decide to make a full new
package with a new name.


Then it's obvious from the package name on which system your users are working.

Though it needs all user to become _aware_ of the new name, and migrate to it
_explicitely_. It will also invalidate most of the user habits and already
available documentation available online.

Example: ???

Wheel only
~~~~~~~~~~

Releasing a package as wheel only can allow you to release for Python 3 only.

For pure python packages it is relatively easy to do.

Many system (and downstream distributors) do though rely – or prefer – a
source-distribution. It is not possible to release wheels for all packages.
Wheels also make a strict Python 2 vs Python 3 dichotomy so do not allow you to
express dependency on minor python revisions.

Example: flit

Metapackage
~~~~~~~~~~~

It is possible to release a meta-package that has _virtually_ no code and rely
on conditional dependency to install its actual core code on the user system.
For example, Frob-6.0 could be a meta-package which depends on
Frob-real-py2 on Python <3.0, and Frob-real-py3 on Python >= 3.4. While
this approach is _doable_ this can make import confusing.

Using a metapackage has the advantage of not changing the package name, though
it requires to publish a second package on PyPI, potentially with a confusing
name. For example : Frob-6.0 (metapackage), FrobForPy3-60 (dependency). This is
annoying from the developers side, who have to maintain 2 packages, and from
user perspective, errors might come from FrobFromPy3.

Moreover, upgrading your package may need the user to explicitly tell pip to
upgrade dependencies as `pip install -U frob` would only upgrade the
meta-package.

Example: None to our knowledge, but we considered it for IPython.


Release multiple Sdist
~~~~~~~~~~~~~~~~~~~~~~

A little know feature of pip, is that if your sdist name ends in `py-X.y`, then
this sdist will only be installed on Python X.y.

Thus you can target only a subset of Python minor version by publishing
multiple sdist. So you _can_ even release only every-other release of Python.

In the other hand, you _have_ to publish N source-dist, including potential
future version of Python. Is the version of Python post 3.9 be 3.10 or 4 ?

Also PyPI does not allow you to upload multiple sdist. So this one is out
anyway !

Example: (??? Ask Donald)


The new way
~~~~~~~~~~~

It's easier now!

If you use setuptools > 24.2, you can set the `python_requires` metadata in you
`setup.py`. With the recent changes to PyPI/warehouse and when your users are
using PIP 9.0+ it will only install compatible version of your software.

You don't need to change name, you don't need to jump through hoops.
It's minimally a single line change to your source.

Though it won't work for your users not on pip 9.0+, and sometime if they don't
have setuptools 24.2.

There will always be cases where one of the above requirements will not be true
on some users' machines, particularly if they do not upgrade their pip.  But,
this is why it is even more important to encourage Python 2 users to upgrade
their pip version to 9.0+.

Regardless of the failures it is _critical_ to provide users with the right
error messages and _solutions_ to the problems that arise. Providing _early_
warnings to regular library users that things are soon to change helps reduce
user surprise and dissatisfaction, which should also stem the flow of the
inevitable compatibility related bug reports.


3. Under the Hood – Updating the Python Packaging stack
-------------------------------------------------------

While Pep 5xx describes a mechanism by which a package can be tagged as Python
3 only, this currently does not work for a variety of reasons. Indeed, for a
Python 3 only package to be installed, 2 critical pieces of software need to
understand this metadata:


A) The package manger

For a package to install only on compatible python versions, the package
manager needs to access the compatibility information. Excepting some
previously mentioned "pseudo-features", pip <9.0 cannot understand the
`requires_python` metadata. It is _preferable_ for the Package manager to get
this information _before_ downloading or (trying to) install packages. It is
useful to understand how pip gets this information from the package repository
and how pip makes its decision. Any team with internal deployments will need
this understanding to diagnose the origin of the errors not-up-to-date users
will encounter.

Pip does parse what is called a `simple repository` format, lo list the
available files  for a given package. Information about the package are
extracted from its _name_, and since pip 9.0 from the `data-`attributes
provided in the HTML. This now allows pip to not bother with downloading a file
(let alone trying to install) when the current Python version is not
compatible.


B) The Package index.

While the package index (aka PyPI), stores some of this information, it lacked
what was needed. If it had been stored, there still was no way to query this
information, nor to expose it to pip.

(META: what is this paragraph accomplishing beyond the rest of the content?)
The current Package distribution infrastructure is a (complex) beast, it is
interesting to dive into it, see the current status, and what changes can come
in a near, or further future. Additionally, by seeing what changes cannot be
made given the current environment, we can learn about the inner workings of
some of the most important parts of the python ecosystem.

We'll have a look on how the package information is stored and see what we did.
PyPI legacy and warehouse share a database. Any change is tricky as PyPI legacy
has close to no tests at all, despite its crucial place in the Python
ecosystem. In order to make the solution available in a future facing manner,
we modified the common database to make it source the correct information.  At
first, we attempted to directly set the `require_python` info on the `release`
table of PyPI by using a `JOIN` operation, which unfortunately turned out to be
a processing bottle neck. Instead, we had to altered the tables directly (using
TRIGGERS).  This would have been sufficient, but PyPI-legacy was operating in
unexpected ways on the database…

Despite many legends about the current state of python packaging, contributing
to this infrastructure is less scary than usually depicted. Recent improvements
documentation continues to improve contributor friendliness.

META: Changes to the PEP.

Outline
=======

  1. Intro (4min):

    - Who am I(the presenter) and Who are we(the authors)
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
        - PyPI legacy. Here be dragons. (many fewer dragons now)
        - Share a single DB. Bottle neck that needed to refactor for speed.
    - You too can contribute to Python packaging now

  4. TL;DL (too long didn't listen), AKA conclusion. (2min)

    - use pip 9+
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
