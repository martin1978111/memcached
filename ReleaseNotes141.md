# Memcached 1.4.1 Release Notes #

Date: 2009-08-29 Sat

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.1.tar.gz

## Overview ##

This is a maintenance release consisting primarily of bug fixes.

## Fixes ##

### Criticial Fixes ###

  * Boundary condition during pipelined decoding caused crash ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Bad initialization during buffer realloc ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Buffer overrun in stats\_prefix\_find ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Memory corruption from negative and invalid item lengths ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))

### Non-critical Fixes ###

  * Update flush stats for binary flushes ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Build fixes for OpenBSD
  * Build fixes for Solaris/gcc
  * Cleanup warnings brought to us by OpenBSD (sprintf, etc...)
  * Lots of fixes with the test tools
  * Various documentation cleanups
  * RPM spec autoupdate

## New Features ##

  * stats slabs returns the number of requested bytes as mem\_requested
  * memcached can bind to ephemeral ports (used for testing)

## Contributors ##

The following people contributed to this release since 1.4.0.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.0..1.4.1` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.1

```
    18  Dustin Sallings
     8  Trond Norbye
     2  dormando
     1  Mat Hostetter
     1  Adam Thomason
     1  Monty Taylor
     1  Steve Yen
     1  Matt Ingenthron
     1  Cosimo Streppone
     1  James Cohen
```