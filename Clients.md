# C / C++ #

libmemcached
  * http://libmemcached.org/ by  [Brian Aker](http://krow.net/), Commercial Support available from [Data Differential](http://datadifferential.com/)
    * BSD license, it has been in production at websites for years. Aggressively optimised, ability to run async, supports binary protocol, triggers, replica, etc.

libmemcache
  * http://people.freebsd.org/~seanc/libmemcache by Sean Chittenden
    * BSD license. It is no longer under active development (last updated in 2006). You should try libmemcached instead.

apr\_memcache
  * http://www.outoforder.cc/projects/libs/apr_memcache by Paul Querna
    * Apache Software License version 2.0 (doesn't appear to be actively maintained since 2005)

memcacheclient
  * http://code.jellycan.com/memcacheclient (cross-platform, but primary focus on Windows (last updated in 2008).

libketama
  * http://www.last.fm/user/RJ/journal/2007/04/10/rz_libketama (the original consistent hashing algorithm from last.fm)

# PHP #

[Comparison of PECL/memcache and PECL/memcached](PHPClientComparison.md)

PECL/memcached
  * http://pecl.php.net/package/memcached (wraps libmemcached)
    * pear install pecl/memcached
    * Announcement: http://gravitonic.com/2009/01/new-memcached-extension

PECL/memcache
  * http://pecl.php.net/package/memcache
  * [php memcached docs](http://www.php.net/manual/en/book.memcache.php)

PHP libmemcached
  * http://github.com/kajidai/php-libmemcached/tree/master (wraps libmemcached)

# Java #

spymemcached
  * http://www.couchbase.org/code/couchbase/java
    * An improved Java API maintained by Matt Ingenthron and others at Couchbase.
    * Aggressively optimised, ability to run async, supports binary protocol, support Membase and Couchbase features, etc. See site for details.

Java memcached client
  * http://www.whalin.com/memcached
    * A Java API is maintained by Greg Whalin from Meetup.com.

More Java memcached clients
  * http://code.google.com/p/javamemcachedclient
  * http://code.google.com/p/memcache-client-forjava
  * http://code.google.com/p/xmemcached

Integrations
  * http://code.google.com/p/simple-spring-memcached
  * http://code.google.com/p/memcached-session-manager

# Python #

pylibmc - a libmemcached wrapper
  * http://sendapatch.se/projects/pylibmc/

python-memcached
  * http://www.tummy.com/Community/software/python-memcached/

pooling wrapper class
  * http://jehiah.cz/download/MemcachePool.py.txt for use in multi-threaded applications

Python libmemcached
  * http://code.google.com/p/python-libmemcached (libmemcached wrapper)

Python-Binary-Memcached - binprot pure-python client
  * https://github.com/jaysonsantos/python-binary-memcached

cmemcache (Note: this library is deprecated, old, buggy, you should **not** use it).
  * http://gijsbert.org/cmemcache/index.html

Django's caching framework works with memcached
  * http://docs.djangoproject.com/en/dev/topics/cache/

Twisted python client
  * http://python.net/crew/mwh/apidocs/twisted.protocols.memcache.html

# Ruby #

dalli
  * https://github.com/mperham/dalli (pure Ruby)

cache\_fu Rails plugin works with memcached
  * http://github.com/defunkt/cache_fu/tree/master
  * http://errtheblog.com/posts/57-kickin-ass-w-cachefu
  * http://blog.onmylist.com/articles/2007/06/15/memcached-and-cache_fu

memcache-client
  * http://dev.robotcoop.com/Libraries/memcache-client/index.html (pure Ruby)
  * http://seattlerb.rubyforge.org/memcache-client/
  * http://www.freshports.org/databases/rubygem-memcache-client

Ruby-MemCache
  * http://www.deveiate.org/projects/RMemCache (pure Ruby)

caffeine
  * http://rubyforge.org/projects/adocca-plugins (compiled, wraps libmemcached, no license)

More info:
  * [Memcached basics for rails](http://nubyonrails.com/articles/memcached-basics-for-rails)
  * [Rails and memcached while developing your app](http://zilkey.com/2008/7/5/rails-cache-memcached-development-mode-and-offline-cache-invalidation)

# Perl #

Cache::Memcached
  * http://search.cpan.org/dist/Cache-Memcached

Cache::Memcached::Fast
  * http://search.cpan.org/dist/Cache-Memcached-Fast

Perl libmemcached wrapper
  * http://code.google.com/p/perl-libmemcached (libmemcached wrapper)

Cache::Memcached-compatible perl libmemcached wrapper wrapper (heh)
  * http://search.cpan.org/dist/Cache-Memcached-libmemcached/

# Windows / .NET #

NMemcached.Client
  * http://nmemcached.codeplex.com/

.Net memcached client
  * https://sourceforge.net/projects/memcacheddotnet

.Net 2.0 memcached client
  * http://www.codeplex.com/EnyimMemcached
  * Client developed in .NET 2.0 keeping performance and extensibility in mind. (Supports consistent hashing.)
  * http://www.codeplex.com/memcachedproviders

BeIT Memcached Client (optimized C# 2.0)
  * http://code.google.com/p/beitmemcached

jehiah
  * http://jehiah.cz/projects/memcached-win32

# MySQL #

MySQL user data functions for memcached
  * https://launchpad.net/memcached-udfs

MySQL Engine
  * no longer developed

# PostgreSQL #

pgmemcache
  * http://pgfoundry.org/projects/pgmemcache The pgmemcache project allows you to access memcache servers from Postgresql Stored Procedures and Triggers.

# Erlang #

erlmc
  * http://github.com/JacobVorreuter/erlmc
  * http://jacobvorreuter.com/erlang-binary-protocol-memcached-client

merle
  * [merle, an erlang memcached client](http://github.com/joewilliams/merle/tree/master)

erlangmc
  * http://code.google.com/p/erlangmc

higepon's memcached client
  * http://github.com/higepon/memcached-client

Zhou Li's memcached client
  * http://github.com/echou/memcached-client


https://github.com/EchoTeam/mcd

# Lua #

http://luamemcached.luaforge.net

# Lisp dialects #

http://common-lisp.net/project/cl-memcached

http://chicken.wiki.br/memcached

http://weblambda.blogspot.com/2009/09/develop-memcached-client-4-bzlibdbd.html

# ColdFusion #

http://memcached.riaforge.org

# OCaml #

  * http://d.hatena.ne.jp/gtaka555/20080727/p1
  * http://tategakibunko.blog83.fc2.com/blog-entry-170.html

# Io #

http://github.com/iamaleksey/memcached-client-io/tree/master - libmemcached based

# CLI #

libmemcached
  * http://libmemcached.org/ by  [Brian Aker](http://krow.net/), Commercial Support available from [Data Differential](http://datadifferential.com/)
    * BSD licensed, contains a full set of CLI tools.

# Protocol #

To write a new client, check out the [binary protocol docs](MemcacheBinaryProtocol.md) and [ascii protocol docs](http://code.sixapart.com/svn/memcached/trunk/server/doc/protocol.txt). Be aware that the most important part of the client is the hashing across multiple servers, based on the key, or an optional caller-provided hashing value. Feel free to join the mailing list for help and/or a link to your client from this site.

# Archive / Old #

Danga Interactive list of clients http://www.danga.com/memcached/apis.bml

http://dealnews.com/developers/memcached.html - fastest client implementations (2006), obsoleted as more languages wrap the C-based libmemcached client library.