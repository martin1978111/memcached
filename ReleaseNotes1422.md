# Memcached 1.4.22 Release Notes #

Date: 2014-12-31

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.22.tar.gz


## Overview ##

Bugfix maintenance release. Fixes to hash table expansion now completely hang all threads very briefly while the hash table pointers are swapped. Once swapped, it unlocks and uses locks as normal.

In previous versions, the hash table was switched to a global lock instead of a map of bucket-locks during expansion. This should be faster overall with a small latency penalty. It's possible to presize the hash table with `-o hashpower`

## Fixes ##

  * gatkq: return key in response
  * Handle SIGTERM the same as SIGINT
  * Fix off-by-one causing segfault in lru\_crawler
  * Fix potential corruption for incr/decr of 0b items
  * Fix [issue #369](https://code.google.com/p/memcached/issues/detail?id=#369) - uninitialized stats\_lock
  * [issue#370](https://code.google.com/p/memcached/issues/detail?id=#370): slab re-balance is not thread-safe in function do\_item\_get
  * Fix potential corruption in hash table expansion
  * use item lock instead of global lock when hash expanding
  * fix hang when lru crawler started from commandline
  * rename thread\_init to avoid runtime failure on AIX
  * Support -V (version option)


## New Features ##


## Contributors ##

The following people contributed to this release since 1.4.21.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.21..1.4.22` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.22

```
     6	dormando
     2	Jason CHAN
     1	Dan McGee
     1	Menghan
     1	Mike Dillon
     1	Oskari Saarenmaa
     1	clark.kang
     1	mckelvin

```

## Control ##