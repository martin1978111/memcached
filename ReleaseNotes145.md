# Memcached 1.4.5 Release Notes #

Date: 2010-04-03

## Download ##

Download Link:

http://memcached.googlecode.com/files/memcached-1.4.5.tar.gz


## Overview ##

This is a maintenance release with some build fixes, doc fixes, and
one new stat.


## Fixes ##

  * Properly detect CPU alignment on ARM. [bug100](https://code.google.com/p/memcached/issues/detail?id=0)
  * Remove 1MB assertion. [bug 119](https://code.google.com/p/memcached/issues/detail?id=19)
  * More automake versions supported.
  * Compiler warning fixes for OpenBSD.
  * potential buffer overflow in vperror
  * Report errors opening pidfiles using vperror


## New Features ##


### New stat: reclaimed ###
This stat reports the number of times an entry was stored using memory
from an expired entry.


### sasl\_pwdb for more simple auth deployments ###

--enable-sasl-pwdb allows memcached to use it's own password file and
verify a plaintext password.

The file is specified with the environment variable
MEMCACHED\_SASL\_PWDB, and is a plain text file with the following
syntax:

```
username:password
```

Please note that you have to specify "mech\_list: plain" in your sasl
config file for this to work.

Ex:

```
   echo "mech_list: plain" > memcached.conf
   echo "myname:mypass" > /tmp/memcached-sasl-db
   export MEMCACHED_SASL_PWDB=/tmp/memcached-sasl-db
   export SASL_CONF_PATH=`pwd`/memcached.conf
   ./memcached -S -v
```

and you should be able to use your favorite memcached client with sasl
support to connect to the server.

(Please note that not all SASL implementations support
SASL\_CB\_GETCONF, so you may have to install the sasl config
(memcached.conf) to the systemwide location)


## Contributors ##

The following people contributed to this release since 1.4.4.

Note that this is based on who contributed changes, not how they were
done.  In many cases, a code snippet on the mailing list or a bug
report ended up as a commit with your name on it.

Note that this is just a summary of how many changes each person made
which doesn't necessarily reflect how significant each change was.
For details on what led up into a branch, either grab the git repo and
look at the output of `git log 1.4.4..1.4.5` or use a web view.

  * Repo list:  http://code.google.com/p/memcached/wiki/DevelopmentRepos
  * Web View: http://github.com/memcached/memcached/commits/1.4.5

```
    6  Trond Norbye
    3  Paul Lindner
    2  Dustin Sallings
    1  Brad Fitzpatrick
    1  Jørgen Austvik
```


## Control ##