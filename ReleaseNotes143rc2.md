# Memcached 1.4.3-rc2 Release Notes #

Date: 2009-11-02 Mon

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.3_rc2.tar.gz

## Overview ##

This is a maintenance release of memcached featuring mostly bug fixes
and one new feature.

### RC history ###

rc2 fixes a multiget bug that showed up in rc1.  A bug was not filed,
but it was found and patched at roughly the same time.

## Fixes ##

### Critical Fixes ###

  * Malicious input can crash server. [bug102](https://code.google.com/p/memcached/issues/detail?id=2)

### Non-critical Fixes ###

  * Removed special case in slab sizing for factor 2. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Provide better errors for deletion scenarios. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Fix get stats accounting. [bug104](https://code.google.com/p/memcached/issues/detail?id=4)
  * Ignore stats prefix for keys without a delimiter. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Work around rpm's broken concept of versions more. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Use slab class growth factor limit. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Added LSB section to init script. [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * Documentation fixes
  * Various build fixes

### Itemized List of Bugs Closed ###

If a bug shows up in this list that wasn't specifically mentioned
above, it's either too minor to mention specifically or the bug was
closed by introducing a test that proves that the bug, as described,
does not exist.

  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=)
  * [bug101](https://code.google.com/p/memcached/issues/detail?id=1)
  * [bug102](https://code.google.com/p/memcached/issues/detail?id=2)
  * [bug104](https://code.google.com/p/memcached/issues/detail?id=4)

## New Features ##

### Support for SASL Authentication ###

Some installations of memcached are not in controlled environments
where simple network filtering keeps bad guys out of your stuff.  To
help with those other environments, we've introduced SASL support.
You can read more about it here:

http://code.google.com/p/memcached/wiki/SASLHowto

### New perl tool `damemtop` in scripts/ ###

dormando's awesome memcached top - a new commandline perl tool for
monitoring small to large memcached clusters. Supports monitoring
arbitrary statistics. See scripts/README.damemtop for more information.

This tool is intended to replace memcached-tool, but not yet.

### Also Noteworthy, Slab Optimizations ###

Objects on the larger end of the limit should be generally more memory
efficient now as more slabs are created (thus are more granular).

## Contributors ##

The following people contributed to this release since 1.4.2.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.2..1.4.3-rc2` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.3-rc2

```
    15  Dustin Sallings
     8  Trond Norbye
     5  dormando
     2  Colin Pitrat
     1  Monty Taylor
     1  Chang Song
     1  CaptTofu
     1  Tomash Brechko
```