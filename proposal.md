# PyCon Proposal


:Title: Lobbing the Holy Hand Grenade: Fixing Python packaging to nullify the
dangerous rabbit-den that lurks within the sunset of Python 2 support

Alt :  Ending Py2/Py3 compatibility in a user friendly manner /// No enough mention of py2/py3
Alt2 : Leaving Python 2 behind without leaving Python 2 users in the dust
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
code base. Nonetheless, this compatibility comes at a development cost and some
library authors are considering ending support for Python 2 in newer versions.
These "once-compatible" libraries pose a real danger for maintaining a smoothly
functioning Python 2 system.

While it may seem simple to cease support for Python 2, the challenge is not in
ending support, but doing so in a way that does not wreak havoc for users who
stay on Python 2. And that is not only a communications problem, but a
technical one.  Why? Put simply, up until recently, it was impossible to tag a
release as Python 3 only; today it is possible.

Like any maintainer of a widely used library, we want to ensure that users
continue to use Python 2 continue to have functioning libraries, even after
development proceeds in a way that does not support Python 2. This can be
addressed by a couple of approaches. 

One approach is to have a parallel "Long Term Support" variant supported by a
different community of developers going forward. This allows the main library
developers to follow their Python 3 only plan, without breaking anything for
Python 2 users, as those users are effectively no longer using the original
package. Especially for those required to use Python 2 because of their
workplace requirements, those workplaces may be able to fund the development of
that "Long Term Support" variant. We do not focus on this route as it is not
sure-proof, since it is unclear whether such a LTS version of the library will
ever exist.  

The more sure-proof approach is to ensure that is to allow easy installation of
older versions.  But, that requires not inappropriately overwriting those older
installed versions when users upgrade, which is a much more "foul, cruel and
bad-tempered rodent" than it at first appears. Users should not need to
manually pin maximal version dependencies across their development environments
and projects if all they want is to use the latest versions of libraries that
are compatible with Python 2. Even if we did expect that of users, consider
what would happen when a package they rely on relies on a package that converts
to be only Python 3 compatible. If they were not tracking the complete
dependency tree, they might upgrade their packages and discover that their
projects no longer work. To avert this they would need to monkey patch the
requirements of the packages they depend on to pin those at the last version
compatible with Python 2. Such a situation is untenable. Users that want to use
Python 2 should not have to go through so much anguish to do so. 

In order to solve this problem, and thereby make both users' and maintainers'
lives easier, we ventured into the rabbit-hole called Packaging.

Though we set off with a singular quest, our tale roves through many lands.
We'll narrate the story of our amending PEPs through discussions with the
autonomous collective, our journeys alongside the knights of setuptools, our
efforts in building the ramparts of the pypa/Warehouse Castle, battles with the
dragons of Pip, and errands in the "land of no unit tests" otherwise known as
PyPI legacy.

By the end of the above tale, the audience members will know the road to Python
3 only libraries had once had hazards that are now easily avoidable. So long as
users upgrade their package management tools. 

Description
===========

This talk will have 3 main parts:

In the first part, we will describe our quest, focusing on libraries moving to
support only toward Python 3. This includes describing the concerns that should
be on the mind of library developers that want end support for Python 2.

In the second part, we will describe several ways to approach the transition
itself, complete with a discussion of the merits and demerits of each approach.
In particular, we will end with a focus on our new solution that avoids many of
the pitfalls of earlier solutions.

In the third part, we describe the work that was needed in order to make to
implement the newest solution and how it avoids unleashing killer rabbits on
unsuspecting Python 2 users.

Audience
========

Library authors. In particular, library authors who are interested in releasing a
version of their library that requires Python 3 but do not wish to break users
systems who continue to use Python 2.

Users and developers of Python 2 libraries, who want to make sure their systems
don't upgrade to incompatible versions of once-compatible packages.

User and developers of Python 2 and 3 libraries who wish to transition to a
Python 3 only codebase, but who want to ensure a path for Python 2 users to
continue to use their software. 

User and developers of Python 3 libraries who want to see more libraries
shifting to Python 3, but who do not want to harm the existing, vibrant Python
2 user and developer communities.

Objective
=========

 - Make users and developers aware of the [Python 3
   statement](http://www.python3statement.org/) as a resource to inform users
   and developers that scientific computing libraries are going to require Python
   3, but that they are trying to do it without breaking users' systems.

 - Make users and community leaders aware of the urgency of encouraging the use
   of pip 9+ and setuptools 24.3+, specifically if they wish to avoid problems
   in their Python 2 installations.

 - Make developers aware of the recent changes in Python Packaging
   (`python_requres` metadata) and how to make use of it.

 - Inform developers of the traps facing the release of a Python 3 only version
   of your package and give tools to avoid those traps.


Detailed Abstract
=================

1. Intro: The Python 3 statement and IPython
--------------------------------------------

A growing number of libraries and library authors have announced that they are
planning on stopping support for Python 2, no later than 2020, if not by the
time of PyCon. It is also known that a number of users will not migrate to 
Python 3 by the time that that change happens.

Currently, these "once-compatible" packages – packages that at one time were
compatible with Python 2 and 3, but are now releasing a Python-3 only version
of the package – create a danger for that libraries extant Python 2 users. In
particular, the challenge lies in ensuring that Python 2 users will install or
upgrade to the most recent Python 2 compatible version of the package. The ease
of upgrading by mistake is too great, and the annoyance is particularly
trenchant as the issue may not manifest until run-time.

Upgrading should be made as seamless as possible, regardless of which Python
version is being used. Breaking user's systems during an upgrade can only
reinforce the feeling that "Python Packaging is broken" and the (inaccurate)
impression that "Python 3 zealots are introducing incompatibilities only to
disrupt the Python 2 ecosystem." In fact, developers who are interested in
moving toward Python 3 care deeply about those who wish to still use Python 2
and are working hard to make it possible for the two ecosystems to live in
harmony.  

2. The gotchas and old solutions
--------------------------------

We'll dive into the various solutions that were available as of mid-2016 for
releasing a Python 3 only package. 


Even then, you could tag wheels as being Python 3 only, but that approach will
break if you also release a `.tar.gz`. Pip `<9.0` considers the latest
`.tar.gz` of a package as the "most recent version", regardless of which Python
version is invoking pip. Then, the new, "once-compatible"  version will be seen
as _still_ compatible with Python 2, thereby breaking users' systems when they
upgrade (and will give no warning about what is happening).  Worse, if even one
dependency upgrades in this manner, that will still be enough to break users'
systems.

Several workarounds were possible then. This includes (META: we should give
examples for each)

Do Nothing
~~~~~~~~~~

Do nothing. Just release your new package that uses some Python 3 only
available on.

It's super easy. You just need to release.

But this approach is not without drawbacks: some users and dependees'
maintainers will chase you to the end of the world with a chainsaw because you
broke their systems. You likely don't want that.

Example: Nikolas

Change your package name
~~~~~~~~~~~~~~~~~~~~~~~~

Instead of releasing a new version, you can release a "new" package with a new
name, that just happens to contain the same functionality as your
"once-compatible" package.

In this case, it's obvious for users which packages are to be used with which
Python systems.

The downside to this is that, all users need to know that the new name exists,
and explicitly migrate to it explicitly everywhere on which they relied on the
previous library. In this action, you invalidate many users' habits, their
code, and much of the documentation that already exists online.

Example: ???

Wheel only
~~~~~~~~~~

Releasing a package only as a wheel can allow you to avoid pip `<9.0`'s Python 3 only problem with `tar.gz`s.

For pure python packages, this is relatively easy to do.

But there are downsides. Many systems and downstream distributors do rely on or
prefer source-distributions. It is not possible to release wheels for all
packages, particularly those that are not pure python packages.  Wheels make a
strict, coarse Python 2 vs Python 3 dichotomy; this stops you from expressing
dependencies on minor python revisions.

Example: flit

Metapackage
~~~~~~~~~~~

It is possible to release a meta-package that has _virtually_ no code and rely
on conditional dependencies that install its actual core code on users' system.
For example, Frob-6.0 could be a meta-package which depends on Frob-real-py2 on
Python <3.0, and Frob-real-py3 on Python >= 3.4. While this approach is
_doable_ this can make for confusing imports.

Using a metapackage has many advantages, or at least lacks the disadvantages of
the other solutions.  For example, you don't change the package name, though it
requires a second package on PyPI, potentially leading name confusion. For
example : Frob-6.0 (metapackage) with FrobForPy3-60 (dependency). 

One of the downsides is that this annoys developers who then have to maintain 2
separate but dependent packages. From a user's perspective, errors might come
from FrobFromPy3 but appear to come from Frob-6.0. Upgrading your package
becomes more complicated, as you need the user to explicitly upgrade
dependencies; `pip install -U frob` would only upgrade the meta-package.

Example: None to our knowledge, but we considered it for IPython.


Release multiple Sdist
~~~~~~~~~~~~~~~~~~~~~~

One little known feature of pip is that if your sdist name ends in `py-X.y`,
then this sdist will only be installed on Python X.y.

This allows targeting only a subset of Python minor versions by publishing
multiple sdist. 

On the other hand, you _have_ to publish N source-dist for you package. And,
this includes potential future version of Python which requires making tough,
unknowable decisions. Is the version of Python post 3.9 be 3.10 or 4 ?

META: ^^ could someone please explain this better; why not just not release an
sdist for that version until it is actually released? why is this a problem?

Also, PyPI does not allow you to upload multiple sdist. So this solution won't
work if you want to be in the cheeseshop anyway!

META: ^^ Does this apply to both legacy and warehouse?

Example: (??? Ask Donald)

The new way
~~~~~~~~~~~

It's easier now!

If you use setuptools > 24.2, you can set the `python_requires` metadata in
your `setup.py`. Recent changes to PyPI/warehouse returns `python_requires`
metadata in response to API calls. Pip 9.0+ knows how to make and interpret
those API calls. This means that if you use setuptools 24.2+ and set
`python_requires` metadata, and your users are using pip 9.0+, then it will
only install versions of your software that are explicitly compatible with
their system, even if your newest release is Python 3 only!

This avoids most of the downsides of other solutions: You don't need to change
package names. You don't need to create a new package. You don't need to
release wheels only. Most importantly, you don't need to abandon your users,
and you don't need to jump through hoops to avoid doing so.  It can be as
simple as a single line change to your source.

There are downsides, but they are of a different type. For example, it won't
work for your users who haven't upgrade to pip 9.0+. Sometime it may not work
if they don't have setuptools 24.2+.

META: ^^ why does it only sometimes not work if they don't have setuptools
24.2, is it if they build from source? Or why else would this be a problem?

This is why it is all the more important to encourage Python 2 users (in
particular) to upgrade their pip version to 9.0+.


META: vv This sounds out of tone for everything else. We don't need to hand
wring, this seems like a general point that we should make before listing
things as it doesn't specifically apply to the new solution

Regardless of the failures it is _critical_ to provide users with the right
error messages and _solutions_ to the problems that arise. Providing _early_
warnings to regular library users that things are soon to change helps reduce
user surprise and dissatisfaction, which should also stem the flow of the
inevitable compatibility related bug reports.


3. Under the Hood – Updating the Python Packaging stack
-------------------------------------------------------

While Pep 5xx describes a mechanism by which a package can be tagged as Python
3 only, this did not work as of mid-2016 for a variety of reasons. For a Python
3 only package to be not installed because of `requires_python` metadata, 2
critical pieces of packaging software need to _understand_ this metadata:

A) The package manger

For a package to install only on compatible python versions, the package
manager needs to access and apply compatibility information. Pip <9.0 does not
understand the `requires_python` metadata. If it did, it is still _preferable_
for the Package manager to get this information _before_ downloading or trying
to install packages. It is useful to understand how pip gets this information
from the package repository and how pip decides what to install. For those with
internal deployments: it is crucial that you understand these inner workings so
you can diagnose the origin of the errors that users who are not up-to-date
will encounter.

META: vv Which version of pip parses that? All versions? If so why is 9.0+ special? 

Pip parses what is called a `simple repository` format, which lists the
available files for a given package. Information about the package is extracted
from its _name_, and (since pip 9.0) from the `data-`attributes provided in the
HTML. This allows pip to ignore package versions incompatible with the current
Python version without even needing to download it to test for compatibility.


B) The Package index.

META: vv which parts were lacking? 

While the package index (aka PyPI), stores some of this information, it lacked
some parts of what was needed. Even if it _had_ been stored, there still was no
way to query this information, nor to expose it to pip.

META: vv what is this paragraph accomplishing beyond the rest of the content?

The current Package distribution infrastructure is a (complex) beast, it is
interesting to dive into it, see the current status, and what changes can come
in a near, or further future. Additionally, by seeing what changes cannot be
made given the current environment, we can learn about the inner workings of
some of the most important parts of the python ecosystem.

We'll discuss how package information is stored and how we changed it.  PyPI
legacy and warehouse share a database.  Any change is tricky as PyPI legacy has
close to no tests at all, despite its crucial place in the Python ecosystem. 

META: vv what do you mean by a future facing manner? What does it mean to
"source the correct information"; Do you mean setuptools 24.2? If so, say that. 

In order to make the solution available in a future facing manner, we modified
the common database to ensure it houses and maintains the correct information.  

META: vv Can someone take a stab at turning this into a more narrative bit?
E.g., why was it a processing bottle-neck? Why was there processing at all,
it's a static database, right? 

At first, we attempted to directly set the `require_python` info on the
`release` table of PyPI by using a `JOIN` operation, which unfortunately turned
out to be a processing bottle neck. 

Instead, we had to alter tables directly using TRIGGERS when new packages were
on either using either PyPI or warehouse mechanisms.  

META: vv Is that still a necessary line?

This would have been
sufficient, but PyPI-legacy was operating in unexpected ways on the database…

META: vv what does this section give us?

Despite many legends about the current state of python packaging, contributing
to this infrastructure is less scary than usually depicted. Recent improvements
documentation continues to improve contributor friendliness.

META: detail more clearly the changes to the PEP that were needed.

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
