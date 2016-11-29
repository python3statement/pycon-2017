# PyCon Proposal


:Title: A journey to sunset Python 2 support and Fix Python Packaging
:Duration: 30/45 minutes
:Level: Intermediate
:Categories: General, Python 2, Python 3, Packaging, Best Practices.

:Authors: Matthias Bussonnier, Michael Pacer, Thomas Kluyver, MinRK





Summary
=======


> "Four shalt thou not count, neither shalt thou count two, excepting that thou
then proceedeth to three."

Python 3 has been around for now more than eight years, and much of the Python
ecosystem is now available both on Python 2 and Python 3, often using a single
code base. There comes a time in the life of such libraries when the authors
start to consider stopping support for Python 2 in a new major version.

While the above statement seem to be relatively simple, it was (or is, depending
on the merge-state of various patches) relatively difficult to achieve without
wreaking havoc for Python 2 users.

Like any maintainer of a widely used library, we care deeply about "not
breaking user space", and thus we ventured into the rabbit-hole of Packaging
(and of Caerbannog). 

We'll narrate our tale through the amending of PEPs, our journeys with the
knights of setuptools, contribution to the Castle of pypa/Warehouse, fights with
the dragons of Pip, and errands in the unittest-less land of PyPI legacy. 

By the end of the above tale, the careful beholder will be aware of the
hazards of migrating their libraries to require Python 3, and of strategies for
overcoming these issues.


Description
===========

This talk will have 3 main categories:

In the first part we will focus on the hiccups that can arise if one wants to
release a new major version of a program/library which focus on being
compatible with Python 3 only, or at least lay the ground work to prepare this
transition. There are many ways that the transition can be done, with advantages
and drawbacks to each. In particular, most of the solution
often advocated on the internet can break on Python 2 systems, increasing the
chasm between Python 2 and Python 3.

Audience
========

Any library authors, but in particular library authors interested in releasing
a version of their library to require Python 3 without breaking Python 2 users'
systems.

Users and developers of Python 2 libraries, who want to make sure their system
don't upgrade to incompatible package version.

User and developers of Python 3 libraries who care about Python 2 users and
developers.


Objective
=========

 - Make users and developers aware of the [Python 3 statement](http://www.python3statement.org/)
   as a resource to inform
   users and developers that some packages are migrating to require Python 3, and
   how to do it without breaking users' systems.

 - Make users aware that they should be using pip 9+ and setuptools 24.3+ to avoid
   problems in their Python 2 installation.

 - Make developers aware of the recent changes in Python Packaging
   (`python_requres` metadata) and how to make use of it. 

 - The traps not to fall into if you plan to release a Python 3 only package
   version.


Detailed Abstract
=================

1. The Python 3 statement
-------------------------

A growing number of libraries and library authors have said that they are
planning on stopping support for Python 2, no later than 2020, if not already.
It is also known that a number of users will not migrate their Python 3 by that
day.

Currently releasing a new version of a Python-2 and Python-3 compatible library
will break on Python-2 systems, 


2. The gotchas and old solutions
--------------------------------

We'll dive into various solutions that could be or have to be used (as of mid
2016) to release a python 3 only package. While it is possible to tag wheels as
being Python 3 only, a Python 2 install with pip `<9.0` will consider the
latest `.tar.gz` of a package are the "most recent" and compatible with Python
2, thus breaking user system, – at best – on install, at worst during runtime. 

Several workaround are possible, from deploying a meta-package with conditional
requirement, uploading wheel only packages or relaying on
little-know-and-not-really-a-a-feature of oldest pip versions.

Regardless of the failures it is _critical_ to provide users with the right
error messages _and solution_ if ever they were encountered. Providing _early_
warnings to regular library users that things are about to change is helpful to
reduce the dissatisfaction from users and flow of bug report to your dev team
post-transition.

While it seam that Python have a way to mark a package as Python 3 only, we'll
see that it is/was not always possible to do so. 

3. Updating the Python Packaging stack
--------------------------------------

While Pep 5xx describe a mechanism by which a package can be tagged as Python 3
only this currently does not work for various reasons. Indead for a Python 3 only
package to be installed 3 critical pieces of software need to understand this metadata:


A) The package manger

For a package to install only on compatible python versions, the package need
to have access to the compatibility information. Beyond some (previously
mentioned "feaures", and alternate package manager (yum, apt, conda), pip <9.0
does not understand the `requires_python` metadata. It is _prefereable_ for the
Package manager to get this information _before_ downloading and (trying to)
install packages. Understanding how pip does get this information from the
package repository and how pip makes its decision is useful for any team with
internals deployments, and to be able to understand errors users can encounter.



B) The Package index.

While the package index (aka PyPI), does store some of this informations, is
quite not what is needed. Plus there was not way to query this information, not
to expose it for consumption for pip. 

The current Package distribution infrastructure is a (complex) beast, it is
interesting to dive into it, see the current status, and what changes can come
in a near, or further future.

Despite many legends about the current state of python packaging, contributing
to such infrastructure is less scary than usually depicted, and should be more
friendly with recent redactors. 


C) The Package builder.

Aka Setuptools, until recently had no way to set the `requires_python`
metadata. Alternative package managers like `flit` were able to do so, but are
wheel-only. It is now 



Outline
=======

  1. Intro (5min):
    - Who am I/Who are we
    - History of Python releases/ End of life. 
    - The Python 3 statement.



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
