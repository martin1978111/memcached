# Memcached 1.4.9 Release Notes #

Date: 2011-10-18

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.9.tar.gz


## Overview ##

Small bugfix release. Mainly fixing a critical issue where using -c to
increase the connection limit was broken in 1.4.8. If you are on 1.4.8, an
upgrade is highly recommended.

## Fixes ##

  * Add a systemd service file
  * Fix some minor typos in the protocol doc
  * [Issue 224](https://code.google.com/p/memcached/issues/detail?id=224) - check retval of main event loop
  * Fix -c so maxconns can be raised above default.

## New Features ##

No new features in this version.

## Contributors ##

The following people contributed to this release since 1.4.8.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.8..1.4.9` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.9

```
     3	dormando
     1	Matt Ingenthron
     1	Miklos Vajna
```

## Control ##