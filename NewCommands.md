﻿#summary Make Me a Sandwich



Memcached handles a small number of basic commands.

Full documentation can be found in the [Protocol Documentation](NewProtocols.md).

## Standard Protocol ##

The "standard protocol stuff" of memcached involves running a command against an "item". An item consists of:

  * A key (arbitrary string up to 250 bytes in length. No space or newlines for ASCII mode)
  * A 32bit "flag" value
  * An expiration time, in seconds. Can be up to 30 days. After 30 days, is treated as a unix timestamp of an exact date.
  * A 64bit "CAS" value, which is kept unique.
  * Arbitrary data

CAS is optional (can be disabled entirely with `-C`, and there are more fields that internally make up an item, but these are what your client interacts with.

### No Reply ###

Most ASCII commands allow a "noreply" version. One should not normally use this with the ASCII protocol, as it is impossible to align errors with requests. The intent is to avoid having to wait for a return packet after executing a mutation command (such as a set or add).

The binary protocol properly implements noreply (quiet) statements. If you have a client which supports or uses the binary protocol, odds are good you may take advantage of this.

## Storage Commands ##

### set ###

Most common command. Store this data, possibly overwriting any existing data. New items are at the top of the LRU.

### add ###

Store this data, only if it does not already exist. New items are at the top of the LRU. If an item already exists and an add fails, it promotes the item to the front of the LRU anyway.

### replace ###

Store this data, but only if the data already exists. Almost never used, and exists for protocol completeness (set, add, replace, etc)

### append ###

Add this data after the last byte in an existing item. This does not allow you to extend past the item limit. Useful for managing lists.

### prepend ###

Same as append, but adding new data before existing data.

### cas ###

Check And Set (or Compare And Swap). An operation that stores data, but only if no one else has updated the data since you read it last. Useful for resolving race conditions on updating cache data.

## Retrieval Commands ##

### get ###

Command for retrieving data. Takes one or more keys and returns all found items.

### gets ###

An alternative get command for using with CAS. Returns a CAS identifier (a unique 64bit number) with the item. Return this value with the `cas` command. If the item's CAS value has changed since you `gets`'ed it, it will not be stored.

## delete ##

Removes an item from the cache, if it exists.

## incr/decr ##

Increment and Decrement. If an item stored is the string representation of a 64bit integer, you may run incr or decr commands to modify that number. You may only incr by positive values, or decr by positive values. They does not accept negative values.

If a value does not already exist, incr/decr will fail.

## Statistics ##

There're a handful of commands that return counters and settings of the memcached server. These can be inspected via a large array of tools or simply by telnet or netcat. These are further explained in the protocol docs.

### stats ###

ye 'ole basic stats command.

### stats items ###

Returns some information, broken down by slab, about items stored in memcached.

### stats slabs ###

Returns more information, broken down by slab, about items stored in memcached. More centered to performance of a slab rather than counts of particular items.

### stats sizes ###

A special command that shows you how items would be distributed if slabs were broken into 32byte buckets instead of your current number of slabs. Useful for determining how efficient your slab sizing is.

**WARNING** this is a development command. As of 1.4 it is still the only command which will lock your memcached instance for some time. If you have many millions of stored items, it can become unresponsive for several minutes. Run this at your own risk. It is roadmapped to either make this feature optional or at least speed it up.

## flush\_all ##

Invalidate all existing cache items. Optionally takes a parameter, which means to invalidate all items after N seconds have passed.

This command does not pause the server, as it returns immediately. It does not free up or flush memory at all, it just causes all items to expire.