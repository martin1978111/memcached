

# Introduction #

Memcached is a high performance key-value cache. This document describes a
proposal for a third generation protocol. This proposal is to alleviate issues
of wasted space (binprot) and ease of use (asciiprot). It is **not** to be taken
as an authoritative document on how to currently access memcached.

As this is still currently in development, be sure to follow [the mailing list discussion](https://groups.google.com/d/topic/memcached/ZOLVZejXvCA/discussion).

## How to Read This Proposal ##

This proposal stands as an addendum to the binary protocol described in
[BinaryProtocolRevamped](BinaryProtocolRevamped.md). Many parts of the protocol, such as byte sizes,
signed/unsignedness, may be omitted from this document. The binary protocol
should be used as a reference for any information omitted from this document.

# Binary-V3 #

## Packet Structure ##
<pre>
Byte/     0       |       1       |       2       |       3       |<br>
/              |               |               |               |<br>
|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|<br>
+---------------+---------------+---------------+---------------+<br>
0/ HEADER                                                        /<br>
/                                                               /<br>
/                                                               /<br>
/                                                               /<br>
+---------------+---------------+---------------+---------------+<br>
H / COMMAND-SPECIFIC EXTRAS (as needed)                           /<br>
+/                                                               /<br>
+---------------+---------------+---------------+---------------+<br>
m/ Key (as needed)                                               /<br>
+/  (note length in key length header field)                     /<br>
+---------------+---------------+---------------+---------------+<br>
n/ Value (as needed)                                             /<br>
+/  (note length is total body length header field, minus        /<br>
+/   sum of the extras and key length body fields)               /<br>
+---------------+---------------+---------------+---------------+<br>
Total H + m + n bytes<br>
<br>
</pre>

## Request header ##
<pre>
Byte/     0       |       1       |       2       |       3       |<br>
/              |               |               |               |<br>
|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|<br>
+---------------+---------------+---------------+---------------+<br>
0-3| Magic byte    | Flag byte     | Opcode        |<br>
+---------------+---------------+---------------+---------------+<br>
4-5| Key length                    |<br>
+---------------+---------------+---------------+---------------+<br>
6-9| Total body length                                             |<br>
+---------------+---------------+---------------+---------------+<br>
10-11| Vbucket id                    |<br>
+---------------+---------------+---------------+---------------+<br>
12-15| Opaque                                                        |<br>
+---------------+---------------+---------------+---------------+<br>
...etc.<br>
Final byte order TBD<br>
</pre>

Required bytes:

  * Magic byte
  * Flag byte
  * Opcode byte
  * Two byte key length

All other headers are conditional on flags. Shortest possible "get" command
would be six bytes.

### Magic byte ###

Magic byte operates the same as binary protocol. New numbers are assigned for
"V3", request, response.

Also, **maybe** a third magic byte value meaning "unsolicited response"

### Flag byte ###

  * 0 extra: contains an extras field
  * 1 vbucket: contains a 2 byte vbucket field
  * 2 body: contains a 4 byte body length
  * 3 opaque: contains a 4 byte opaque
  * 4 CAS: contains an 8 byte CAS.
  * 5 quiet: suppress unimportant responses
  * 6 status: contains a status code

If any of these flags are unset, the associated data will not appear in the
header.

Final flags are reserved, but will stay **undefined** until a legitimate
usage is proposed.

Return headers are identical, except the "status" flag is set. **maybe**, not
sure if this is necessary.

### Opaque or Keys in return packets ###

If an opaque header is **not** set, commands are processed with their equivalent
binary K command (ie GETK) and the key is returned.

### Opcode byte ###

Opcode byte is identical to previous binprot, except "getq" variants are no
longer required.

### Extra Length ###

I'm tentatively omitted the "extras length" byte, as all (known to me) cases
of extras have a fixed size per-command. If they exist at all, they are of
such and so size. If this is wrong, please correct me :)


---


# Ascii-V3 #

new ASCII protocol must have feature parity with binprot. This allows two
parsers, but identical implementation. Server can continue to use the same
binary structs it's always used, but filled from ascii content.

I won't hear no bitching on this: It is not hard to support and makes it
leagues easier for simple clients to exist. With feature parity, clients can
choose one over the other (ie libmemcached only has to support binprot).

Only thing I'm not thrilled about is how "bodylen" is treated, as it's
potentially confusing. This is meant to act in a similar way to how binprot
counts bytes.

Also the "z" at the start is to instruct which parser is used. This isn't
ideal but I don't see what downside there is beyond ugliness and a wasted
byte under asciiprot.

## Request ##
```
zCOMMAND vBUCKET bLENGTH kLENGTH oOPAQUE cCAS q (EXTRAS)\r\n
(key in raw bytes)
(rest of body in raw bytes)
```
### Extras for Ascii ###

on commands which require extras, the one-char identifier is
context-dependent.
It **may** be reused as a different token type for another command, though that
is discouraged.

```
eEXPIRATION
fFLAGS
aAMOUNT (incr/decr)
iINITIALVAL (incr/decr)
lVERBOSITY (level)
```

### QUIET (noreply) ###

The singular 'q' flag sets command to quiet mode

### NOOP ###

In order to match feature parity with binprot, ASCII gains a NOOP command.

## Response ##

zCOMMAND sSTATUS oOPAQUE kLENGTH bLENGTH (EXTRAS)

## Example ##

```
zSET b6 k3 q\r\n
foobar
```
The above sets key "foo" to value "bar" (body is keylen + value len)

# Notes #

## Why not Avro/Buffers/JSON/etc ##

Apache Avro (or JSON or protocol buffers or whatever) look like wonderful ways
to serialize an application's data structures. To me, Avro has a ton of
potential in this area.

However, memcached has a very specific set of operations (key/value), with a
very common subset of headers (key, flags, expiration, etc). It is much
simpler for someone to write a client in any language with a succinct, focused
protocol description, than to dig up and work a second level of dependency
just to parse the protocol itself.

In this case, only JSON is pervasive enough to be considered an option for
being easily accessable by many languages. We're not writing a high speed
protocol on top of JSON, sorry.

This doesn't rule out that we could; the protocol could easily be described
with a JSON packet. It's still best to have options.

## Whyyyyyy another one ##

Adoption of binprot is difficult. For some users, it could inflate bandwidth
usage. For others, it may just be simpler to implement in a more _ascii_ way.

It's time to solve this damn thing.

## What about the old protocols? ##

The previous protocols can continue to exist for as long as is reasonable. New
features will be added only to the new protocol, but there shouldn't be any
great reason to hasten removal of the older protocols.