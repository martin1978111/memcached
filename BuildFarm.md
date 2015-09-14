# The Build Farm #

Link to the [memcached build farm](http://builds.memcached.org/waterfall?category=memcached) waterfall status page.

Memcached builds are tests against a variety of hosts and configurations via [buildbot](http://buildbot.net/) software before and after every bit of code is accepted.

# Contributing #

If you'd like to contribute, please install the latest version of the following tools available for your system:

## Software Requirements ##

  * git
  * autotools (automake, autoconf)
  * libevent
  * gnu make
  * C compiler
  * buildbot

## Availability Requirements ##

We place no strict uptime requirements on build slaves, but it would be nice if they were generally available.  If your slave goes down, we'll try to get hold of you to get it running again.

## System Impact ##

The build process for memcached is currently fairly lightweight and buildbot's slave overhead is minimal.

Builds may be run most frequently during development cycles after a developer has performed a local test, and will always follow a push to the central repo.

If you are providing two builders for a host (e.g. ± some feature), you may ask for us not to run those concurrently.  For special cases, you may also request that two of your seemingly unrelated code do not run all or part of a build concurrently.

Before a build, the minimal set of git objects required to represent a build tree are retrieved via git and an optional patch **may** be included inline.

After a successful build, the tested memcached binary is gzipped and retrieved from the system.

## Setting It Up ##

Once that's all in place, contact the list and let us know you're ready to contribute.  One of us will work with you on making it part of the official builds.