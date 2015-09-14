﻿#summary Punchcards and Mainframes



This basic tutorial shows via pseudocode how you can get started with integrating memcached into your application. If you're an application developer, it isn't something you just "turn on" and then your site goes faster. You have to pay attention.

If you're confused on how memcached works and integrates into an application, you may want to read the [TutorialCachingStory](TutorialCachingStory.md) if you haven't yet.

# Basic Data Caching #

The "hello world" of memcached is to fetch "something" from somewhere, maybe process it a little, then shove it into the cache, to expire in N seconds.

## Initializing a Memcached Client ##

Read the documentation carefully for your client.

```
# perl
my $memclient = Cache::Memcached->new({ servers => [ '10.0.0.10:11211', '10.0.0.11:11211' ]});
```

```
# pseudocode
memcli = new Memcache
memcli:add_server('10.0.0.10:11211')
```

Some rare clients will allow you add the same servers over and over again, without harm. Most will require that you carefully construct your memcached client object **once** at the start of your request, and perhaps persist it between requests. Initializing multiple times may cause memory leaks in your application or stack up connections against memcached until you cause a failure.

## Wrapping an SQL Query ##

Memcached is famous for reducing load on SQL databases. Unlike a query cache which can be centralized, implemented in slow middleware, or mass invalidated, you can easily get yourself moving by caching query results.

```
# Don't load little bobby tables
sql = "SELECT * FROM user WHERE user_id = ?"
key = 'SQL:' . user_id . ':' . md5sum(sql)
# We check if the value is 'defined', since '0' or 'FALSE' # can be 
# legitimate values!
if (defined result = memcli:get(key)) {
	return result
} else {
	handler = run_sql(sql, user_id)
	# Often what you get back when executing SQL is a special handler
	# object. You can't directly cache this. Stick to strings, arrays,
	# and hashes/dictionaries/tables
	rows_array = handler:turn_into_an_array
	# Cache it for five minutes
	memcli:set(key, rows_array, 5 * 60)
	return rows_array
}
```

Wow, zippy! When you cache this user's row(s), they will now see that same data for up to five minutes. Unless you actively invalidate the cache when a user makes a change, it can take up to five minutes for them to see a difference.

Often this is enough to help. If you have some complex queries, such as a count of users or number of posts in a thread. It might be acceptable to limit how often those queries can be issued by having a flat cache.

## Wrapping Several Queries ##

The more processing that you can turn into a single memcached request, the better. Often you can replace several SQL queries with a single, fast, memcached lookup.

```
sql1 = "SELECT * FROM user WHERE user_id = ?"
sql2 = "SELECT * FROM user_preferences WHERE user_id = ?"
key  = 'SQL:' . user_id . ':' . md5sum(sql1 . sql2)
if (defined result = memcli:get(key)) {
	return result
} else {
	# Remember to add error handling, kids ;)
	handler = run_sql(sql1, user_id)
	t[info] = handler:turn_into_an_array
	handler = run_sql(sql2, user_id)
	t[pref] = handler:turn_into_an_array
	# Client will magically take this hash/table/dict/etc
	# and serialize it for us.
	memcli:set(key, t, 5 * 60)
	return t
}
```

When you load a user, you fetch the user itself **and** their site preferences (whether they want to be seen by other users, what theme to show, etc). What was once two queries and possibly many rows of data, is now a single cache item, cached for five minutes.

## Wrapping Objects ##

Some languages allow you to configure objects to be serialized. Exactly how to do this in your language is beyond the scope of this document, however some tips remain.

  * Consider if you actually need to serialize a whole object. Odds are your constructor could pull from cache.
  * Serialize it as efficiently and simply as possible. Spending a lot of time in object setup/teardown can drag CPU.

Further consider, if you're deserializing a huge object for a request, and then using one small part of it, you might want to cache those parts separately.

## Fragment Caching ##

Once upon a time ESI (Edge Side Includes) were all the rage. Sadly they require special proxies/caching/etc. You can do this within your app for dynamic, authenticated pages just fine.

Memcached isn't just all about preventing database queries. You can cache computed items or objects as well.

```
# Lets generate a bio page!
user          = fetch_user_info(user_id)
bio_template  = fetch_biotheme_for(user_id)
page_template = fetch_page_theme
pagedata      = fetch_page_data

bio_fragment = apply_template(bio_template, user)
page         = apply_template(page_template, bio_fragment)
print "Content-Type: text/html", page
```

In this grossly oversimplified example, we're loading user data (which could be using a cache!), loading the raw template for the "bio" part of a webpage (which could be using a cache!). Then it loads the main template, which includes the header and footer.

Finally, it processes all that together into the main page and returns it. Applying templates can be costly. You can cache the assembled bio fragment, in case you're rendering a custom header for the viewing user. Or if it doesn't matter, cache the whole 'page' output.

```
key = 'FRAG-BIO:' . user_id 
if (result = memcli:get(key)) {
	return result
} else {
	user         = fetch_user_info(user_id)
	bio_template = fetch_biotheme_for(user_id)
	bio_fragment = apply_template(bio_template, user)
	memcli:set(key, bio_fragment, 5 * 15)
	return bio_fragment
}
```

See? Why do more work than you have to. The more you can roll up the faster pages will render, the happier your users.

# Extended Functions #

Beyond 'set', there are add, incr, decr, etc. They are simple commands but require a little finesse.

## Proper Use of `add` ##

`add` allows you to set a value if it doesn't already exist. You use this when initializing counters, setting locks, or otherwise setting data you don't want overwritten as easily. There can be some odd little gotchas and race conditions in handling of `add` however.

```
# There can be only one
key = "the_highlander"
real_highlander = memcli:get(key)
if (! real_highlander) {
	# Hmm, nobody there.
	var = fetch_highlander
	if (! memcli:add(key, var, 3600)) {
		# Uh oh! Somebody beat us!
		# We can either use the variable we fetched,
		# or issue `get` again in case it might be newer.
		real_highlander = memcli:get(key)
	} else {
		# We win!
	    gloat
	}
}
return real_highlander
```

## Proper Use of `incr` or `decr ##

`incr` and `decr` commands can be used to maintain counters. Such as how many hits a page has received, when you rate limit a user, etc. These commands will allow you to add values from 1 or higher, or even negative values.

They do not, however, initialize a missing value.

```
# Got a hit!
key = 'hits: ' . user_id
if (! memcli:incr(key, 1)) {
	# Whoops, key doesn't already exist!
	# There's a chance someone else just noticed this too,
	# so we use `add` instead of `set`
	if (! memcli:add(key, 1, 60 * 60 * 24)) {
		# Failed! Someone else already put it back.
		# So lets try one more time to incr.
		memcli:incr(key, 1)
	} else {
		return success
	}
} else {
	return success
}
```

If you're not careful, you could miss counting that hit :) You can doll this up and retry a few times, or no times, depending on how important you think it is. Just don't run a `set` when you mean to do an `add` in this case.

# Cache Invalidation #

Levelling up in memcached requires that you learn about actively invalidating (or revalidating) your cache.

When a user comes along and edits their user data, you should be attempting to keep the cache in sync some way, so the user has no idea they're being fed cached data.

## Expiration ##

A good place to start is to tune your expiration times. Even if you're actively deleting or overwriting cached data, you'll still want to have the cache expire occasionally. In case your app has a bug, a crash, a network blip, or some other issue where the cache could become out of sync.

There isn't a "rule of thumb" when picking an expiration time. Sit back and think about your users, and what your data is. How long can you go without making your users angry? Be honest with yourself, as "THEY _ALWAYS_ NEED FRESH DATA" isn't necessarily true.

Expiration times can be set from `0`, meaning "never expire", to 30 days. Any time higher than 30 days is interpreted as a unix timestamp date. If you want to expire an object on january 1st of next year, this is how you do that.

## `delete` ##

The simplest method of invalidation is to simply delete it, and have your website re-cache the data next time it's fetched.

So user Bob updates his bio. You want Bob to see his latest info when he so vainly reloads the page. So you:

```
memcli:delete('FRAG-BIO: ' . user_id)
```

... and next time he loads the page, it will fetch from the database and repopulate the cache.

## `set` ##

The most efficient idea is to actively update your cache as your data changes. When Bob updates his bio, take bob's bio object and shove it into the cache via 'set'. You can pass the new data into the same routine that normally checks for data, or however you want to structure it.

Play your cards right, and your database only ever handles writes, and data it hasn't seen in a long time.

## Invalidating by Tag ##

TODO: link to namespacing document + say how this isn't possible.

# Key Usage #

Thinking about your keys can save you a lot of time and memory. Memcached is a hash, but it also remembers the full key internally. The longer your keys are, the more bytes memcached has to hash to look up your value, and the more memory it wastes storing a full copy of your key.

On the other hand, it should be easy to figure out exactly where in your code a key came from. Otherwise many laborous hours of debugging wait for you.

## Avoid User Input ##

It's very easy to compromise memcached if you use arbitrary user input for keys. The ASCII protocol uses spaces and newlines. Ensure that neither show up your keys, live long and prosper. Binary protocol does not have this issue.

## Short Keys ##

64-bit UID's are clever ways to identify a user, but suck when printed out. 18446744073709551616. 20 characters! Using base64 encoding, or even just hexadecimal, you can cut that down by quite a bit.

With the binary protocol, it's possible to store anything, so you can directly pack 4 bytes into the key. This makes it impossible to read back via the ASCII protocol, and you should have tools available to simply determine what a key is.

## Informative Keys ##

```
key = 'SQL' . md5sum("SELECT blah blah blah")
```

... might be clever, but if you're looking at this key via tcpdump, strace, etc. You won't have any clue where it's coming from.

In this particular example, you may put your SQL queries into an outside file with the md5sum next to them. Or, more simply, appending a unique query ID into the key.

```
key = 'SQL' . query_id . ':' . m5sum("SELECT blah blah blah")
```