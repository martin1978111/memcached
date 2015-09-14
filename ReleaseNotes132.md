# Memcached 1.3 Beta 2 Release Notes #

Date: 2009-03-11 Wed


## New Features ##

### Binary Protocol ###

A new feature that brings new features.  We now have goodness like
CAS-everywhere (e.g. delete), silent, but verifiable mutation
commands, and many other wonders.

Note that the original protocol is **not** deprecated.  It will be
supported indefinitely, although some new features may only be
available in the binary protocol.

#### Client Availability ####

Many clients for the binary protocol are available.

**C**

> libmemcached supports just about anything you can do with a memcached
> protocol and is the foundation for many clients in many different
> languages (which you can find linked from the project page).

> Project page:  http://tangent.org/552/libmemcached.html

**Java**

> spymemcached has very good text and binary protocol support over IPv4
> and IPv6 with a quite comprehensive test suite.

> Project page:  http://code.google.com/p/spymemcached/

**Protocol Spec**

> NIH problem?  Go write your own client.  :)

> http://cloud.github.com/downloads/dustin/memcached/protocol-binary.txt


## Performance ##

Lots of effort has gone into increasing performance.

There is no longer a build-time distinction between a single-threaded
and multi-threaded memcached.  If you want a single-threaded
memcached, ask for one thread (though there'll still be utility
threads and other such things in the background).  This change lets us
focus on a future where multiple cores can be saturated for servicing
requests.

Facebook-inspired contention reduction with per-thread stat collection
and the Facebook connection dispatch and thread starvation prevention
contributions helped our scalability.

Lock analysis also showed us that we had quite a bit of contention on
hash table expansion which has been moved into its own thread, greatly
improving the scalability on multicore hardware.

A variety of smaller things also shook out of performance testing and
analysis.

There's also a memory optimization for users who don't actually make
use of CAS.  Running memcached with -C disables the use of CAS
resulting in a savings of about eight bytes per item.  If you have big
caches, and don't use CAS, this can lead to a considerable savings.

## Stats ##

There are several new stats and some new ways to look at older stats.

### New Stats ###

**Delete**

> The global stats now contain statistics on deletion.

> delete\_hits refers to the number of times a deletion command was
> issued which resulted in a modification of the cache while
> delete\_misses refers to the number of times a deletion command was
> issued which had no effect due to a key mismatch.

**Incr/Decr**

> Incr and decr each have a pair of stats showing when a
> successful/unsuccessful incr occurred.  incr\_hits, incr\_misses,
> decr\_hits, and decr\_misses show where such mutations worked and where
> they failed to find an existing object to mutate.

**CAS**

> CAS stats are tracked in three different ways:

> + cas\_hits

> Number of attempts to CAS in a new value that worked.

> + cas\_misses

> Number of attempts to CAS in a value where the key was not found.

> + cas\_badval

> Number of attempts to CAS in a value where the CAS failed due to the
> object changing between the gets and the update.

**slab class evicted time**

> Per slab class, you can now see how recently accessed the most recent
> evicted data was.  This is a useful gauge to determine eviction
> velocity on a slab so you can know whether evictions are healthy or if
> you've got a problem.


### More Granular Stats ###

Where possible, stats are now tracked individually by slab class.  The
following stats are available on a per-slab-class basis (via "stats slabs"):

  * get\_hits
  * cmd\_set
  * delete\_hits
  * incr\_hits
  * decr\_hits
  * cas\_hits
  * cas\_badval

(misses are obviously not available as they refer to a non-existent item)

### Removed stats ###

"stats malloc" and "stats maps" have been removed.

If you depended on these commands for anything, please let us know so
we can bring them back in a more maintainable way.

## Bug Fixes ##

  * Build fixes on ubuntu (gcc and icc) and FreeBSD
  * bad interaction with cas + incr ([bug 15](https://code.google.com/p/memcached/issues/detail?id=5))
  * setuid failures are reported properly at daemonization time
  * decr overflow causing unnecessary truncation to 0 ([bug 21](https://code.google.com/p/memcached/issues/detail?id=1))
  * failure to bind on Linux with no network (i.e. laptop dev)
  * some memcached-tool cleanup

## Development Info ##

We've added a bunch of tests and new code coverage reports.

All included code in this release has been tested against the
following platforms (using the in-tree test suite):

  * ubuntu 8.10 (64-bit, both gcc and icc)
  * ubuntu 8.04 (32-bit)
  * OS X 10.5 (ppc and intel)
  * OpenSolaris 5.11 x86 (with and without dtrace)
  * FreeBSD 7 x86

## Feedback ##

Please try this version.  Make it suffer.  Report feedback to the list
or file bugs as you find them.

  * Mailing List:  http://groups.google.com/group/memcached
  * Issue Tracker:  http://code.google.com/p/memcached/issues/list
  * IRC:  #memcached on freenode

## Contributors ##

The following people contributed to this release since 1.2.6.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.2.6..1.3.2` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/dustin/memcached/commits/1.3.2

```
  104  Dustin Sallings
   49  Trond Norbye
   32  Toru Maesaka
   31  dormando
    8  Steve Yen
    7  hachi
    6  Aaron Stone
    6  Brian Aker
    4  Victor Kirkebo
    2  Ricky Zhou
    1  Jonathan Bastien-Filiatrault
    1  Evan Klitzke
    1  Eric Lambert
```