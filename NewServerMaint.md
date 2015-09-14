﻿#summary Mechanics Union



# Watching Server Health #

Memcached has a lot of statistical counters. Most are tallies, but others serve as warnings for an administrator to notice. Note that there are many tools and top programs floating around which serve to help you digest this information.

## Issuing Commands ##

Monitoring scripts should ideally issue test sets/gets/deletes to the servers occasionally, timing the response of each. If it's slow to connect you might have a connection or network problem. If it's slow to respond the server might be swapping or in ill health. You should always strictly monitor the health of a server, but an extra layer can be appreciated.

## Important Stats ##

These are found when issuing a simple `stats` command to a memcached server.

### curr\_connections ###

Lists the number of clients presently connected. Monitor that this number doesn't come too close to your max connection setting (`-c`).

### listen\_disabled\_num ###

An obscure named statistic counting the number of times memcached has hit its connection limit. When memcached hits the max connections setting, it disables its listener and new connections will wait in a queue. When someone disconnects, memcached wakes up the listener and starts accepting again.

Each time this counter goes up, you're entering a situation where new connections _will_ lag. Make sure it stays at or close to zero.

### accepting\_conns ###

Related to the above listen\_disabled\_num, if you're already connected to memcached you can see if it's hit max connections or not by checking if this value is 1 or 0.

### limit\_maxbytes ###

Sometimes it's nice to ensure that how you think memcached starts and how it actually starts are in line. By checking limit\_maxbytes you can verify that the `-m` argument took. Occasionally people using init scripts can be misconfigured and memcached will start with default values.

### cmd\_flush ###

While not exactly server health per-se, this is a good one to generically monitor. Every time someone issues a `flush_all` command, all items inside the cache are invalidated, and this counter is incremented. Sometimes debug code or misinformed people can leave scripts or callbacks running to "invalidate" the entire cache. Watch this value in production and sound the alarms if it starts moving, unless you really intended it to.

## `stats sizes` ##

Written a few times in these pages, it bears repeating; don't run this thing in production unless you can handle having the cache hang for up to several minutes. As of this writing this is the last command which iterates over all known cache items to run a calculation.

## Slab imbalance ##

A more complicated issue is an imbalance of the amount of memory assigned per slab, compared to where your actual data wants to go. Confusing speech for a simple algorithm:

  1. Monitor the global 'evictions' stat. If it starts going up. (or you can skip straight to #2)
  1. Monitor the 'evicted' and 'evicted\_nonzero' stats inside `stats items`. 'evicted\_nonzero' means the object was evicted early and did not have an infinite expiration time. This information is per-slab.
  1. Read the 'total\_pages' stats from `stats slabs`, and overlay those numbers with the evicted stats from above.
  1. If the slabs with the most evictions line up with the slabs containing the highest number of pages, you might legitimately be out of memory.
    * Alternatively (and perhaps more important) you can correlate the per-slab hitrate with the number of pages.
  1. If the slabs with the highest evictions do not line up with the pages very well, you need to restart memcached so it can re-allocate memory.

Sadly issuing a flush\_all does not do enough. You actually have to restart memcached if you ever end up in this position. It's rare but it happens.

Future versions will have memcached mitigate this situation itself.

## `tailrepairs` and `outofmemory` ##

These two stats appear in `stats items`, and can indicate bugs in the memcached code, or in the case of outofmemory, severe memory pressure paired with high read activity.

If you see these counters move with any regularity, please contact the mailing list with as much info as you can. While we haven't hit problems with these for a long time, we feel it's better to be safe and instrument the case of the more common runtime bugs.

## Troubleshooting Client Timeouts ##

See [Timeouts](Timeouts.md) for help.

# Stats for Application Health #

Monitoring these stats can tell you how the available memory in memcached, and how your app behaves, changes efficiency.

## Global hitrate ##

Hitrate is defined as: `get_hits / (get_hits + get_misses)`. The higher this value is, the more often your application is finding cache results instead of finding dead air. Watch that the value is both higher than what you expect, and watch to see if it changes with time or releases of your application.

## Hitrate per slab ##

While not possible to calculate the actual hitrate per-slab, since a "get miss" doesn't know how large the item _could have been_, you can monitor the number of get\_hits and cmd\_set on a per-slab basis via `stats slabs`. Watch that the number of get\_hits is usually above cmd\_set (more fetches than updates is often what you want), and watch for changes over time.

## Evictions ##

An item is "evicted" from the cache if it still has time to live, but ended up at the tail end of the LRU cache when it comes time to allocate a new item. While memcached tries a little to find an expired item from the tail, it doesn't guarantee it.

`stats items` shows per-slab eviction status. 'evicted' is the total number of items tossed early from that slab, 'evicted\_nonzero' are the number of tossed items that did not have an unlimited expiration time, and 'evicted - evicted\_nonzero' are the number of items tossed which were stored forever.

'evicted\_time' notes how many seconds it's been since the last item to be evicted, was last fetched. If this number is small, it means you're evicting items which were recently used.

## Looks Can be Deceiving ##

You see a low hit rate, a high eviction rate, and assume that you need to buy more memory for memcached. Unfortunately this isn't necessarily true either. Consider some potential scenarios:

  * Your application sets large numbers of keys, most of which are never used again.
  * Your application tests for one or more keys on most requests that do not exist and are not recached, such a flags or values for a buggy user.

The former can cause high eviction count even though the data was not important to begin with. Audit what our application stores and see if this is a fault.

The latter can show poor hitrates, as many get\_misses are accounted for by this flag. A workaround for this is to have the app "recache" a flag with a value of "not set". So the flag stays around in cache even if it's disabled, and doesn't throw off your hitrate calculations.

# OS Health (avoid swap!) #

Memcached interacts hard with the network and with RAM. It's common for people to monitor swap usage, which can cause severe performance degredation to memcached.

It's also important to watch your network stack. Linux, for example, sports a number of counters for its network interfaces related to dropped packets, underruns, crc errors, etc. Most switches also have per-port counters on packet issues or link issues. Degraded performance could trace down to bad wiring, bad switchport, a dying NIC, or a network driver in need of tuning.

# Upgrading #

Most minor releases of memcached focus on fixing bugs and adding instrumentation. New features are rare and tend to only happen on middle version bumps now. IE; 1.2 to 1.4 added the binary protocol and a large number of counters. 1.4.0 to 1.4.1 was mostly bugfixes and missing counters.

Minor features may show up as well. SASL authentication support happened in the mid-1.4 series, but the change is isolated and should not break existing clients.

You should follow new releases and the release notes. It's often better to uprade early so you have the extra instrumentation when you need it, instead of wishing you had it when things go wrong.