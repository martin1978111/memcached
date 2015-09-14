﻿#summary Dandelions



Clients have a set of common features that they share. The intent here is to give you an overview of how typical clients behave, and some helpful features to look for.

This section does not describe which clients implement what, but merely what most do implement.

## Hashing ##

All clients should be able to hash keys across multiple servers.

## Consistent Hashing ##

Most clients have the ability to use consistent hashing, either natively or via an external library.

## Storing Binary Data or Strings ##

If passed a flat string or binary data, all clients should be able to store these via set/add/etc commands.

## Serialization of Data Structures ##

Most clients are able to accept complex data structures when passed in via set/add/etc commands. They are serialized (usually via some form of native system), a special flag is set, and then the data is stored.

Clients are **not** able to store all types of complex structures. Objects are usually not serializable, such as row objects returned from a mysql query. You must turn the data into a pure array, or hash/table type structure before being able to store or retrieve it.

Since item flags are used when storing the item, the same client is able to know whether or not to deserialize a value before returning it on a 'get'. You don't have to do anything special.

## Compression ##

Most clients are able to compress data being sent to or from a server. They set a special flag bit if data is over a certain size threshold, or it is specifically requested. Then compress the data and store it.

Since item flags are used, the clients will automatically know whether or not to decompress the value on return.

## Timeouts ##

Various timeouts, including timeouts while waiting for a connection to establish, or timeouts while waiting for a response.

## Mutations ##

Standard mutations as listed in [Commands](NewCommands.md)

## Get ##

Standard fetch commands as listed in [Commands](NewCommands.md)

## Multi-Get ##

Most clients implement a form of multi-get. Exactly how the multi-get is implemented will vary a bit.

Given a set of 10 keys you wish to fetch, if you have three servers, you may end up with 3-4 keys being fetched from each server.

  * Keys are first sorted into which servers they map onto
  * Gets are issued to each server with the list of keys for each. This attempts to be efficient.
  * Depending on the client, it might write to all servers in parallel, or it might contact one at a time and wait for responses before moving on.

Find that your client does the latter? Complain to the author ;)

# Less Common Features #

## Get-By-Group-Key ##

In the above case of Multi-Get, sometimes having your keys spread out among all servers doesn't make quite as much sense.

For example, we're building a list of keys to fetch to display a user's profile page. Their name, age, bio paragraph, IM contact info, etc.

If you have 50 memcached servers, issuing a Multi-Get will end up writing each key to individual servers.

However, you may choose to store data by an intermediate "key". This group key is used by the client to discover which server to store or retrieve the data.  Then any keys you supply are all sent to that same server. The group key is **not** retained on the server, or added to the existing key in any way.

So in a final case, we have 50 memcached servers. Keys pertaining to a single user's profile are stored under the group key, which is their userid. Issuing a Multi-Get for all of their data will end up sending a single command to a single server.

This isn't the best for everything. Sufficiently large requests may be more efficiently split among servers. If you're fetching fixed set of small values, it'll be more efficient on your network to send a few packets back and forth to a single server, instead of many small packets from many servers.

## Noreply/Quiet ##

Depending on if noreply is implemented via ascii or not, it may be difficult to troubleshoot errors, so be careful.

Noreply is used when you wish to issue mutations to a server but not sit around waiting for the response. This can help cut roundtrip wait times to other servers. You're able to blindly set items, or delete items, in cases where you don't really need to know if the command succeeds or not.

## Multi-Set ##

New in some binary protocol supporting clients. Multi-Set is an extension of the "quiet" mode noted above. With the binary protocol many commands may be packed together and issued in bulk. The server will respond only when the responses are interesting (such as failure conditions). This can end up saving a lot of time while updating many items.