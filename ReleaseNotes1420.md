# Memcached 1.4.20 Release Notes #

Date: 2014-5-11

## Download ##

Download Link:

http://www.memcached.org/files/memcached-1.4.20.tar.gz


## Overview ##

Just one tiny change to fix a regression causing threads to lock up and spin
max CPU.

1.4.18 and 1.4.19 were affected. 1.4.17 and earlier were not affected. If you
are on .18 or .19 an upgrade to 1.4.20 is strongly advised.

Thanks to commando.io for reporting the bug initially and putting up with me
being blind for a few weeks.

## Fixes ##

  * Fix a race condition causing new connections to appear closed, causing an inifinte loop.


## New Features ##

None, see 1.4.18 for new interesting features, or 1.4.19 for other useful
bugfixes.

## Contributors ##

The following people contributed to this release since 1.4.19.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.19..1.4.20` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.20

```
     1	dormando

```

## Control ##