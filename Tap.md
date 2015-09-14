# Memcached Tap #

## Overview ##

Tap provides a mechanism to observe from the outside data changes
going on within a memcached server.


## Use Cases ##

Tap is a building block for lots of new types of things that would
like to react to changes within a memcached server without having to
actually modify memcached itself.


### Replication ###

One simple use case is replication.

Upon initial connect, a client can ask for all existing data within a
server as well as to be notified as values change.

We receive all data related to each item that's being set, so just
replaying that data on another node makes replication an easy
exercise.


### Observation ###

Requesting a tap stream of only future changes makes it very easy to
see the types of things that are changing within your memcached
instance.


### Secondary Layer Cache Invalidation ###

If you have frontends that are performing their own cache, requesting
a tap stream of future changes is useful for invalidating items stored
within this cache.

Ideally, such a stream would not include the actual data that had
changed.  Today, all tap streams include full bodies, but specifying
new features that can be implemented by engines such as requesting the
omission of values is very straightforward.


### External Indexing ###

A tap stream pointed at an index server (e.g. sphinx or solr) will
send all data changes to the index allowing for an always-up-to-date
full-text search index of your data.


### vbucket transition ###

For the purposes of vbucket transfer between nodes, a new type of tap
request can be created that is every item stored in a vbucket (or set
of vbuckets) that is both existing and changing, but with the ability
to terminate the stream and cut-over ownership of the vbucket once the
last item is enqueued.


## Protocol ##

A tap session begins by initiating a command from the client which
tells the server what we're interested in receiving and then the
server begins sending /client/ commands back across the connection
until the connection is terminated.


### Initial Base Message from Client ###

A tap stream begins with a binary protocol message with the ID of
=0x40=.

The packet's key may specify a unique client identifier that can be
used to allow reconnects (resumable at the server's discretion).

A simple base message from a client referring to itself as "node1"
would appear as follows.

```
  Byte            0            1            2            3  
 ------+------------+------------+------------+------------
     0         0x80         0x40         0x00         0x05  
     4         0x00         0x00         0x00         0x00  
     8         0x00         0x00         0x00         0x05  
    12         0x00         0x00         0x00         0x00  
    16         0x00         0x00         0x00         0x00  
    20         0x00         0x00         0x00         0x00  
    24   0x6e ('n')   0x6f ('o')   0x64 ('d')   0x65 ('e')  
    28   0x31 ('1')                                         
```

```
Field        (offset) (value)
Magic        (0)    : 0x80
Opcode       (1)    : 0x40
Key length   (2,3)  : 0x0005
Extra length (4)    : 0x00
Data type    (5)    : 0x00
Reserved     (6,7)  : 0x0000
Total body   (8-11) : 0x00000005
Opaque       (12-15): 0x00000000
CAS          (16-23): 0x0000000000000000
Extras              : None
Key          (24-29): The textual string: "node1"
Value               : None
```


#### Options ####

Additional tap options may be specified as a 32-bit flags specifying
options.  The flags will appear in the "extras" section of the request
packet.  If omitted, it is assumed that all flags are 0.

Options may or may not have values.  For options that do, the values
will appear in the body in the order they're defined (LSB -> MSB).


##### Backfill #####

=BACKFILL= (=0x01=) contains a single 64-bit body that represents the
oldest entry (from epoch) you're interested in.  Specifying a time in
the future (for the server you are connecting to), will cause it to
start streaming only current changes.

An example tap stream request that specifies a backfill of -1 (meaning
future only) would look like this:

```
  Byte            0            1            2            3  
 ------+------------+------------+------------+------------
     0         0x80         0x40         0x00         0x05  
     4         0x00         0x00         0x00         0x00  
     8         0x00         0x00         0x00         0x0c  
    12         0x00         0x00         0x00         0x00  
    16         0x00         0x00         0x00         0x00  
    20         0x00         0x00         0x00         0x00  
    24         0x00         0x00         0x00         0x00  
    28         0x00         0x00         0x00         0x01  
    32   0x6e ('n')   0x6f ('o')   0x64 ('d')   0x65 ('e')  
    36   0x31 ('1')         0x00         0x00         0x00  
    40         0x00         0xff         0xff         0xff  
    44         0xff                                         
```

```
Field        (offset) (value)
Magic        (0)    : 0x80
Opcode       (1)    : 0x40
Key length   (2,3)  : 0x0005
Extra length (4)    : 0x08
Data type    (5)    : 0x00
Reserved     (6,7)  : 0x0000
Total body   (8-11) : 0x00000015
Opaque       (12-15): 0x00000000
CAS          (16-23): 0x0000000000000000
Extras       (24-31): 0x0000000000000001
Key          (32-36): The textual string: "node1"
Value               : 0x00000000ffffffff
```


##### Dump #####

=DUMP= (=0x02=) contains no extra body and will cause the server to
transmit only existing items and disconnect after all of the items
have been transmitted.

An example tap stream request that specifies only dumping existing
records would look like this:

```
  Byte            0            1            2            3  
 ------+------------+------------+------------+------------
     0         0x80         0x40         0x00         0x05  
     4         0x00         0x00         0x00         0x00  
     8         0x00         0x00         0x00         0x0c  
    12         0x00         0x00         0x00         0x00  
    16         0x00         0x00         0x00         0x00  
    20         0x00         0x00         0x00         0x00  
    24   0x6e ('n')   0x6f ('o')   0x64 ('d')   0x65 ('e')  
    28   0x31 ('1')         0x00         0x00         0x00  
    32         0x00         0xff         0xff         0xff  
    36         0xff                                         
```

```
Field        (offset) (value)
Magic        (0)    : 0x80
Opcode       (1)    : 0x40
Key length   (2,3)  : 0x0005
Extra length (4)    : 0x08
Data type    (5)    : 0x00
Reserved     (6,7)  : 0x0000
Total body   (8-11) : 0x0000000c
Opaque       (12-15): 0x00000000
CAS          (16-23): 0x0000000000000000
Extras       (24-31): 0x0000000000000002
Key          (32-36): The textual string: "node1"
Value               : None
```


##### VBucket List #####

=LIST\_BUCKETS= (=0x04=) is used to limit a request to a specific set
of vbuckets.

The vbuckets are included as values of 16-bits each, starting with a
16-bit number indicating the number of vbuckets in the list.

An example tap stream request that specifies vbuckets 1, 2, and 5
would look like this:

```
  Byte            0            1            2            3  
 ------+------------+------------+------------+------------
     0         0x80         0x40         0x00         0x05  
     4         0x00         0x00         0x00         0x00  
     8         0x00         0x00         0x00         0x0c  
    12         0x00         0x00         0x00         0x00  
    16         0x00         0x00         0x00         0x00  
    20         0x00         0x00         0x00         0x00  
    24   0x6e ('n')   0x6f ('o')   0x64 ('d')   0x65 ('e')  
    28   0x31 ('1')         0x00         0x00         0x00  
    32         0x00         0xff         0xff         0xff  
    36         0xff         0x00         0x03         0x00  
    40         0x01         0x00         0x02         0x00  
    44         0x05                                         
```

```
Field        (offset) (value)
Magic        (0)    : 0x80
Opcode       (1)    : 0x40
Key length   (2,3)  : 0x0005
Extra length (4)    : 0x08
Data type    (5)    : 0x00
Reserved     (6,7)  : 0x0000
Total body   (8-11) : 0x0000000c
Opaque       (12-15): 0x00000000
CAS          (16-23): 0x0000000000000000
Extras       (24-31): 0x0000000000000004
Key          (32-36): The textual string: "node1"
Value               : {3, 1, 2, 5} (16-bits each)
```


##### Takeover VBuckets #####

=TAKEOVER\_VBUCKETS= (=0x08=) is used to indicate that the client
wishes to completely take the given vbuckets away from the server.


##### Support ACK #####

=SUPPORT\_ACK= (=0x10=) indicates the client supports explicit ACKing.
See ACK support for more information on this mechanism.


##### Request Keys Only #####

=REQUEST\_KEYS\_ONLY= (=0x20=) does pretty much what you'd think.  It
requests that the server does not send the values along with the
keys.  The server is not required to understand this so the client
shouldn't assume it will be guaranteed to not receive values.


### Response Commands ###

After initiating tap, a series of responses will begin streaming
commands back to the caller.  These commands are similar to, but not
necessarily the same as existing commands.

In particular, each command includes a section of engine-specific data
as well as a TTL to avoid replication loops.


#### General Response Flags ####

The flag section of the packet has two defined flags that may be
present for any given packet:


##### ACK #####

=0x01= indicates the packet requests an ACK.  The client must reply to
this packet (i.e. send a packet back upstream with the same opaque) to
acknowledge receipt of this packet.


##### NO\_VALUE #####

=0x02= indicates the item may have a value, but it is not included in
the request.

[extended formats](describe.md)


#### Mutation ####

All mutation events arrive as =TAP\_MUTATION= (=0x41=) events.  These
are conceptualy similar to set commands.


#### Delete ####

# 0x42 #


#### Flush ####

# 0x43 #


#### Opaque ####

# 0x44 #

Engine specific extensions are sent as tap opaque messages.  A client
may ignore any of these it doesn't understand (though must still honor
the ACK if it the message has one).


#### VBucket Set ####

# 0x45 #

When doing a takeover, this message indicates a vbucket state
transition is ready.


## ACK Support ##

In default mode, all events are streamed out of the server effectively
blindly and as fast as possible.  With ACKs, a client can more
reliably receive messages by letting the server know at a higher
protocol level that it has successfully processed tap messages.

TCP, of course, guarantees message delivery, but a message can be
delivered to a remote system that can crash before processing it.
With ACKs enabled (and if supported by the server), the server can
retransmit any messages whose ACKs were pending at the time of a
connection drop if the same named connection reattaches to a tap
session that is still live.

The server chooses how frequently to send ACKs, and may dynamically
adjust the interval at which it sends ACK requests to achieve maximum
throughput.  Clients need not make any assumptions about message ACKs
other than any message requesting an ACK needs a response.
3

## Implementations ##


### Java ###

[spymemcached](http://code.google.com/p/spymemcached/) supports a pretty rich API in tap in recent versions

### Python ###

[ep-engine](https://github.com/membase/ep-engine) has a python implementation that's used in a lot of
utilities

### go ###

[gotap](https://github.com/dustin/gotap) is a working go implementation of tap, though has no real-world
deployments.

### C ###

[Trond's libcouchbase](https://github.com/trondn/libcouchbase) has support for tap.