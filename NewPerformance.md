﻿#summary Ponies


We expect memcached to be fast. Occasionally folks test memcached and see something they don't expect. This is a short list of how we (the developers and users) expect memcached to perform. If you're seeing different, it's likely a configuration problem at some layer. You should troubleshoot or request help in getting a properly oiled setup.

# General Performance #

---


## Handles Extremely High Load ##

On a fast machine with very high speed networking, memcached can easily handle 200,000+ requests per second. With heavy tuning or even faster hardware it can go many times that. Hitting it a few hundred times per second, even on a slow machine, usually isn't cause for concern.

## It Should Not Hang ##

Memcached operations are almost all O(1). Connecting to it and issuing a get or stat command should never lag. If connecting lags, you may be hitting the max connections limit. See [ServerMaint](NewServerMaint.md) for details on stats to monitor.

If issuing commands lags, you can have a number of tuning problems. Most common are hardware problems, not enough RAM (swapping), network problems (bandwidth, dropped packets, half-duplex connections). On rare occasion OS bugs or memcached bugs can contribute.

### OS Issues / Firewalls ###

It's not usually necessary to run a stateful firewalling system in front of a memcached server. You might if you run them by default, or need to protect memcached from the internet at large.

These systems have limits in the number of connections that can pass through it in a certain number of time, and how fast those connections or packets can pass through. A lot of connectivity bugs end up being tracked back to someone's firewall.

## It Should Respond Quickly ##

On a good day memcached can serve requests in less than a millisecond. After accounting for outliers due to OS jitter, CPU scheduling, or network jitter, very few commands should take more than a millisecond or two to complete.

## Expiration Times Should Be Accurate ##

Memcached tries to expire items when you tell it to. To avoid hitting the clock constantly, it updates an internal clock value every second. This means your expiration times should be accurate up to the nearest second. So an expiration can be a second early or a second late, but should not waver beyond that.

Since expiration values are stored as timestamps, be sure your OS clock is correct. If your OS clock is set to some time in the future, you're storing items into memcached, and you suddenly adjust the clock far into the past, those items will not expire on time. Inversely, jumping a clock forward can prematurely expire items.

## How It Handles `set` Failures ##

There are conditions which memcached will "fail" a set. If it cannot find free memory, if the object is too large, or some unknown factor. In most cases where you are issuing a `set` against an item that already exists, a failure will cause the old item to be removed from the cache.

Memcached assumes that if you are issuing a set, even if the item is invalid, that means your intent was to update the cache, and whatever it contains is now stale. It error's on the side of caution and throws the old item out for you.

# Theoretical Limits #

---


## Max Clients ##

Since memcached uses an event based architecture, a high number of clients
will not generally slow it down. Users have hundreds of thousands of connected
clients, working just fine.

There are a few hard limits:

  * Each connected client uses some TCP memory. You can only connect as many clients as you have spare RAM

  * There is only one thread to accept new client connections. If you are cycling connections very quickly, you can overwhelm the thread. Use persistent connections or UDP in this case.

  * High connection churn requires OS tuning. You will run out of local ports, TIME\_WAIT buckets, and similar. Do research on how to properly tune the TCP stack for your OS.

## Maximum number of nodes in a cluster ##

### From the Client Perspective ###

A large number of servers can slow down a client from a few angles. If you are
calculating the server hash table on every request, that will slow down as
you add servers. Most clients will calculate the table once on startup and
reuse it between requests.

Looking up a server to issue a request is simply a hash lookup, so it is not a
performance issue.

  * Establishing too many TCP sockets from the client wastes RAM

  * Disabling persistent connections means the client will likely connect to all servers on every request. The 3 way handshake will add latency and packets to each request.

### The Multiget Hole ###

There was once a discussion on a facebook note noting that as you add servers
to a cluster, the more spread out multigets are. If you issue a multiget
request for 10 keys against a 2 server cluster, that will turn into two
syscalls; one for each server. If you have 10 servers, you will end up with
one syscall per server. The servers themselves are doing more syscalls to cope
with the smaller incoming requests, and their capacity drops.

This is not an insurmountable problem. It's possible with many clients to
group related keys together. IE; all keys for a particular user's profile
would end up on the same memcached instance, so a multiget will hit a single
server with all of its keys and use far fewer syscalls in total.

### A Well Designed Binary Protocol Client ###

Using spymemcached as an example; the client may take many application threads
and use a single TCP connection back to memcached. With the binary protocol,
it is possible to pack requests from different client instances into the same
TCP socket, then dole back results to the right owners.

This means an application server, could in theory only require a single TCP
socket per memcached instance. If you run 50 threads or processes per
application node, this can vastly reduce overhead from large installs.

### Remember Moore's Law ###

If many large sites were using the same hardware they had in 2007 for new
memcached instances, they could have four times or more the server count than
they do now. As years pass memory becomes cheaper and network capacity
increases. Hardware increases won't necessarily stay behind your growth curve,
but they do take a huge edge off as you can in theory halve your server count
every few years.

### Economy of Scale ###

I will also briefly note that massive installations attract better talent. The
reality of the situation is much more likely to involve domain experts if your
traffic is massive. The larger the cluster, the weirder the issues you run
into, the more likely you are to have a team of people involved in maintaining
it. These are extreme cases for extremely large clusters requiring guaranteed
performance. Most users will have occasional problems to overcome, but not
enough to warrant hires.
