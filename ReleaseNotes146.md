# Memcached 1.4.6 Release Notes #

Date: 2011-07-15

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.6.tar.gz


## Overview ##

This is a maintenance release with some build fixes, many small bug fixes, and
a few major bug fixes. incr/decr are now actually atomic, and a crash with
hitting the max connection limit while using multiple interfaces has been
fixed.


## Fixes ##

  * Gcc on Solaris sparc wants -R and not -rpath
  * [Issue 121](https://code.google.com/p/memcached/issues/detail?id=121): Set the runtime path when --with-libevent is used
  * Fix autogen failure when unable to find supported command.
  * fix race crash for accepting new connections
  * fix incr/decr race conditions for binary prot
  * fix incr/decr race conditions for ASCII prot
  * Compile fix (-Werror=unused-but-set-variable warnings)
  * Bind each UDP socket to an a single worker thread in multiport env
  * Add support for using multiple ports
  * [Issue 154](https://code.google.com/p/memcached/issues/detail?id=154): pid file out of sync (created before socket binding)
  * [Issue 163](https://code.google.com/p/memcached/issues/detail?id=163): Buggy mem\_requested values
  * Fix cross compilation issues in configure
  * [Issue 140](https://code.google.com/p/memcached/issues/detail?id=140) - Fix age for items stats
  * [Issue 131](https://code.google.com/p/memcached/issues/detail?id=131) - ChangeLog is outdated
  * [Issue 155](https://code.google.com/p/memcached/issues/detail?id=155): bind to multiple interface
  * [Issue 161](https://code.google.com/p/memcached/issues/detail?id=161) incorrect allocation in cache\_create
  * Fix type-punning issues exposed with GCC 4.5.1
  * Simplify stats aggregation code
  * Reverse backward expected/actual params in test
  * [Issue 152](https://code.google.com/p/memcached/issues/detail?id=152): Fix error message from mget
  * Refuse to start if we detect libevent 1.[12](12.md)
  * Fix compilation issue on Solaris 9 wrt isspace() macro - Resolves [issue 111](https://code.google.com/p/memcached/issues/detail?id=111)


## New Features ##

### Multiple port binding ###

You may now specify multiple addresses by listing -l multiple times.

## Contributors ##

The following people contributed to this release since 1.4.5.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.5..1.4.6` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.6

```
    13  Trond Norbye
     6  dormando
     5  Dan McGee
     2  Paul Lindner
     1  Jon Jensen
     1  nirvanazc
```


## Control ##