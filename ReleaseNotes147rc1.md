# Memcached 1.4.7-rc1 Release Notes #

Date: 2011-08-10

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.7_rc1.tar.gz


## Overview ##

This is a maintenance release with many small bugfixes. Now (mostly) immune
from time travelers.

## Fixes ##

  * Use a monotonically increasing timer
  * Immediately expire items when given a negative expiration time
  * fix memcached-tool to print about all slabs
  * Properly daemonize memcached for debian
  * Don't permanently close UDP listeners on error
  * Allow memcached-init to start multiple instances (not recommended)
  * [Issue 214](https://code.google.com/p/memcached/issues/detail?id=214): Search for network libraries before searching for libevent
  * [Issue 213](https://code.google.com/p/memcached/issues/detail?id=213): Search for clock\_gettime in librt
  * [Issue 115](https://code.google.com/p/memcached/issues/detail?id=115): accont for CAS in item\_size\_ok
  * Fix incredibly slim race for maxconns handler. Should no longer hang ever
  * [Issue 183](https://code.google.com/p/memcached/issues/detail?id=183) - Reclaim items dead by flush\_all
  * [Issue 200](https://code.google.com/p/memcached/issues/detail?id=200): Don't fire dtrace probe as the last thing in a function


## New Features ##

### Montonic Clock ###

This isn't really a feature, but is the main change. If your system has
clock\_gettime with CLOCK\_MONOTONIC support, memcached will attempt to use it.
If your clock does wild adjustments, memcached will do its best to continue to
count forward and not backward.

However, if you use the "expiration is an absolute time" feature, where
specifying an value expiration time as a specific date, it can still break.
You must ensure that memcached is started after your clocks have been
synchronized. This has always been the case, though.

## Contributors ##

The following people contributed to this release since 1.4.6.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.6..1.4.7-rc1` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.7-rc1

```
     9  dormando
     6  Trond Norbye
     1  Clint Byrum
     1  Gordon Franke
```


## Control ##