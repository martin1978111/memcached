﻿#summary Installing Memcached Binaries



The version of memcached you install matters a lot for what support will be available to you. Ancient versions lack bugfixes, statistics, etc. Commands may be missing, and we may not be able to assist you if the software is too old. Try to have at least 1.4.4 or higher, if that's not too difficult ;)

# Dependencies #

Memcached is a C program, depending on a recent version of GCC and a recent version of [libevent](http://www.monkey.org/~provos/libevent/). The recommended method of installation is to first try your distribution's package manager. If the version it contains is too old, you may have to try installing from a backport, or by source.

# Installing From Your Distribution #

## Ubuntu & Debian ##
```
apt-get install memcached
```

You will also need libevent installed, and apt should fetch that for you.

Be warned that most versions of ubuntu and debian have very old versions of memcached packaged. We're working on improving this situation. If you are a Debian user, [Debian Backports](http://backports.org) may have a more recent version available.

## Redhat/Fedora ##
```
yum install memcached
```

Pretty easy eh? Sadly you're likely to pull an old version still.

## FreeBSD ##

```
portmaster databases/memcached
```

(or substitute whatever ports management tool you use)

# Installing Clients #

Some of the popular clients will likely be available in your distribution. Search with `apt` or `yum` and see what you can find!

## libmemcached ##

Most languages have one or two main clients which depend on [libmemcached](http://libmemcached.org). This is the standard C library for accessing memcached-speaking servers. Some clients will bundle a compatible version, and some will require it to be installed separately.

## PEAR/CPAN/GEM/etc ##

Don't forget to check the standard repositories for your preferred language. Installing a client might be a simple command or two.