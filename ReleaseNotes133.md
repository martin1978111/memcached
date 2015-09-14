# Memcached 1.3 Beta 3 Release Notes #

Date: 2009-04-03 Fri

## Download ##

Download link:

http://memcached.googlecode.com/files/memcached-1.3.3.tar.gz

## Features ##

### Can set listen backlog on the commandline. ###

Prevent connection refused during connection storms at the cost of
kernel memory.

### stats settings ###

Show all current server settings (useful for troubleshooting as well
as internal verification).

## Bug fixes ##

  * Alignment bug in binary stats ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Occasional buffer overflow in stats ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Try to recycle memory more aggressively. ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * incr validation ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * 64-bit incr/decr delta fixes ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * ascii UDP set ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * stats slabs' used chunks ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * stats reset should reset item stats, eviction counters, etc... ([https://code.google.com/p/memcached/issues/detail?id=](https://code.google.com/p/memcached/issues/detail?id=))
  * Fix all stat buffer management

## Misc ##
  * More tests
  * More/better documentation
  * Code cleanup

## Stable fixes from Dormando ##

### New Stats ###

#### accepting\_conns ####

1 or 0 to indicate whether the server is currently accepting
connections or not.

The server will stop accepting connections when it has as many as it's
configured to take.

#### listen\_disabled\_num ####

The number of times socket listeners were disabled due to hitting the
connection limit.

#### cmd\_flush ####

The number of times the flush command was issued.

### missing key debugging ###

With verbosity enabled, you can see **why** objects were not found.  In
many cases, an item exists under a given key, but is considered
invalid due to lazy expiration or flush.

### tail repair ###

There is a rare, unidentified reference leak that causes a slab to be
full of invalid objects that cannot be evicted via the LRU nor will
they expire on their own.

Tail repair is a strategy by which we forcefully evict objects that
are marked as ``in-use'' (that is, in-flight or otherwise being used),
but haven't been accessed in a long time (currently three hours).

There is an additional stat that comes along with this (tailrepairs on
a slab) that will allow you to detect that this condition has occurred
on one of your slabs.

### socket listen bugs ###

There were some issues listening to sockets on machines with different
network interface configurations (i.e. no network, only ipv4, only
ipv6, etc...).

## Contributors ##

The following people contributed to this release since 1.3.2.  Please
refer to the 1.3.2 release notes for more info:

ReleaseNotes133

```
    28 Dustin Sallings
     8 Trond Norbye
     6 dormando
     5 Brad Fitzpatrick
     4 Steve Yen
     1 Eric Lambert
     1 Clinton Webb
     1 Chris Goffinet
```