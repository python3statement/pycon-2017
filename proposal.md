# PyCon Proposal


:Title: A journey to sunset Python 2 support and Fix Python Packaging
:Duration: 30/45 minutes
:Level: Intermediate
:Categories: 

:Auhors: Matthias Bussonnier, Michael Pacer, Thomas Kluyver, MinRK



Summary
=======


> "Four shalt thou not count, neither shalt thou count two, excepting that thou
then proceedeth to three."

Python 3 has been around for now more than eight years, and most of the Python
ecosystem is now available both on Python 2 and Python 3, often using a single
code base. It comes a time in the lifespan of such libraries when the authors
start to consider stopping support for Python 2 in a new major version.

While the above statement seem to be relatively simple it was (or is depending
on the merge-state of various patches) relatively difficult to achieve without
wreaking havoc for Python 2 user. 

As any maintainer of relatively widely used library, we deeply care about "not
breaking user space", and adventured in the rabbit-hole of Packaging (and of
Caerbannog). 

We'll narrate our tale through the amending on peps, our journeys with the
knights of setuptools, contribution to the Castle of pypa/Warehouse, fights
the dragons of Pip, and errance in the unittest-less land of PyPI legacy. 

At the end off the above tale, the careful beholders will be aware of the
hazards of migrating their libraries to Python 3 only, aware of the pitfalls 


Description
===========

This talk will have 3 main category, 

In the first part we will focus on the hiccups that can arise if one want to
release a new major version of a program/library which focus on being
compatible with Python 3 only, or at least lay the ground work to prepare this
transition. There are many way that the transition can be done, with most of
them having their advantages and drawbacks. In particular most of the solution
often advocated on the internet can break on Python 2 systems increasing the
chasm between Python 2 and Python 3 users comforting the former one that Python
3 was a "bad idea" and isolating the later one in the illusion of "it works on
my machine".

1. The gotchas and old solutions
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
warnings to regular library users tha things are about to change is helpful to
reduce the dissatisfaction from users and flow of bug report to your dev team
post-transition.

While it seam that Python have a way to mark a package as Python 3 only, we'll
see that it is/was not always possible to do so. 

2. Updating the Python Packaging stack
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





Ideas,

why you would like (or not) make the transition
Warning pep 440 !
Does not apply to Python 3 only _new_ libraries.

