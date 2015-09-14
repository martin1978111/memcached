# Memcached 1.4.21 Release Notes #

Date: 2014-10-12

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.21.tar.gz


## Overview ##


## Fixes ##

  * makefile cleanups
  * Avoid OOM errors when locked items stuck in tail

If clients occasionally fetch many items, more than can fit the TCP buffers, then hang for a very long period of time, that slab class could OOM. In older versions this could cause a crash. Since 1.4.20 this will cause OOM errors.

Now, if a locked item lands in the LRU tail, it will be bumped back to the head and an lrutail\_reflocked counter incremented. If you're concerned about having stuck clients, watch that counter.

Big thanks to Jay Grizzard et all at Box for helping track this down!

## New Features ##

None.

## Contributors ##

The following people contributed to this release since 1.4.20.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.20..1.4.21` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.21

```
     4	Steve Wills
     3	dormando
     1	Jay Grizzard

```

## Control ##