# Memcached 1.4.4 Release Notes #

Date: 2009-11-26 Sat

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.4.tar.gz


## Overview ##

This is a maintenance release of memcached with a workaround for
common client issue as well as a few new stats.

## Fixes ##

### Add partial backwards compatibility for delete with timeout 0 ###

Before version 1.4.0, there was an optional argument to delete that
would allow a client to specify that a deleted object should exist in
the cache after the deletion occurred such that add operations would
fail even though objects did not appear in the cache.

This feature was removed completely in 1.4.0, but a parser bug caused
it to slip through.  The bug was fixed in 1.4.3.  If anyone was
attempting to use it legitimately in the 1.4 series, it would simply
not work as expected.

The 1.4.4 backwards compatibility change allows specifically the value
of 0 (i.e. non-lingering delete), while continuing to reject others.
This will satisfy clients that always wish to send a value even when
they do not wish the item to linger.

## New Features ##

### New Stats ###

#### auth\_enabled\_sasl ####

This is a general stat that indicates whether SASL authentication is
enabled or not.

#### auth\_cmds ####

Indicates the total number of authentication attempts.

#### auth\_errors ####

Indicates the number of failed authentication attempts.

## Contributors ##

The following people contributed to this release since 1.4.3.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.3..1.4.4` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.4

```
     2  Dustin Sallings
     2  Matt Ingenthron
     1  dormando
```