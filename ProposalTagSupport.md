# Proposal #

  * Add\_Tag (key, tag\_name(s))
  * Invalidate\_Tag (tag\_name) - Not atomic

## General Synopsis ##
Invalidation of a tag would basically be a ``global\_generation = ++tags[tag](tag.md)'' kind of operation.

Each cache item contains a space for pointers to tags with their individual generation numbers and a local generation number.
Adding an existing tag to an item must not cause any modification to the item (i.e. check first).

Each time an item is requested from a cache, the local generation number is compared against the global generation number. If it differs, each tag is checked to ensure the tag generation number equals the number stored for that tag.
If they're all the same, the local generation number is set to the global generation number.

If they're different, this record doesn't exist.
Protocol should not be updated to support ‘tags’ under add/set/replace/cas.

## Allocations / Optimizations ##

  * Tag hash table
  * Tag array per item (supporting multiple tags per item)
  * Thread-lock wise?

  * Global version counter.
  * Tag counters
  * A single global generation number is used to track invalidation events.
  * Tag hash table in general - Biglock?

Structures in the tag hash should definitely be reusable in a free list, like most of the other structures. Having one or more per key could be massive suck if you're storing small items. Otherwise the goal should still be to avoid malloc/free if at all possible.
Presize the tag table?
Free list the tag name/version linked list?

Tag array per item?
8 bytes per item overhead - Counter & Pointer. Whether the pointer is a linked list or an array, the overhead is fixed for non-tag users.
Realloc the item header?
Tag support as experimental
Could probably be a config (not ./configure) option though, and avoid that memory overhead.

## Lock-free hashtable ##
The hashtable itself can at worst have a lock per bucket. There's an implementation of a lock-free hashtable in java that should be possible in C as well. Perhaps we could just leave this optimization to those who need it since the overlap between people who really need MT and people who really wants tags seems to be quite small so far.
Each tag will have a reference count that is increment every time the tag is successfully added to an item (i.e. must not increment if an item lookup fails or the item already has the tag).

When the item is deallocated, all tags should be decremented for that item.
When the tags reference count hits zero, we'll pull it from the global map.

Memory churn may be an issue. Someone who knows allocators better than I do can decide what to do here.
Increments are not atomic across SMP/multi-core.

  * Not only do you have issues with multicore, you have worse issues with SMP because of the costs around synchronizing the variables across the CPU transport (aka the bus between the CPU's).
  * When incrementing/changing values you need to wrap the write in a mutex if you want to be sure of the change.
  * Short of pinning all of the increments to a single CPU, you're just going to have to deal with synchronizing this state.
  * You don't need a mutex if you have CAS. Java's AtomicInteger is implemented using a volatile integer (volatile mostly means that it can't be stored in a CPU cache, but also is used to establish a happens-before relationship with other threads on reads and writes).

So, given a facility for cache line synchronization
and a CAS, I imagine you'll end up with a lot of code that looks like
this (from Java's AtomicInteger):
```
    public final int addAndGet(int delta) {
        for (;;) {
            int current = get();
            int next = current + delta;
            if (compareAndSet(current, next))
                return next;
        }
    }
```

glib has something similar as well. It's not guaranteed to be lock-free, but it can be done on a few platforms anyway.

  * Use APR's atomic apr\_atomic\_inc32 or Windows' interlocked InterlockedIncrement. These use CPU-specific instructions to perform the increment atomically with respect to other threads. No mutexes or other synchronization is required.

### Quick Notes By Dormando ###

  * libatomic\_ops for atomic operations.
  * must be fully switchable at runtime... if you switch off tag support, zero extra memory is allocated/used.