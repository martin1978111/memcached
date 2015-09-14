# Introduction #

In order to use memcached in a hostile network (e.g. a cloudy ISP where the infrastructure is shared and you can't control it), you're going to want some kind of way to keep people from messing with your cache servers.

[SASL](http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) (as described in [RFC2222](http://tools.ietf.org/html/rfc2222)) is a standard for adding authentication mechanisms to protocols in a way that is protocol independent.

# Getting Started #

In order to deploy memcached with SASL, you'll need two things:

  1. A memcached server with SASL support (version 1.4.3 or greater built with `--enable-sasl`)
  1. A client that supports SASL

## Configuring SASL ##

For the most part, you just do the normal SASL admin stuff.

```
# Create a user for memcached.
saslpasswd2 -a memcached -c cacheuser
```

## Running Memcached ##

In order to enable SASL support in the server you must use the `-S` flag.

The `-S` flag does a few things things:

  1. Enable all of the SASL commands.
  1. Require binary protocol _only_.
  1. Require authentication to have been successful before commands may be issued on a connection.

## Further Info ##

Read more about memcached's [SASL auth protocol](SASLAuthProtocol.md).