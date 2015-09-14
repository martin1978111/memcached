# Memcached 1.4.11-rc1 Release Notes #

Date: 2012-1-11

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.11_rc1.tar.gz


## Overview ##

This is a release candidate for 1.4.11. Please help test!

## Fixes ##

  * [bug237](https://code.google.com/p/memcached/issues/detail?id=7): Don't compute incorrect argc for timedrun
  * fix 'age' stat for stats items
  * binary deletes were not ticking stats counters
  * Fix a race condition from 1.4.10 on item\_remove
  * close some idiotic race conditions
  * initial slab automover
  * slab reassignment
  * clean do\_item\_get logic a bit. fix race.
  * clean up the do\_item\_alloc logic
  * shorten lock for item allocation more
  * Fix to build with cyrus sasl 2.1.25


## New Features ##

Slab page reassignment and bug fixes over 1.4.10.

### Bug Fixes ###

There were some race conditions and logic errors introduced in 1.4.10, they
should be rare, but users are strongly encouraged to upgrade.

### Slab Reassign ###

Long running instances of memcached may run into an issue where all available
memory has been assigned to a specific slab class (say items of roughly size
100 bytes). Later the application starts storing more of its data into a
different slab class (items around 200 bytes). Memcached could not use the 100
byte chunks to satisfy the 200 byte requests, and thus you would be able to
store very few 200 byte items.

1.4.11 introduces the ability to reassign slab pages. This is a **beta** feature
and the commands may change for the next few releases, so **please** keep this
in mind. When the commands are finalized they will be noted in the release
notes.

Enable slab reassign on startup:

`$ memcached -o slab_reassign`

Once all memory has been assigned and used by items, you may use a command to
reassign memory.

`$ echo "slabs reassign 1 4" | nc localhost 11211`

That will return an error code indicating success, or a need to retry later.
Success does not mean that the slab was moved, but that a background thread
will attempt to move the memory as quickly as it can.

### Slab Automove ###

While slab reassign is a manual feature, there is also the start of an
automatic memory reassignment algorithm.

`$ memcached -o slab_reassign,slab_automove`

The above enables it on startup, and it may also be enabled or disabled at
runtime:

`$ echo "slabs automove 0" | nc localhost 11211`

The algorithm is slow and conservative. If a slab class is seen as having the
highest eviction count 3 times 10 seconds apart, it will take a page from a
slab class which has had zero evictions in the last 30 seconds and move the
memory.

There are lots of cases where this will not be sufficient, and we invite the
community to help improve upon the algorithm. Included in the source directory
is `scripts/mc_slab_mover`. See perldoc for more information:

`$ perldoc ./scripts/mc_slab_mover`

It implements the same algorithm as built into memcached, and you may modify
it to better suit your needs and improve on the script or port it to other
languages. Please provide patches!

### Slab Reassign Implementation ###

Slab page reassignment requires some tradeoffs:

  * All items larger than 500k (even if they're under 730k) take 1MB of space

  * When memory is reassigned, all items that were in the 1MB page are evicted

  * When slab reassign is enabled, an extra background thread is used

The first item will be improved in later releases, and is avoided if you start
memcached without the -o slab\_reassign option.

### New Stats ###

```
STAT slab_reassign_running 0
STAT slabs_moved 0
```

slab\_reassign\_running indicates if the slab thread is attempting to move a
page. It may need to wait for some memory to free up, so it could take several
seconds.

slabs\_moved is simply a count of how many pages have been successfully moved.

## Contributors ##

The following people contributed to this release since 1.4.10.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.10..1.4.11-rc1` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.11-rc1

```
    15	dormando
     1	Dustin Sallings
     1	Steve Wills
```

## Control ##