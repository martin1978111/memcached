﻿#summary No Handlebars



## Namespacing ##

Memcached does not natively support namespaces or tags. It's difficult to support this natively as you cannot atomically expire the namespaces across all of your servers without adding quite a bit of complication.

However you can emulate them easily.

### Simulating Namespaces with Key Prefixes ###

Using a coordinated key prefix, you can create a virtual namespace that spans your entire memcached cluster. The prefix can be stored in your configuration and changed manually, or stored in an external key.

### Deleting By Namespace ###

Given a user and all his related keys, you want a one-stop switch to invalidate all of their cache entries at the same time.

Using namespacing, you would set up a tertiary key with a version number inside it. You end up doing an extra round trip to memcached to figure the namespace.

```
user_prefix = memcli:get('user_namespace:' . user_id)
bio_data    = memcli:get(user_prefix . user_id . 'bio')
```

Invalidating the namespace simply requires editing that key. Your application will no longer request the old keys, and they will eventually fall off the end of the LRU and be reclaimed.

Careful in how you implement the prefix. You'll want to use `add` so you don't blow away an existing namespace. You'll also want to initialize it to something with a low probability of coming up again.

An easy recommendation is a unix timestamp.

```
# Namespace management, basic fetch.
key = 'namespace:' . user_id
namespace = memcli:get(key)
if (!namespace) {
    namespace = time()
	if (! memcli:add(key, namespace)) {
		# lost the race.
		namespace = memcli:get(key)
		# Could re-test it and jump to the start of the loop, hard fail, etc.
	}
	# Send back the new namespace.
	return namespace
}
```

And on invalidation:

```
key = 'namespace:' . user_id
if (! memcli:incr(key, 1)) {
	# Increment failed! Key must not exist.
	memcli:add(key, time())
}
```

This isn't a perfect algorithm either, but a simple one. The key is initialized via a timestamp, and then incremented by one each time the data is to be invalidated. This works well if the invalidations are infrequent, as a missing key will always end up being replaced with a larger number than was slowly incremented from before.

You can drop the race condition further by using millisecond resolution instead of seconds, but that makes your key prefix longer. For bonus points, base64 encode the number before sticking it in front of the other keys.

## Storing sets or lists ##

Storing lists of data into memcached can mean either storing a single item with a serialized array, or trying to manipulate a huge "collection" of data by adding, removing items without operating on the whole set. Both should be possible.

One thing to keep in mind is memcached's 1 megabyte limit on item size, so storing the whole collection (ids, data) into memcached might not be the best idea.

Steven Grimm explains a better approach on the mailing list: http://lists.danga.com/pipermail/memcached/2007-July/004578.html

Chris Hondl and Paul Stacey detail alternative approaches to the same ideal: http://lists.danga.com/pipermail/memcached/2007-July/004581.html

A combination of both would make for very scalable lists. IDs between a range are stored in separate keys, and data is strewn about using individual keys.

## Managing lists with `append`/`prepend` ##

Assuming you pick a route of storing a list of ids (numbers) in one a memcached key, and fetch the full data of a sets items separately, you can use `append`/`prepend` for atomic updates.

Lets take an AJAX-y feature of managing a users' list of interests. You give the user a box to type into. They type "hammers" and your fancy AJAX script updates their list of interests with "hammers", and anyone viewing their profile can instantly see "hammers" added to their list of interests. Normally you either have to try a CAS update to fetch the old cache item, add the hammer interest-id, then re-set the value, or simply delete the cache and pull the whole list from the database on the next view.

Instead, you can use append. When adding an interest-id onto the users' interest cache entry, simply issue an append command with a binary packed string representing the id. Do this similar to incr/decr, as append will fail if the cache list doesn't already exist.

So `append`, if fail, load from database and run `add`, if fail, decide how much you care to ensure the item got in and do more work.

Now lets say that user adds several hundred interests, then goes back to the beginning and decides he doesn't like "hammers" anymore, as he actually likes nails more. How do you handle this? You still have the old options of trying to pull the list, edit it, and CAS it back in, or deleting the whole thing. Or you can maintain a blacklist.

When initializing the list, the first byte in the list can be a "zero marker", a whole four or eight byte (depending on how big your interest-ids are) value that contains nothing but zeros.

When you're loading the list in from memcached, the first set of items you read will be "blacklist" items. Once you hit a value of "0", start reading the list as items that are supposed to exist. You can check each item against the "blacklist" and not enter it into the list for display.

So for common cases where the list won't get too large, you can add stuff to the end and then prepend removed items to the front. If the cache gets blown and reloaded, it won't have the blacklisted items in it to begin with, so the cache entry is cleaned.

This has obvious limitations based on cache size, but is a clever way to handle avoiding excessively expensive recaching operations with fickle users.

## Zero byte values ##

Don't do this:

```
if (data = memcli:get('helloworld')) {
	# Yay stuff!
}
```

... because you could have perfectly valid data that has a result of 0, or false, or empty. Empty keys are useful for advisory locks, caching status flags, and the like. If you can get all that you need from the existence of the key alone, you don't need to waste bytes with extra data.

## Reducing key size ##

The smaller your keys, the less memory overhead you have. With smaller items in the lower slab classes this can matter even more, as shaving a few bytes could end up putting the item into a more efficient slab class. Also, keys are limited to 250 characters (effectively. this may be raised to 65k in the future).

Compress keys when it's easy or makes sense. "super\_long\_function\_names\_abstract\_key" might be descriptive but is a waste. Boil it down to a function id you can grep your code for. "slfnak", or whatever.

Base64 encode long numbers. Easy enough to use a commandline program to turn that back into a numeric.

Binary protocol allows setting arbitrary keys. Instead of base64 encoding you can byte pack them down to their native size. Also instead of 'sflnak', you could pack a two byte identifier and map the number back to your code.

However, don't do any of this unless you're really hurting for extra memory. Some easy changes like this can sometimes save between 5 and 20% of your memory, but often buying a few gigs of ram is cheaper than your time.

## Accelerating counters safely ##

TODO: This is a memcached/mysql hybrid for avoiding running `count(*) from table where user_id = ?` constantly. I'll be fleshing this out later since I see some bugs in what I have here (deadlocks?)

## Rate limiting ##

TODO: There were a couple decent posts on this. I've seen some slides that were buggy. Need to round them up.

## Ghetto central locking ##

While we don't recommend doing this for any "serious" locking situation, sometimes you would benefit from an advisory, sometimes reliable "lock" obtainable via memcached.

Given a cache item that is popular and difficult to recreate, you could end up with dozens (or hundreds) of processes slamming your database at the same time in an attempt to refill a cache. Discussed more below as the "stampeding herd" problem, we'll describe a simple method of using `add` to create an advisory "ghetto lock"

```
key  = "expensive_frontpage_item"
item = memcli:get(key)
if (! defined item) {
	# Oh crap, we have to recache it!
	# Give us 60 seconds to recache the item.
	if (memcli:add(key . "_lock", 60)) {
		item = fetch_expensive_thing_from_database
		memcli:add(key, item, 86400)
		memcli:delete(key . "_lock")
	} else {
		# Lost the race. We can do any number of things:
        # - short sleep, then re-fetch.
        # - try the above a few times, then slow-fetch and return the item
        # - show the user a page without this expensive content
        # - show some less expensive content
        # - throw an error
	}
}
return item
```

Worst case you can end up operating without the lock at all. Best case you can reduce the amount of parallel queries going on without adding more infrastructure.

Use at your own risk! 'add' can fail because the key already exists, or because the remote server was down. If your client doesn't give you a way to tell the difference, you have to make a decision on how hard to try before running the query anyway or throwing an error.

## Avoiding stampeding herd ##

It's a big problem when cache misses on hot (or expensive) items cause a mess of application processes to slam your database for answers. There are a large array of choices one has to avoid this problem, and we'll discuss a few below.

### Ghetto lock ###

As shown above, in a pinch you can reduce the odds of needing to run the query by using memcached's `add` feature.

### Outside mutex ###

A third party centralized mutex can also be used. MySQL has `SELECT GET_LOCK() ... RELEASE_LOCK()` which is fast but requires bothering your database a little bit. Other services exist as well, but this author isn't confident enough in what he knows to recommend any ;)

### Scaling expiration ###

A common trick is to use soft expiration values embedded in your cached object. If your object is due to expire in an hour, set it to actually expire in 1.5 hours or more. Inside your object set a "soft timeout" for when you think the object is old.

When you fetch an object and it has passed the soft timeout, you can pick any method that agrees with you to re-cache it:

  * Do a "lock" as noted above. If you fail to aquire the lock, return the old cached item. Lock winner recaches.
  * Also store a "hard" timeout, or just assume the hard timeout is soft timeout + a value. Randomly decide if you want to recache, and increase the odds of recaching the value the older the item is.
  * Dispatch an asyncronous job to recache the object.
  * etc.

The cache object can still go away for many reasons (server restart, LRU, eviction, etc). Use this as mitigation but not your only line of defense.

### Gearmand or similar ###

Using a job server can be an easy win. [Gearman](http://gearman.org/) is a common, fast, scalable job service. While funneling recache requests through a job server will certainly add overhead, you can selectively use the service or rely purely on background jobs. Perhaps you'll want to funnel high traffic users through gearmand, but no one else.

Gearmand has two magic tricks; asyncronous or syncronous job processing.

In the case of a scaling expiration value, you can issue an asyncronous job to recache the object, then return the cache to a user. Gearmand can collapse similar jobs down so you don't end up executing millions of them.

In the case of a syncronous update, gearmand can coalesce incoming jobs with the same parameters. So the first process to issue the job request will get a worker to recache the data. Every other procecss after him will "subscribe" to the results of that first job, and not create more parallelism. When the first job finishes, gearmand broadcasts the response to all listeners and they all continue forward as though they had issued the request directly.

Very handy.

## Ghetto replication ##

Some clients may natively support replication. It will pick two unique servers to store a value on, and either linearly or randomly retrieve the value again. Sometimes you don't have this feature!

You can "try" but not guarantee replication by modifying your key and storing a value twice. If you want to store a highly retrieved value from three locations, you could add '1', '2', or '3' to the end of the key, and store them all. Then on fetch randomly pick one.

Has a lot of gotchas; storage failure means you're more likely to get stale data back. Cute hack if you're in a pinch though.

## "Touching" keys with `add` ##

Create an item that you want to expire in a week? Don't always fetch the item but want it to remain near the top of the LRU for some reason? `add` will actually bump a value to the front of memcached's LRU if it already exists. If the `add` call succeeds, it means it's time to recache the value anyway.
