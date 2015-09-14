﻿#summary No Guts No Glory



It is important that developers using memcached understand a little bit about how it works internally. While it can be a waste to overfocus on the bits and bytes, as your experience grows understanding the underlying bits become invaluable.

Understanding memory allocation and evictions, and this particular type of LRU is most of what you need to know.

## How Memory Gets Allocated For Items ##

Memory assigned via the `-m` commandline argument to memcached is reserved for item data storage. The primary storage is broken up (by default) into 1 megabyte pages. Each `page` is then assigned into `slab classes` as necessary, then cut into chunks of a specific size for that `slab class`.

Once a page is assigned to a class, it is **never** moved. If your access patterns end up putting 80% of your pages in class 3, there will be less memory available for class 4. The best way to think about this is that memcached is actually many smaller individaul caches. Each class has its own set of statistical counters, and its own LRU.

Classes, sizes, and chunks are shown best by starting up memcached with `-vv`:

```
$ ./memcached -vv
slab class   1: chunk size        80 perslab   13107
slab class   2: chunk size       104 perslab   10082
slab class   3: chunk size       136 perslab    7710
slab class   4: chunk size       176 perslab    5957
slab class   5: chunk size       224 perslab    4681
slab class   6: chunk size       280 perslab    3744
slab class   7: chunk size       352 perslab    2978
slab class   8: chunk size       440 perslab    2383
slab class   9: chunk size       552 perslab    1899
slab class  10: chunk size       696 perslab    1506
[...etc...]
```

In slab class 1, each chunk is 80 bytes, and each page can then contain 13,107 chunks (or items). This continues all the way up to 1 megabyte.

When storing items, they are pushed into the slab class of the nearest fit. If your key + misc data + value is 50 bytes total, it will go into class 1, with an overhead loss of 30 bytes. If your data is 90 bytes total, it will go into class2, with an overhead of 14 bytes.

You can adjust the slab classes with `-f` and inspect them in various ways, but those're more advanced topics for when you need them. It's best to be aware of the basics because they can bite you.

## What Other Memory Is Used ##

Memcached uses chunks of memory for other functions as well. There is overhead in the hash table it uses to look up your items through. Each connection uses a few small buffers as well. This shouldn't add up to more than a few % extra memory over your specified `-m` limit, but keep in mind that it's there.

## When Memory Is Reclaimed ##

Memory for an item is not actively reclaimed. If you store an item and it expires, it sits in the LRU cache at its position until it falls to the end and is reused.

However, if you fetch an expired item, memcached will find the item, notice that it's expired, and free its memory. This gives you the common case of normal cache churn reusing its own memory.

Items can also be evicted to make way for new items that need to be stored, or expired items are discovered and their memory reused.

## How Much Memory Will an Item Use ##

An item will use space for the full length of its key, the internal datastructure for an item, and the length of the data.

You can discover how large an Item is by compiling memcached on your system, then running the "./sizes" utility which is built. On a 32bit system this may look like 32 bytes for items without CAS (server started with -C), and 40 bytes for items with CAS. 64bit systems will be a bit higher due to needing larger pointers. However you gain a lot more flexibility with the ability to put tons of ram into a 64bit box :)

```
$ ./sizes 
Slab Stats	56
Thread stats	176
Global stats	108
Settings	88
Item (no cas)	32
Item (cas)	40
Libevent thread	96
Connection	320
----------------------------------------
libevent thread cumulative	11472
Thread stats cumulative		11376
```

## When Are Items Evicted ##

Items are evicted if they have not expired (an expiration time of 0 or some time in the future), the slab class is completely out of free chunks, and there are no free pages to assign to a slab class.

## How the LRU Decides What to Evict ##

Memory is also reclaimed when it's time to store a new item. If there are no free chunks, and no free pages in the appropriate slab class, memcached will look at the end of the LRU for an item to "reclaim". It will search the last few items in the tail for one which has already been expired, and is thus free for reuse. If it cannot find an expired item however, it will "evict" one which has not yet expired. This is then noted in several statistical counters.

## libevent + Socket Scalability ##

Memcached uses [libevent](http://www.monkey.org/~provos/libevent/) for scalable sockets, allowing it to easily handle tens of thousands of connections. Each worker thread on memcached runs its own event loop and handles its own clients. They share the cache via some centralized locks, and spread out protocol processing.

This scales very well. Some issues may be seen with extremely high loads (200,00+ operations per second), but if you hit any limits please let us know, as they're usually solvable :)
