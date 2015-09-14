# Overview #

Several implementations of servers speaking the memcached protocol
have backends that allow for inexpensive operations over ranges of
keys.

The purpose of this document is to standardize the protocol for making
use of these operations.

# General Properties of Range Commands #

Commands that take ranges all follow the same pattern of key
representation.

Each range representation will have a minimum and maximum range as
well as flags to indicate whether each end is inclusive or exclusive.

Either end may be null indicating there's no limit on the side where
the null appears.

Keys should be treated as byte arrays for range comparisons.  The
implementation is not expected to perform any type of character
conversion or collation.

## Abstract Representation of Ranges ##

This document uses standard [interval notation](http://en.wikipedia.org/wiki/Interval_%28mathematics%29), but you can see the
possibilities listed here:

  * (null, null) :: Operate over all keys.
  * (null, "X") :: Operate over every key up to (not including) "X".
  * (null, "X"] :: Operate over every key up to (and including) "X".
  * ("F", null) :: Operate over every key from (not including) "F".
  * ("F", "X") :: Operate over every key from "F" to "X" (not including either).
  * ("F", "X"] :: Operate over every key from "F" to "X" (excluding "F", including "X").
  * ["F", "X") :: Operate over every key from "F" to "X" (including "F", excluding "X").
  * ["F", "X"] :: Operate over every key from "F" to "X" (including both).

## Binary Protocol Generalities ##

### Requests ###

In the binary protocol, we've got some extra space and can apply that
to defining ranges.

The "key" field will be used as the start key.  A key length of zero
represents null as an empty key can't be stored.

The extra info will contain the following info:

| **Fields**    | **Size** |
|:--------------|:---------|
| End key len   |     16   |
| Reserved      |      8   |
| Flags         |      8   |
| Max results   |     32   |

Flags is a bitmask indicating whether the ends are inclusive or
exclusive.

  * (flags & 1) :: If non-zero, start key is inclusive.
  * (flags & 2) :: If non-zero, end key is inclusive.

Max results represents the number of objects the operation is
permitted to affect.  A value of zero indicates there should be no
limit.

A server **may** enforce a maximum limit and respond with an error if
the requested limit exceeds the limit enforced by the server (or is
zero).

The end key immediately follows the maximum results.

### Responses ###

Each requests will generally have many responses and will return
similarly to the way stats works in the binary protocol.  The final
response will be one with an empty key.

The binary protocol allows for commands to operate in a "quiet mode"
where most responses are quelled.  No such facility is available for
the text protocol as it can't be done consistently without introducing
new verbs (and then can't be done safely).

## Text Protocol Generalities ##

The general form of a range command is as follows (please see details
for how this varies by command):

```
CMD <start inclusion> <end inclusion> <max items> <start key> [end key]\r\n
```

The end key is optional.  If not specified, it is considered null.
Null is not explicitly included for the start key as there is a
natural minimum of keys which can be used in its place.

The inclusion flags are either 0 or 1.  If 0, the endpoint is not
included.  If 1, the endpoint is included.

This _is_ different from the rget as specified in memcachedb to allow
for infinite ranges.

# Supported commands #

## get ##

Ranged get operations allow multiple key/value pairs returned for a
single key request.

### Binary Protocol Details ###

The command ID for binary get is 0x30.

### Text Protocol Details ###

The command for text get is rget.

## set ##

Ranged set operations allow for a range of **existing** items to be set
to a specific value.

For example, if you have stats stored in a server with keys prefixed
as "stats." you can reset all of them in a single operation by setting
("stats.", "stats/") to "0".  This range specifies everything that
begins with "stats." excluding "stats." itself.

Upon batch mutation, each modified value will receive a new distinct
CAS identifier.

### Binary Protocol Details ###

The command ID for binary set is 0x31.  The command ID for a binary
quiet get is 0x32.

Each modified value will respond as a plain set does although the key
will be included in the response.  The final response will contain no
key.

In the case of a quiet request, no response will be given unless the
command fails.

### Text Protocol Details ###

The command for text set is rset and has the following form:

```
rset <start inclusion> <end inclusion> <max items> <flags> <exptime> <bytes> <start key> [end key]\r\n
<data>\r\n
```

The response will be similar to a gets, but with no values included.
Example:

```
VALUE <key> <flags> 0 [<cas>]\r\n
\r\n
```

## append ##

Ranged append will append a value to every item within a range.

For both protocols, the response will be in the form of their
respective protocol-specific response in ranged set.

### Binary Protocol Details ###

The command ID for binary range append is 0x33.  The command ID for a
silent append is 0x34.

### Text Protocol Details ###

The command for text append has the following orm:

```
rappend <start inclusion> <end inclusion> <max items> <bytes> <start key> [end key]\r\n
<data>\r\n
```

## prepend ##

Prepend follows the same pattern of append.

### Binary Protocol Details ###

The command ID for a binary range prepend is 0x35.  The command ID for
a silent prepend is 0x36.

### Text Protocol Details ###

The command for a text range prepend is "rprepend".

## delete ##

Ranged delete will remove all values in the specified range.

For both protocols, the response will be in the form of their
respective protocol-specific response in ranged set.

### Binary Protocol Details ###

The command ID for a binary range delete is 0x37.  The command ID for
a silent delete is 0x38.

### Text Protocol Details ###

The text protocol follows the common form of a range command, but uses
the command "rdelete".

## incr ##

Ranged incr will increment all existing keys within a range by the
same amount.  This follows the same patterns as existing increment
with the obvious exception that it will not initialize keys since it's
only modifing things that already exist.

### Binary Protocol Details ###

The command ID for binary range increment is 0x39.  The command ID for
a silent increment is 0x3a.

The extra data includes the standard incr/decr extra data **after** the
ranged extra data.

The responses will include a key and value for every modified item.

### Text Protocol Details ###

The text command for range increment is "rincr" and is in the
following form:

```
rincr <start inclusion> <end inclusion> <max items> <value> <start key> [end key]\r\n
```

Responses are the same as a multi-gets.

## decr ##

Ranged decr follows ranged incr.

### Binary Protocol Details ###

The command ID for binary range decrement is 0x3b.  The command ID for
a silent range decrement is 0x3c.

### Text Protocol Details ###

The command for range decrement is "rdecr".

## replace (omitted) ##

Ranged replace is explicitly omitted as it it would mean the same
thing as a set.

# Command Reference #

The following table is a quick reference to all of the commands
defined herein.  Commands omitted from the text column are intentional
as they are not defined outside of the binary protocol.

| **Command**     | **Bin CMD ID** | **Text Command** |
|:----------------|:---------------|:-----------------|
| Get             |         0x30   | rget             |
| Set             |         0x31   | rset             |
| Quiet Set       |         0x32   |                  |
| Append          |         0x33   | rappend          |
| Quiet Append    |         0x34   |                  |
| Prepend         |         0x35   | rprepend         |
| Quiet Prepend   |         0x36   |                  |
| Delete          |         0x37   | rdelete          |
| Quiet Delete    |         0x38   |                  |
| Incr            |         0x39   | rincr            |
| Quiet Incr      |         0x3a   |                  |
| Decr            |         0x3b   | rdecr            |
| Quiet Decr      |         0x3c   |                  |

# Isolation #

Isolation levels are specifically left unspecified.  Operations
concurrently working on the same data set may interleave, mutually
exclude each other, or just stomp all over each other.

Implementors are encouraged to document their intentions.