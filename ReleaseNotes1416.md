# Memcached 1.4.16 Release Notes #

Date: 2013-12-9

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.16.tar.gz


## Overview ##

A quick bugfix release to get the tree moving again after a long absence. I
don't want to make too many changes at once, so here are a number of platform
and crash fixes, as well as some introspection.

If you run 1.4.16 and experience any sort of memory leak or
segfault/crash/hang, **please** contact us. Please do the following:

- Build memcached 1.4.16 from the tarball.
- Run the "memcached-debug" binary that is generated at make time under a gdb
> instance
- Don't forget to ignore SIGPIPE in gdb: "handle SIGPIPE nostop"
- Grab a backtrace "thread apply all bt" if it crashes and post it to the
> mailing list or otherwise hunt me down.
- Grab "stats", "stats settings", "stats slabs", "stats items" from an
> instance that has been running for a while but hasn't crashed yet.

These crashes have been around too long and I would love to get rid of them
soon.

Thanks!

## Fixes ##

  * Builds on OS X Mavericks (with clang)
  * Add statistics for allocation failures
  * [Issue 294](https://code.google.com/p/memcached/issues/detail?id=294): Check for allocation failure
  * Make tail leak expiry time configurable (-o tail\_repair\_time=60)
  * Fix segfault on specially crafted packet.
  * Close connection on update\_event error while parsing new commands
  * Don't truncate maxbytes stat from 'stats settings'
  * Add the "shutdown" command to the server. This allows for better automation
  * fix enable-sasl-pwdb


## New Features ##

Adjusting tail repair time:
-o tail\_repair\_time=60 (in seconds)

"tail repairs" are a failsafe within memcached where if a cache item is leaked
via an unfixed or obscure bug, the item will be recycled anyway if it ends up
at the bottom of the LRU and hasn't been touched in a long period of time.
Most releases do not have these bugs, but some have so we've left the
mechanism in place. The default time before reaping is 3 hours.
For a busy site that sucks. we've lowered the default to one hour, which is much
longer than any object should ever take to download.

If you need dead items to be pulled more quickly, use this override. Make sure
you don't set it too low if you have clients which download items very slowly
(unlikely, but eh).

## Contributors ##

The following people contributed to this release since 1.4.15.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.15..1.4.16` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.16

```
     5	Trond Norbye
     4	dormando
     2	Brian Aker
     2	Eric McConville
     1	Gabriel A. Samfira
     1	Huzaifa Sidhpurwala
     1	Kenneth Steele
     1	Keyur
     1	Wing Lian
     1	liu bo

```

## Control ##