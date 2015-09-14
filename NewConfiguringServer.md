﻿#summary Square Peg Into Round Hole



# Commandline Arguments #

Memcached comes equipped with basic documentation about its commandline arguments. View `memcached -h` or `man memcached` for up to date documentation. The service strives to have mostly sensible defaults.

When setting up memcached for the first time, you will pay attention to `-m`, `-d`, and `-v`.

`-m` tells memcached how much RAM to use for item storage (in megabytes). Note carefully that this isn't a global memory limit, so memcached will use a few % more memory than you tell it to. Set this to safe values. Setting it to less than 48 megabytes does not work properly in 1.4.x and earlier. It will still use the memory.

`-d` tells memcached to daemonize. If you're running from an init script you may not be setting this. If you're using memcached for the first time, it might be educational to start the service **without** `-d` and watching it.

`-v` controls verbosity to STDOUT/STDERR. Multiple `-v`'s increase verbosity. A single one prints extra startup information, and multiple will print increasingly verbose information about requests hitting memcached. If you're curious to see if a test script is doing what you expect it to, running memcached in the foreground with a few verbose switches is a good idea.

Everything else comes with sensible defaults; you should alter these only if necessary.

# Init Scripts #

If you have installed memcached from your OS's package management system, odds are it already comes with an init script. They come with alternative methods to configure what startup options memcached receives. Such as via a /etc/sysconfig/memcached file. Make sure you check these before you run off editing init scripts or writing your own.

If you're building memcached yourself, the 'scripts/' directory in the source tarball contains several examples of init scripts.

# Multiple Instances #

Running multiple local instances of memcached is trivial. If you're maintaining a developer environment or a localhost test cluster, simply change the port it listens on, ie: `memcached -p 11212`.

There is an unmerged (as of this writing) set of example init scripts for managing multiple instances over at [bug 82](http://code.google.com/p/memcached/issues/detail?id=82). This will likely be merged (in some fashion) for 1.4.6.

# Networking #

By default memcached listens on TCP and UDP ports, both 11211. `-l` allows you to bind to specific interfaces or IP addresses. Memcached does not spend much, if any, effort in ensuring its defensibility from random internet connections. So you **must not** expose memcached directly to the internet, or otherwise any untrusted users. Using SASL authentication here helps, but should not be totally trusted.

## TCP ##

`-p` changes where it will listen for TCP connections. When changing the port via `-p`, the port for UDP will follow suit.

## UDP ##

`-U` modifies the UDP port, defaulting to on. UDP is useful for fetching or setting small items, not as useful for manipulating large items. Setting this to 0 will disable it, if you're worried.

## Unix Sockets ##

If you wish to restrict a daemon to be accessable by a single local user, or just don't wish to expose it via networking, a unix domain socket may be used. `-s <file>` is the parameter you're after. If enabling this, TCP/UDP will be disabled.

# Connection Limit #

By default the max number of concurrent connections is set to 1024. Configuring this correctly is important. Extra connections to memcached may hang while waiting for slots to free up. You may detect if your instance has been running out of connections by issuing a `stats` command and looking at "listen\_disabled\_num". That value should be zero or close to zero.

Memcached can scale with a large number of connections very simply. The amount of memory overhead per connection is low (even lower if the connection is idle), so don't sweat setting it very high.

Lets say you have 5 webservers, each running apache. Each apache process has a MaxClients setting of 12. This means that the maximum number of concurrent connections you may receive is 5 x 12 (60). Always leave a few extra slots open if you can, for administrative tasks, adding more webservers, crons/scripts/etc.

# Threading #

Threading is used to scale memcached across CPU's. Its model is by "worker threads", meaning that each thread makes itself available to process as much as possible. Since using libevent allows good scalability with concurrent connections, each thread is able to handle many clients.

This is different from some webservers, such as apache, which use one process or one thread per active client connection. Since memcached is highly efficient, low numbers of threads are fine. In webserver land, it means it's more like nginx than apache.

By default 4 threads are allocated. Unless you are running memcached extremely hard, you should not set this number to be any higher. Setting it to very large values (80+) will not make it run any faster.

# Inspecting Running Configuration #

```
$ echo "stats settings" | nc localhost 11211
STAT maxbytes 67108864
STAT maxconns 1024
STAT tcpport 11211
STAT udpport 11211
STAT inter NULL
STAT verbosity 0
STAT oldest 0
STAT evictions on
STAT domain_socket NULL
STAT umask 700
STAT growth_factor 1.25
STAT chunk_size 48
STAT num_threads 4
STAT stat_key_prefix :
STAT detail_enabled no
STAT reqs_per_event 20
STAT cas_enabled yes
STAT tcp_backlog 1024
STAT binding_protocol auto-negotiate
STAT auth_enabled_sasl no
STAT item_size_max 1048576
END
```

cool huh? Between 'stats' and 'stats settings', you can double check that what you're telling memcached to do is what it's actually trying to do.