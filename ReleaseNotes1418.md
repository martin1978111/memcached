# Memcached 1.4.18 Release Notes #

Date: 2014-4-17

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.18.tar.gz


## Overview ##


## Fixes ##

  * fix LRU contention for first minute of uptime
    * This made some synthetic benchmarks look awful.
  * Make hash table algorithm selectable
  * Don't lose item\_size\_max units in command line
  * Add a "stats conns" command to show the states of open connections.
  * Allow caller-specific error text in binary protocol
  * Stop returning ASCII error messages to binary clients
  * Fix reference leak in binary protocol "get" and "touch" handlers
  * Fix reference leak in process\_get\_command()


## New Features ##

### Stats conns ###

New "stats conns" command, which will show you what currently open connections are up to, how idle they've been, etc.

### Starttime Hash Algorithm Selection ###

The jenkins hash was getting a little long in the tooth, and we might want to
add specific hash algorithms for different platforms in the future. This makes
it selectable in some sense. We've initially added murmur3 hash to the lineup
and that seems to run a tiny bit faster in some tests.

`-o hash_algorithm=murmur3`

### LRU Crawler ###

A new background thread emerges! Currently experimental, so the syntax might
change. If you run into bugs please let us know (though it's been testing fine
in torture tests so far).

If you wish to clean your slab classes of items which have been expired,
either one-time or periodically, this will do it with low impact as a
background operation.

Currently it requires kicking off a crawl via manual command:

First, enable the thread:
`lru_crawler enable`
or use `-o lru_crawler` as a starttime option.

`lru_crawler crawl 1,3,5`

... would crawl slab classes 1,3,5 looking for expired items to add to the
freelist.

This is generally not useful or required, unless you have memory with very
mixed TTLs, you do not fetch items frequently enough or otherwise cause them
to expire, and you don't want items with longer TTLs block reclaiming expired
items, or to be evicted early.

Future uses of the thread should allow examining and purging items via a
plugin interface: IE crawl all items matching some string and remove them, or
count them. It is simple to modify to experiment with as of now.

See doc/protocol.txt for full explanation of related commands and counters.

## Contributors ##

The following people contributed to this release since 1.4.17.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.17..1.4.18` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.18

```
    21	dormando
     8	Steven Grimm
     1	Andrew Glinskiy

```

## Control ##