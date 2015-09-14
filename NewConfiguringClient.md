﻿#summary You Can't Get There From Here



# Common Client Configurables #

Most clients are similar in some important ways. They may implement some ideas differently, but they contain many common concepts to twiddle and fiddle.

## Hashing ##

All clients support at least one method of "hashing" keys among servers. Keep in mind that most of these defaults are not compatible with each other. If you're using the perl Cache::Memcached and expect to resolve keys to servers the same way as a PHP client, you're in for trouble.

There are exceptions to this, as clients based on [libmemcached](http://libmemcached.org) should all have access to the same hasing algorithms.

## Consistent Hashing ##

Consistent Hashing is a model that allows for more stable distribution of keys given addition or removal of servers. In a normal hashing algorithm, changing the number of servers can cause many keys to be remapped to different servers, causing huge sets of cache misses. [Consistent Hashing](http://en.wikipedia.org/wiki/Consistent_hashing) describes methods for mapping keys to a list of servers, where adding or removing servers causes a very minimal shift in where keys map to.

So in short, with a normal hashing function, adding an eleventh server may cause 40%+ of your keys to suddenly point to different servers than normal.

However, with a consistent hashing algorithm, adding an eleventh server should cause less than 10% of your keys to be reassigned. In practice this will vary, but it certainly helps.

TODO: I know there's a better discussion of this that's linkable. help find it? I can never describe it well enough.

## Configuring Servers **Consistently** ##

When adding servers to your configuration, pay attention that the list of servers you supply to your clients are exactly the same across the board.

If you have three webservers, and each webserver is also running a memcached instance, you may think it would be clever to address the "local" instance as "localhost". This will **not** work as expected, as the servers are now different between webservers. This means webserver 1 will map keys differently than server 2, causing mass hysteria among your users and business development staff.

The ordering is also important. Some clients will sort the server list you supply to them, but others will not. If you have servers "A, B, C", list them as "A, B, C" everywhere.

Use Puppet/Chef/rsync/whatever is necessary to ensure these files are in sync :)

## "Weighting" ##

Given an imperfect world, sometimes you may have one memcached instance that has more RAM available than others. Some clients will allow you to apply more "weight" to the larger server. Others will allow you to specify one server multiple times to get it more chances of being selected.

Either way, you'd probably do well to verify that the "weighting" is doing what you expect it to do.

## Failure, or Failover ##

What will your client do when a server is unavailable or provides an invalid response?

In the dark days of memcached, the default was to always "failover", by trying the next server in the list. That way if a server crashes, its keys will get reassigned to other instances and everything moves on happily.

However there're many ways to kill a machine. Sometimes they don't even like to stay dead. Given the scenario:

  * Sysadmin Bob walks by Server B and knocks the ethernet cable out of its port.
  * Server B's keys get "rerouted" to other instances.
  * Sysadmin Bob is an attentive (if portly) fellow and dutifully restores the ethernet cable from its parted port.
  * Server B's keys get "rerouted" back to itself.

Now everything goes scary. Any updates you've made to your cache in the time it took Bob to realize his mistake have been lost, and old data is presented to the user. This gets even worse if:

  * Server B's ethernet clip was broken by Bob's folly and later falls out of its port unattended.

Now your data has flipped back to yet another set. Annoying.

Another erroneous client feature would actually amend the server list when a server goes out of commission, which ends up remapping far more keys than it should.

Modern life encourages the use of "Failure", when possible. That is, if the server you intend to fetch or store a cache entry to is unavailable, simply proceed as though it was a cache miss. You might still flap between old and new data if you have a Server B situation, but the effects are reduced.

## Compression ##

Compressing large values is a great way to get more bang out of your memory buck. Compression can save a lot of memory for some values, and also potentially reduce latency as smaller values are quicker to fetch over the network.

Most clients support enabling or disabling compression by threshold of item size, and some on a per-item basis. Smaller items won't necessarily benefit as much from having their data reduced, and would simply waste CPU.

## Managing Connection Objects ##

A common first-timer failure is that no matter what you do, you seem to run memcached flat out of connections. Your small server is allocating 50,000 connections to memcached and you have no idea what's going on.

Be wary of how you manage your connection objects! If you are constantly initializing connection objects every time you wish to contact memcached, odds are good you're going to leak connections.

Some clients (like PHP ones) have a less obvious approach to managing how many connections it will open. Continually calling 'addServer()' may just leak connections on you, even if you've already added a server. Read your clients' documentation to confirm what actions create connections and what will not.