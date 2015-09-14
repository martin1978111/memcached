# Memcached 1.4.12 Release Notes #

Date: 2012-2-1

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.12.tar.gz


## Overview ##

Fix a small number of bugs, mostly in building on different platforms.

For the real meat, see [1.4.11 Release Notes](ReleaseNotes1411.md)

## Fixes ##

  * fix glitch with flush\_all (exptime)
  * Skip SASL tests unless RUN\_SASL\_TESTS is defined.
  * Look around for saslpasswd2 (typically not in the user's path).
  * build fix:  Define sasl\_callback\_ft on older versions of sasl.
  * fix segfault when sending a zero byte command
  * fix warning in UDP test
  * properly detect GCC atomics
  * tests: loop on short binary packet reads
  * fix slabs\_reassign tests on 32bit hosts

## New Features ##

Fewer bugs!

## Contributors ##

The following people contributed to this release since 1.4.11.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.11..1.4.12` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.12

```
     5	Dustin Sallings
     5	dormando
```

## Control ##