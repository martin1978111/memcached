# Memcached 1.4.10 Release Notes #

Date: 2011-11-9

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.10.tar.gz


## Overview ##

This release is focused on thread scalability and performance
improvements. This release should be able to feed data back faster than any
network card can support as of this writing.

## Fixes ##

  * Disable [issue 140](https://code.google.com/p/memcached/issues/detail?id=140)'s test.
  * Push cache\_lock deeper into item\_alloc
  * Use item partitioned lock for as much as possible
  * Remove the depth search from item\_alloc
  * Move hash calls outside of cache\_lock
  * Use spinlocks for main cache lock
  * Remove uncommon branch from asciiprot hot path
  * Allow all tests to run as root


## New Features ##

### Performance ###

For more details, read the commit messages from git. Each change was carefully
researched to not increase memory requirements and to be safe from deadlocks.
Each change was individually tested via mc-crusher
(http://github.com/dormando/mc-crusher) to ensure benefits.

Tested improvements in speed between 3 and 6 worker threads (-t 3
to -t 6). More than -t 6 reduced speed.

In my tests, set was raised from 300k/s to
around 930k/s. Key fetches/sec (multigets) from 1.6 million/s to around
3.7 million/s for a quadcore box. A machine with more cores was able to
pull 6 million keys per second. Incr/Decr performance increased similar
to set performance. Non-bulk tests were limited by the packet rate of
localhost or the network card.

Multiple NUMA nodes reduces performance (but not enough to really
matter). If you want the absolute highest speed, as of this release you can
run one instance per numa node (where n is your core count):

`numactl --cpunodebind=0 memcached -m 4000 -t n `

Older versions of memcached are plenty fast for just about all users. This
changeset is to allow more flexibility in future feature additions, as well as
improve memcached's overall latency on busy systems.

Keep an eye on your hitrate and performance numbers. Please let us know
immediately if you experience any regression from these changes. We have tried
to be as thorough as possible in testing, but you never know.

## Contributors ##

The following people contributed to this release since 1.4.9.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.9..1.4.10` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.10

```
    10	dormando
```

## Control ##