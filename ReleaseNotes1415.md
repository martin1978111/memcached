# Memcached 1.4.15 Release Notes #

Date: 2012-9-3

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz


## Overview ##

This is a somewhat experimental release which pushes thread performance even
more than before. Since this **is** a more experimental release than usual, and
contains no other major fixes or features, we urge some caution for important
deployments. We feel as though it is high quality software, but please take
caution and do slow rollouts or testing. Thanks!

## Fixes ##

  * Add some mild thread documentation
  * README.md was missing from dist tarball
  * [Issue 286](https://code.google.com/p/memcached/issues/detail?id=286): --disable-coverage drops "-pthread" option
  * Reduce odds of getting OOM errors in some odd cases

## New Features ##

Thread scalability is much improved for reads, and somewhat improved for
writes. In a pure read-only situation on a dual socket six core NUMA machine
I've tested key fetch rates around 13.6 million keys per second.

More tuning is necessary and you'd get significant lag at that rate, but that
shows the theoretical limit of the locks.

## Contributors ##

The following people contributed to this release since 1.4.14.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.14..1.4.15` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.15

```
     6	dormando
     1	Trond Norbye
```

## Control ##