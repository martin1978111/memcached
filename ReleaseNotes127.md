# Memcached 1.2.7 Release Notes #

Date: 2009-04-03 Fri


## Download ##

Download link:

http://memcached.googlecode.com/files/memcached-1.2.7.tar.gz

## Notes ##

With the release of memcached 1.2.7, the 1.2 tree is now officially in
maintenance mode. Only bugfixes and very minor improvements will be
added to the 1.2 tree. All development is now happening on the 1.3
tree. Efforts are now being made to stabilize the 1.3 tree into a 1.4
series stable release. Please help test :)

1.2.7 appears to be a good, stable release, and is a decent farewell
to the codebase that has helped scale many companies.

-Dormando

## Features ##

  * UDP/TCP can be disabled by setting their port to 0
  * Can set the listen backlog on the commandline (-b)

### Stats ###

Handful of new stats.

#### evicted\_time ####

Under 'stats items', this lists the time since the last evicted object was
last accessed. If an object was evicted a day after it had last been fetched,
you would see 86400 as the time.

#### other stats also noted in 1.3.3 ####

- accepting\_conns
- listen\_disabled\_num
- cmd\_flush

## other improvements also noted in 1.3.3 ##

- missing key debugging.
- tail repair.

### tail repair ###

Tail repair is an important stability fix, and is worth repeating here.

There is a rare, unidentified reference leak that causes a slab to be
full of invalid objects that cannot be evicted via the LRU nor will
they expire on their own.

Tail repair is a strategy by which we forcefully evict objects that
are marked as ``in-use'' (that is, in-flight or otherwise being used),
but haven't been accessed in a long time (currently three hours).

There is an additional stat that comes along with this (tailrepairs on
a slab) that will allow you to detect that this condition has occurred
on one of your slabs.

## Bugfixes ##
  * use a dedicated accept/dispatch thread.
  * prevent starvation by busy threads.
  * startup crash fix under certain distros.
  * better errors/warnings on the listen code.
  * fix listen errors in odd setups (no network, ipv4 only, etc).
  * ensure udp works in non-threaded mode.
  * update CAS on incr/decr properly.
  * incr/decr bugfixes.
  * improved tests
  * make 'stats slabs' used\_checks report correctly

## Contributors ##

The following people contributed to this release since 1.2.6. This is not a
measure of the amount of effort per commit, just the total.

```
    18  dormando
    11  Dustin Sallings
     4  Brian Aker
     1  Chris Goffinet
     1  Evan Klitzke
     1  Jonathan Bastien-Filiatrault
     1  Ricky Zhou
```