# Memcached 1.4.2-rc1 Release Notes #

Date: 2009-10-07 Wed

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.2_rc1.tar.gz

## Overview ##

This is a maintenance release consisting primarily of bug fixes.

## Fixes ##

### Critical Fixes ###

  * Reject keys larger than 250 bytes in the binary protocol ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Bounds checking on stats cachedump ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Binary protocol set+cas wasn't returning a new cas ID ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))

### Non-critical Fixes ###

  * Binary quitq didn't actually close the connection ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Build fix on CentOS 5 ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Slab boundary checking cleanup (bad logic in unreachable code)
  * Removed some internal redundancies.
  * Use the OS's provided htonll/ntohll if present ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Test fixes/cleanup/additions.
  * Get hit memory optimizations ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Disallow -t options that cause the server to not work ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * memcached -vv now shows the final slab
  * Killed off incomplete slab rebalance feature.
  * Better warnings.
  * More consistent verbosity in binary and ascii ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))

## New Features ##

### Support for libhugetlbfs (in Linux) ###

From http://libhugetlbfs.ozlabs.org/ -

> libhugetlbfs is a library which provides easy access to huge
> pages of memory. It is a wrapper for the hugetlbfs file
> system.

If you are running memcached with a very large heap in Linux, this
change will make it available to you.  The hugetlbfs HOWTO provides
detailed information on how to configure your Linux system and provide
advice to applications (such as memcached) to make use of it.

### Support for evictions, evict\_time and OOM counts in memcached-tool ###

memcached-tool is a commandline tool to display information about your
server.  It displays more now.

### Configurable maximum item size ###

Many people have asked for memcached to be able to store items larger
than 1MB, while it's generally recommended that one _not_ do this, it
is now supported on the commandline.

A few enlightened folk have also asked for memcached to reduce the
maximum item size.  That is also an option.

The new -I parameter allows you to specify the maximum item size at
runtime.  It supports a unit postfix to allow for natural expression
of item size.

Examples:

```
memcached -I 128k # Refuse items larger than 128k.
memcached -I 10m  # Allow objects up to 10MB
```

### New stat: 'evicted\_nonzero' ###

The evicted\_nonzero stat is a counter of all of the evictions for
items that had an expiration time greater than zero.

This can be used to help distinguish "healthy" evictions from
"unhealthy" ones.  If all of your evictions are for objects with no
expiration, then they're naturally falling off the LRU as opposed to
being evicted before their maximum expiry that was set at item store
time.

### Protocol definitions for range protocol ###

memcached ships with a binary protocol header that can be used when
implementing your own protocol parsers and generators.  The structure
definitions and opcodes for the range specification are included in
this header.

Note that the server _does not_ support these operations.

## Contributors ##

The following people contributed to this release since 1.4.1.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.1..1.4.2-rc1` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.2-rc1

```
    12  Dustin Sallings
    10  Trond Norbye
     9  dormando
     1  Vladimir
     1  Ryan Tomayko
     1  Mat Hostetter
     1  Dmitry Isaykin
```