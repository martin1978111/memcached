# Memcached 1.4.14 Release Notes #

Date: 2012-7-30

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.14.tar.gz


## Overview ##


## Fixes ##

  * fix compile issue with new GCC's
  * Added support for automake-1.12 in autogen.sh
  * Use Markdown for README.
  * Fixed issue with invalid binary protocol touch command expiration time (http://code.google.com/p/memcached/issues/detail?id=275)
  * Define touch command probe for DTrace support
  * Error and exit if we don't have hugetlb support (changes -L behavior)
  * update reassign/automove documentation
  * Remove USE\_SYSTEM\_MALLOC define
  * slab rebalancing from random class
  * split slab rebalance and automove threads
  * pre-split slab pages into slab freelists
  * Avoid race condition in test during pid creation by blind retrying


## New Features ##

This release mainly features a number of small bugfixes, but also a change to
slab rebalance behavior.

Previously, if you moved a slab page from one slab to another, you had to wait
until that new page was fully used before moving another one. That wait has
been removed, and you can move pages as fast as the system can ... move them.

A few new features as well:

### slabs reassign ###

`slabs reassign -1 15` will pick a page from any slab class and move it to
class 15.

### slabs automove ###

`slabs automove 2` now enables an ultra aggressive page reassignment
algorithm. On every eviction, it will try to move a slab page into that
class. You should **never** run this in production unless you have a very, very
good idea of what's going to happen. For most people who have spurious
evictions everywhere, you'll end up mass evicting random data and hurting your
hit rate. It can be useful to momentarily enable for emergency situations, or
if you have a data access pattern where evictions should never happen.

This was work we were planning on doing already, but twitter's rewrite has
people presently interested in trying it out. You've been warned.

## Contributors ##

The following people contributed to this release since 1.4.13.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.13..1.4.14` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.14

```
    18	dormando
     1	Clint Byrum
     1	Eric McConville
     1	Fordy
     1	Maksim Zhylinski
     1	Toru Maesaka
     1	yuryur

```

## Control ##