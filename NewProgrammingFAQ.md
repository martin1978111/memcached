﻿#summary Never Stops For Directions



# Basics #

## How can you list all keys? ##

With memcached, you can't list all keys. There is a debug interface, but that is not an advisable usage.

### Why are you trying to dump the cache? ###

Think about why you need to do it. If you're trying to troubleshoot your application, we find that it's often more informative to look at the **flow** rather than the current state. Run memcached in a screen session with `-vv` or `-vvv` to have it print what it's doing. You may also use [MaatKit](http://www.maatkit.org) to use tcpdump to analyze traffic to an instance.

Watching the flow is very useful. You can see if your application is fetching keys inappropriately (one at a time vs multiget, or repeatedly in a single request), or you can see why memcached decided to invalidate a key (if ran in `-vvv` mode). Inspecting the state can't tell you any of this useful information anyway.

### My application requires it ###

If it's a requirement that your application is able to pull or walk keys, you really want a database. Tokyo Tyrant, MySQL, etc, are good candidates for this. Memcached as a caching service cannot support the ability to safely walk keys without locking out all other operations. Adding indexes, multiversioning, etc, can make this possible but will lower memory and cpu efficiency.

You "can" via the debug interface `stats cachedump`, but that will only ever be a partial dump, and is slow.

## Why only RAM? ##

Everything memcached does is an attempt to guarantee latency and speed. If you have to sometimes hit disk, that's no longer true.

## Why no complex operations? ##

All operations should run in O(1) time. They must be atomic. This doesn't necessarily mean complex operations can never happen, but it means we have to think very carefully about them first. Many complex operations can be emulated on top of more basic functionality.

## Why is memcached not recommended for sessions? Everyone does it! ##

If a session disappears, often the user is logged out. If a portion of a cache disappears, either due to a hardware crash or a simple software upgrade, it should not cause your users noticable pain. [This overly wordy post](http://dormando.livejournal.com/495593.html) explains alternatives. Memcached can often be used to reduce IO requirements to very very little, which means you may continue to use your existing relational database for the things it's good at.

Like keeping your users from being knocked off your site.

## What about the MySQL query cache? ##

The MySQL query cache can be a useful start for small sites. Unfortunately it uses many global locks on the mysql database, so enabling it can throttle you down. It also caches queries per table, and has to expire the entire cache related to a table when it changes, at all. If your site is fairly static this can work out fine, but when your tables start changing with any frequency this immediately falls over.

Memory is also limited, as it requires using a chunk of what's directly on your database.

## Is memcached atomic? ##

Aside from any bugs you may come across, yes all commands are internally atomic. Issuing multiple sets at the same time has no ill effect, aside from the last one in being the one that sticks.

## Why is there a binary protocol? ##

Because it's awesome. TODO: link to the new protocol page.

## How do I troubleshoot client timeouts? ##

See [Timeouts](Timeouts.md) for help.

# Setup Questions #

## How do I authenticate? ##

You don't! Well, you used to not be able to. [Now you can](SASLHowto.md). If your client supports it, you may use SASL authentication to connect to memcached.

Keep in mind that you should do this only if you really need to. On a closed internal network this ends up just being added latency for new connections (if minor).

## How do you handle failover? ##

You don't. Some clients have a "failover" option that will try the next server in the case of a failure. As noted in [Configuring Clients](NewConfiguringClient.md) this isn't always the best idea.

## How do you handle replication? ##

It doesn't. Adding replication to the system halves your effective cache size. If you can't handle even a few percent extra cache misses, you have serious problems. Even with replication, things can break. More moving parts. Software to crash.

## Can you persist cache between restarts? ##

No. Sometime in the future it might, but as of now it does not. This is often a bad idea since the cache that comes back up will be out of date. Folks who use this really want a database instead.

## Do clients and servers all need to talk to each other? ##

Nope. The less chatter, the more scalable.

# Monitoring #

## Why Isn't curr\_items Decreasing When Items Expire? ##

Expiration in memcached is lazy.  In general, an item cannot be known to be expired until something looks at it.

Think of it this way:  You can add billions of items to memcached that all expire at the exact same second, but no additional work is performed during that second by memcached itself.  Only as you attempt to retrieve (or update) those items will memcached ever notice that they shouldn't be there.  At this point, curr\_items will be decremented by each item it has seen expired.

We may also notice expired items while searching for memory for new items, though this isn't likely to create an observable difference in curr\_items because we'll be replacing it with a new item anyway.

# Use Cases #

## When would you not want to use memcached? ##

It doesn't always make sense to add memcached to your application.

TODO: link to that whynot page here or just inline new stuff?

## Why can't I use it as a database? ##

Because it's a cache. Storage engines will start to support this use case, but primarily there're benefits for treating this **as a cache**, even if you were using a Key/Value database, it can be useful to have a cache in front of it.

## Can using memcached make my application slower? ##

Yes, absolutely. If your DB queries are all fast, your website is fast, adding memcached might not make it faster.

Also, this:

```
my @post_ids = fetch_all_posts($thread_id);
my @post_entries = ();
for my $post_id (@post_ids) {
	push(@post_entries, $memc->get($post_id));
}
# Yay I have all my post entries!
```

See that? Don't do that. Use a multi-get. Fetching a single item from memcached still requires a network roundtrip and a little processing. The more you can fetch at once the better.

# Architectural #

## Why can't we use memcached as a queue server? ##

TODO: need to expand on this more.

To be succinct: queue servers should either push to their workers, or notify their workers. If using memcached, they must constantly poll. Scaling out beyond one server also ends up being silly. Workers poll all servers, and that doesn't mesh well with memcached clients, who want to resolve keys to a particular server.

If you think your memcached client is broken because you're trying to use it with multiple memcached queues, it's not broken. Sorry :)