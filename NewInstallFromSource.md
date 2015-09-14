﻿#summary Using the Source, Luke



# Why Build From Source #

Before you build from source, consider why? If you have a perfectly good package of a recent version, you're better off using that.

# Building From Source #
## Prereqs ##
You'll likely need to install the development package for libevent
  * **Ubuntu:** `apt-get install libevent-dev`
  * **Redhat/Fedora:** `yum install libevent-devel`
## Get ##
```
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
```

## Config ##
### Optional install destination ###
If your compiling from source you likely want to specify a destination directory as well, replace `/usr/local/memcached` with whatever you fancy.
```
./configure --prefix=/usr/local/memcached
```
## Make and install ##
```
make && make test
sudo make install
```

If you wish to build with SASL support, ensure the cyrus-sasl libraries are built and run `./configure --enable-sasl`. See the [SASLHowto](SASLHowto.md) for more information.

# To Build a Package, or `make install` ? #

If you're deploying memcached to more than one server, you probably really want to package it. That way you may have cleaner updates, easy uninstalls, easy re-installs, future installs, etc. `make install` is for developers and chumps.

## Building an RPM ##

The memcached source tarball has contained a workable .spec file. To use it, create a build directory for RPM and compile memcached using the commands below. **Do not** run this as root, as tests will not pass.
```
echo "%_topdir /home/you/rpmbuild" >> ~/.rpmmacros
mkdir -p /home/you/rpmbuild/{SPECS,BUILD,SRPMS,RPMS,SOURCES}
wget http://memcached.org/latest
rpmbuild -ta memcached-1.x.x.tar.gz
```
You will need gcc and libevent-devel installed. (`yum install gcc libevent libevent-devel`)

Then install the RPM via a standard `rpm -Uvh memcached-etc.rpm`

## Building a deb ##

TODO: this section

# Building clients #

Note that many clients depend on [libmemcached](http://libmemcached.org). They either include it in their sources, or require an external build. You can follow the above practices for fetching and installing libmemcached as well.

## PEAR/CPAN/GEM/etc ##

If you're building from source, especially remember that most major languages have distribution systems which make installation easy.