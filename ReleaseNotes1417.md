# Memcached 1.4.17 Release Notes #

Date: 2013-12-20

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.17.tar.gz


## Overview ##

Another bugfix release along with some minor new features. Most notable is a
potential fix for a crash bug that has plagued the last few versions. If you
see crashes with memcached, **please** try this version and let us know if you
still see crashes.

The other notable bug is a SASL authentication bypass glitch. If a client
makes an invalid request with SASL credentials, it will initially fail.
However if you issue a second request with bad SASL credentials, it will
authenticate. This has now been fixed.

If you see crashes please try the following:

- Build memcached 1.4.17 from the tarball.
- Run the "memcached-debug" binary that is generated at make time under a gdb
> instance
- Don't forget to ignore SIGPIPE in gdb: "handle SIGPIPE nostop"
- Grab a backtrace "thread apply all bt" if it crashes and post it to the
> mailing list or otherwise hunt me down.
- Grab "stats", "stats settings", "stats slabs", "stats items" from an
> instance that has been running for a while but hasn't crashed yet.

... and send as much as you can to the mailing list. If the data is sensitive
to you, please contact dormando privately.

## Fixes ##

  * Fix potential segfault in incr/decr routine.
  * Fix potential unbounded key prints (leading to crashes in logging code)
  * Fix bug which allowed invalid SASL credentials to authenticate.
  * Fix udp mode when listening on ipv6 addresses.
  * Fix for incorrect length of initial value set via binary increment protocol.

## New Features ##

  * Add linux accept4() support. Removes one syscall for each new tcp
> > connection.
  * scripts/memcached-tool gets "settings" and "sizes" commands.
  * Add parameter (-F) to disable flush\_all. Useful if you never want to be
> > able to run a full cache flush on production instances.

## Contributors ##

The following people contributed to this release since 1.4.16.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.16..1.4.17` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.17

```
     6	dormando
     1	Adam Szkoda
     1	Alex Leone
     1	Andrey Niakhaichyk
     1	Daniel Pañeda
     1	Jeremy Sowden
     1	Simon Liu
     1	Tomas Kalibera
     1	theblop
     1	伊藤洋也

```

## Control ##