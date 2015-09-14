﻿#summary what is this thing?



# Memcached #

**Free & open source, high-performance, distributed memory object caching system**, generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load.

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.

**Memcached is simple yet powerful**. Its simple design promotes quick deployment, ease of development, and solves many problems facing large data caches. Its API is available for most popular languages.

At heart it is a simple Key/Value store.

# Memcached In Slightly More Words #

See the [memcached.org about page](http://memcached.org/about) for a brief overview.

## I Turned It On and My App Isn't Faster!!! ##

Memcached is a developer tool, not a "code accelerator", nor is it database middleware. If you're trying to set up an application you have downloaded or purchased to use memcached, your best bet is to head back their way and read your app's documentation on how to utilize memcached. Odds are these documents will not help you much.

## What is it Made Up Of? ##

  * Client software, which is given a list of available memcached servers.
  * A client-based hashing algorithm, which chooses a server based on the "key" input.
  * Server software, which stores your values with their keys into an internal hash table.
  * Server algorithms, which determine when to throw out old data (if out of memory), or reuse memory.

## What are the Design Philosophies? ##

### Simple Key/Value Store ###

The server does not care what your data looks like. Items are made up of a key, an expiration time, optional flags, and raw data. It does not understand data structures; you must upload data that is pre-serialized. Some commands (incr/decr) may operate on the underlying data, but the implementation is simplistic.

### Smarts Half in Client, Half in Server ###

A "memcached implementation" is implemented partially in a client, and partially in a server. Clients understand how to send items to particular servers, what to do when it cannot contact a server, and how to fetch keys from the servers.

The servers understand how to receive items, and how to expire them.

### Servers are Disconnected From Each Other ###

Memcached servers are generally unaware of each other. There is no crosstalk, no syncronization, no broadcasting. The lack of interconnections means adding more servers will usually add more capacity as you expect. There might be exceptions to this rule, but they are exceptions and carefully regarded.

### O(1) Everything ###

For everything it can, memcached commands are O(1). Each command takes roughly the same amount of time to process every time, and should not get noticably slower anywhere. This goes back to the "Simple K/V Store" principle, as you don't want to be processing data in the cache service your tens or hundreds or thousands of webservers may need to access at the same time.

### Forgetting Data is a Feature ###

Memcached is, by default, a Least Recently Used cache. It is designed to have items expire after a specified amount of time. Both of these are elegant solutions to many problems; Expire items after a minute to limit stale data being returned, or flush unused data in an effort to retain frequently requested information.

This further allows great simplification in how memcached works. No "pauses" waiting for a garbage collector ensures low latency, and free space is lazily reclaimed.

### Cache Invalidation is a Hard Problem ###

Given memcached's centralized-as-a-cluster nature, the job of invalidating a cache entry is trivial. Instead of broadcasting data to all available hosts, clients direct in on the exact location of data to be invalidated. You may further complicate matters to your needs, and there are caveats, but you sit on a strong baseline.

## How Is Memcached Evolving? ##

Memcached has been evolving as a platform. Some of this is due to the slow development nature, and the many clients. Mostly it happens as people explore the possibilities of K/V stores. Learning to cache SQL queries and rendered templates used to keep developers occupied, but they thirst for more.

### The Protocol ###

Memcached is not the only implementation of the protocol. There are commercial entities, other OSS projects, and so on. Memcached clients are fairly common, and there are many uses for a memcached-like cluster layout. We will continue to see other projects "speak" memcached, which in turn influences memcached as a culture and as software itself.

### Other Protocols ###

The industry is experimenting with many different ways to communicate with networked services. Google protocol buffers, Facebook Thrift, Avro, etc. There might be better ways to do this, as the future will show.

### Persistent Storage ###

Many users want K/V stores that are able to persist values beyond a restart, or beyond available physical memory. In many cases memcached is not a great fit here; expensive flash memory is needed to keep data access performant. In most common scenarios, disaster can ensue if a server is unavailable and later comes up with old data. Users see stale data, features break, etc.

However, with the price of SSD's dropping, this is likely an area to be expanded into.

### Storage Engines ###

Storage Engines in general are an important future to memcached as a service. Aside from our venerable slabbing algorithm, there are other memory backends to experiment with. tcmalloc, jemalloc, CPU-local slabs, hierarchies, etc. Storage engines foster experimentation to improve speed and memory efficiency, as well as specialized services able to speak the memcached protocol.
