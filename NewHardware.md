﻿#summary Bling Bling



# Hardware Requirements #

Memcached is easy to spec out hardware for. In short, it is generally low on CPU usage, will take as much memory as you give it, and network usage will vary from mild to moderate, depending on the average size of your items.

## CPU Requirements ##

Memcached is typically light on CPU usage, due to its goal to respond very fast. Memcached is multithreaded, defaulting to 4 worker threads. This doesn't necessarily mean you have to run 100 cores to have memcached meet your needs. If you're going to need to rely on memcached's multithreading, you'll know it. For the common case, any bits of CPU anywhere is usually sufficient.

## RAM Requirements ##

The major point of memcached is to sew together sections of memory from multiple hosts and make your app see it as one large section of memory. The more memory the better. However, don't take memory away from other services that might benefit from it.

It is helpful to have each memcached server have roughly the same amount of memory available. Cluster uniformity means you can simply add and remove servers without having to care about one's particular "weight", or having one server hurt more if it is lost.

### Avoid Swapping ###

Assign physical memory, with a few percent extra, to a memcached server. Do not over-allocate memory and expect swap to save you. Performance will be very, very poor. Take extra care to monitor if your server is using swap, and tune if necessary.

### Is High Speed RAM Necessary? ###

Not so much, no. Getting that extra high speed memory will not likely net you visible benefits, unless you are issuing extremely high read traffic to memcached.

# Hardware Layouts #

## Running Memcached on Webservers ##

An easy layout is to use spare memory on webservers or compute nodes that you may have. If you buy a webserver with 4G of RAM, but your app and OS only use 2G of RAM at most, you could assign 1.5G or more to memcached instances.

This has a good tradeoff of spreading memory out more thinly, so losing any one webserver will not cause as much pain.

Caveats being extra maintenance, and keeping an eye on your application's multi-get usage, as it can end up accessing every memcached in your list. You also run a risk of pushing a machine into swap or killing memcached if your app has a memory leak. Often it's a good idea to run hosts with very little swap, or no swap at all. Better to let an active service die than have it turn into a tarpit.

## Running Memcached on Databases ##

Not a great idea. Don't do it. If you have a database host, give as much ram as possible to it. When cache misses do happen, you'll get more benefit from ensuring your indexes and data are already in memory.

## Using Dedicated Hosts ##

Using dedicated hardware for memcached means you don't have to worry about other programs on the machine interfering with memcached. You can put a lot of memory (64G+) into a single host and have fewer machines for your memory requirements.

This has an added benefit of being able to more easily expand large amounts of memory space. Instead of adding new webservers that may go idle, you can add specialized machines to throw gobs of RAM at the problem.

This ends up having several caveats. The more you compress down your memcached cluster, the more pain you will feel when a host dies.

Lets say you have a cache hitrate of 90%. If you have 10 memcached servers, and 1 dies, your hitrate may drop to 82% or so. If 10% of your cache misses are getting through, having that jump to 18% or 20% means your backend is suddenly handling **twice** as many requests as before. Actual impact will vary since databases are still decent at handling repeat queries, and your typical cache miss will often be items that the database would have to look up regardless. Still, **twice**!

So lets say you buy a bunch of servers with 144G of ram, but you can only afford 4 of them. Now when you lose a single server, 25% of your cache goes away, and your hitrate can tank even harder.

## Capacity Planning ##

Given the above notes on hardware layouts, be sure you practice good capacity planning. Get an idea for how many servers can be lost before your application is overwhelmed. Make sure you always have more than that.

If you cannot take down memcached instances, you ensure that upgrades (hardware or software), and normal failures are excessively painful. Save yourself some anguish and plan ahead.

## Network ##

Network requirements will vary greatly by the average size of your memcached items. Your application should aim to keep them small, as it can mean the difference between being fine with gigabit inter-switch uplinks, or being completely toast.

Most deployments will have low requirements (< 10mbps per instance), but a heavy hit service can be quite challenging to support. That said, if you're resorting to infiniband or 10 gigabit ethernet to hook up your memcached instances, you could probably benefit from spreading them out more.
