# Memcached 1.4.23 Release Notes #

Date: 2015-4-19

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.23.tar.gz


## Overview ##

Major new release with a complete overhaul of the LRU system. Potentially huge benefits in memory efficiency are possible if the new features are enabled. By default the code should behave similar to how it did in all previous versions, though locking is improved and the new code is still used in some ways.

Please read the feature notes carefully and try it out!

Real world examples have shown huge memory efficiency increases when using items of mixed TTL's (some short, some long). When all items have unlimited TTLs, hit ratios have still improved by several percent.

## Fixes ##

  * spinlocks removed since they never seem to improve performance.
  * flush\_all was not thread safe.
  * better handle items refcounted in tail by unlinking them from the LRU's

## New Features ##

This release is a reworking of memcached's core LRU algorithm.

  * global cache\_lock is gone, LRU's are now independently locked.
  * LRU's are now split between HOT, WARM, and COLD LRU's. New items enter the HOT LRU.
  * LRU updates only happen as items reach the bottom of an LRU. If active in HOT, stay in HOT, if active in WARM, stay in WARM. If active in COLD, move to WARM.
  * HOT/WARM each capped at 32% of memory available for that slab class. COLD is uncapped.
  * Items flow from HOT/WARM into COLD.
  * A background thread exists which shuffles items between/within the LRU's as capacities are reached.

The primary goal is to better protect active items from "scanning". items which are never hit again will flow from HOT, through COLD, and out the bottom. Items occasionally active (reaching COLD, but being hit before eviction), move to WARM. There they can stay relatively protected.

A secondary goal is to improve latency. The LRU locks are no longer used on item reads, only during sets and from the background thread. Also the background thread is likely to find expired items and release them back to the slab class asynchronously, which speeds up new allocations. Further work on the thread should improve this.

There are a number of new statistics to monitor this. Mainly you'll just want to judge your hit ratio before/after, as well as any potential latency issues.

To enable: start with `-o lru_maintainer,lru_crawler`

To adjust percentage of memory reserved for HOT or WARM LRU's (default to 32% each):
`-o lru_maintainer,lru_crawler,hot_lru_pct=32,warm_lru_pct=32`

A recommended start line:
`-o lru_maintainer,lru_crawler,hash_algorithm=murmur3`

An extra option: -o expirezero\_does\_not\_evict (when used with lru\_maintainer) will make items with an expiration time of 0 unevictable. Take caution as this will crowd out memory available for other evictable items.

Some caveats exist:

  * Some possible tunables are currently hardcoded.
  * Max number of slab classes is now 62, instead of 200. The default slab factor gives 42 classes.

This is loosely inspired by the 2Q algorithm. More specifically the OpenBSD variant of it: http://www.tedunangst.com/flak/post/2Q-buffer-cache-algorithm

It's then extended to cope with the fact that memcached items do not behave the same way as a buffer pool. TTL's mean extra scanning/shuffling is done to improve memory efficiency for valid items.

## Contributors ##

The following people contributed to this release since 1.4.22.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.22..1.4.23` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.23

```
    31	dormando

```

## Control ##