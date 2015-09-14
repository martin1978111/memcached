#summary Klingon

Memcached supports two main protocols; the classic ASCII, and the newer binary. It's simpler to write clients and debug problems via the ASCII protocol, but binary affords us many new abilities.

  * [Text Protocol](http://github.com/memcached/memcached/blob/master/doc/protocol.txt)
  * [Binary Protocol](BinaryProtocolRevamped.md)
    * [Slides on binary protocol ](http://www.slideshare.net/tmaesaka/memcached-binary-protocol-in-a-nutshell-presentation/) by Toru Maesaka (2008)

Further, there are sub protocols and proposals

  * [SASL Authentication](SASLAuthProtocol.md)
  * [Range operations](RangeOps.md) - Not to be supported in core, but defined for storage engines and compatible clients.