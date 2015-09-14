# Memcached 1.4.19 Release Notes #

Date: 2014-5-1

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.19.tar.gz


## Overview ##


## Fixes ##

  * Fix endianness detection during configure.
    * Fixes a performance regression with binary protocol (up to 20%)
  * Fix rare segfault in incr/decr.
  * disable tail\_repair\_time by default.
    * Likely not needed anymore, and can rarely cause bugs.
  * use the right hashpower for the item\_locks table. Small perf improvement.
  * Fix crash for LRU crawler while using lock elision (haswell+ processors)


## New Features ##

See the release notes for 1.4.18 for recent interesting features.

## Contributors ##

The following people contributed to this release since 1.4.18.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.18..1.4.19` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.19

```
     9	dormando
     1	Dagobert Michelsen
     1	Eric McConville

```

## Control ##