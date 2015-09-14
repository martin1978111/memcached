
```



Network Working Group                                        Norbye, Ed.
Internet-Draft                                     Sun Microsystems, INC
Intended status: Informational                              Maesaka, Ed.
Expires: June 21, 2009                                         mixi, INC
                                                       December 18, 2008


                       Memcache Engine Interface
                    draft-norbye-engine-interface-02

Status of this Memo

   This document is an Internet-Draft and is NOT offered in accordance
   with Section 10 of RFC 2026, and the author does not provide the IETF
   with any rights other than to publish as an Internet-Draft.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF), its areas, and its working groups.  Note that
   other groups may also distribute working documents as Internet-
   Drafts.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   The list of current Internet-Drafts can be accessed at
   http://www.ietf.org/ietf/1id-abstracts.txt.

   The list of Internet-Draft Shadow Directories can be accessed at
   http://www.ietf.org/shadow.html.

   This Internet-Draft will expire on June 21, 2009.

Abstract

   This memo describes the engine interface.














Norbye & Maesaka          Expires June 21, 2009                 [Page 1]

Internet-Draft          Memcache Engine Interface          December 2008


Table of Contents

   1.  Introduction . . . . . . . . . . . . . . . . . . . . . . . . .  3
     1.1.  Conventions Used In This Document  . . . . . . . . . . . .  3
   2.  Data structures  . . . . . . . . . . . . . . . . . . . . . . .  3
     2.1.  Error codes  . . . . . . . . . . . . . . . . . . . . . . .  3
     2.2.  Store operations . . . . . . . . . . . . . . . . . . . . .  3
     2.3.  rel_time_t . . . . . . . . . . . . . . . . . . . . . . . .  3
     2.4.  struct item  . . . . . . . . . . . . . . . . . . . . . . .  4
     2.5.  Interface structure  . . . . . . . . . . . . . . . . . . .  5
   3.  Create instance  . . . . . . . . . . . . . . . . . . . . . . .  5
   4.  Interface structure v. 1 . . . . . . . . . . . . . . . . . . .  5
     4.1.  interface  . . . . . . . . . . . . . . . . . . . . . . . .  7
     4.2.  get_info . . . . . . . . . . . . . . . . . . . . . . . . .  7
     4.3.  initialize . . . . . . . . . . . . . . . . . . . . . . . .  7
     4.4.  destroy  . . . . . . . . . . . . . . . . . . . . . . . . .  7
     4.5.  item_allocate  . . . . . . . . . . . . . . . . . . . . . .  7
     4.6.  item_delete  . . . . . . . . . . . . . . . . . . . . . . .  8
     4.7.  item_release . . . . . . . . . . . . . . . . . . . . . . .  8
     4.8.  get  . . . . . . . . . . . . . . . . . . . . . . . . . . .  8
     4.9.  get_stats  . . . . . . . . . . . . . . . . . . . . . . . .  9
     4.10. store  . . . . . . . . . . . . . . . . . . . . . . . . . . 10
     4.11. arithmetic . . . . . . . . . . . . . . . . . . . . . . . . 10
     4.12. flush  . . . . . . . . . . . . . . . . . . . . . . . . . . 11
     4.13. touch  . . . . . . . . . . . . . . . . . . . . . . . . . . 11
   5.  Asynchronous interface . . . . . . . . . . . . . . . . . . . . 11
   6.  Security Considerations  . . . . . . . . . . . . . . . . . . . 12
   7.  Normative References . . . . . . . . . . . . . . . . . . . . . 12
   Authors' Addresses . . . . . . . . . . . . . . . . . . . . . . . . 12






















Norbye & Maesaka          Expires June 21, 2009                 [Page 2]

Internet-Draft          Memcache Engine Interface          December 2008


1.  Introduction

1.1.  Conventions Used In This Document

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [KEYWORDS].


2.  Data structures

2.1.  Error codes

   The engine interface defines the enum ENGINE_ERROR_CODE with the
   following valid error messages:

   ENGINE_SUCCESS      The command executed successfully
   ENGINE_KEY_ENOENT   The key does not exists
   ENGINE_KEY_EEXISTS  The key already exists
   ENGINE_ENOMEM       Could not allocate memory
   ENGINE_NOT_STORED   The item was not stored
   ENGINE_EINVAL       Invalid arguments
   ENGINE_ENOTSUP      The engine does not support this
   ENGINE_EWOULDBLOCK  This would cause the engine to block
   ENGINE_E2BIG        The data is too big for the engine

2.2.  Store operations

   The engine interface defines the enum ENGINE_STORE_OPERATION with the
   following values:

   OPERATION_ADD       TODO: fillme!!
   OPERATION_SET
   OPERATION_REPLACE
   OPERATION_APPEND
   OPERATION_PREPEND
   OPERATION_CAS

2.3.  rel_time_t

   Items will only expire some time in the future, and the rel_time_t is
   a datatype to represent a time period relative to process startup
   time.

   rel_time_t

     typedef uint32_t rel_time_t;




Norbye & Maesaka          Expires June 21, 2009                 [Page 3]

Internet-Draft          Memcache Engine Interface          December 2008


2.4.  struct item

   Each "key-value" pair stored in memcached is represented with an item
   structure consisting of a fixed part and a variable part.  This might
   sound unnecessary complex, but adds extra flexibility for the various
   storage engines.

   The fixed part of the item structure consists of:

   struct item

   typedef struct {
     rel_time_t exptime;
     uint32_t nbytes;
     uint32_t flags;
     uint16_t nkey;
     uint16_t iflag;
   } item;

   exptime             When the item will expire (relative to process
                       startup)
   nbytes              The total size of the data (in bytes)
   flags               Flags associated with the item
   nkey                The total length of the key (in bytes)
   iflag               Item flags describing the content of the item
                       structure.  The upper 8 bits is reserved for the
                       storage engine to use

   The variable part of the item structure contains the following
   optional data.  Please note that the optional fields MUST be present
   in the specified order.

   struct item variable data

   cas_id          Optional, stored as uint64_t
   inline key      Optional, stored as n bytes containing the key
   pointer to key  Optional, stored as a void*
   inline data     Optional, stored as n bytes containing the data
   padding         Optional, stored as n bytes
   pointer to data Optional, Stored as a void*

   cas_id              Only present if iflag & ITEM_WITH_CAS
   inline key          Present if !(iflag & ITEM_KEY_PTR)
   pointer to key      Present if iflag & ITEM_KEY_PTR







Norbye & Maesaka          Expires June 21, 2009                 [Page 4]

Internet-Draft          Memcache Engine Interface          December 2008


   inline data         Present if !(iflag & ITEM_DATA_PTR)
   padding             Only present if !(iflag & ITEM_KEY_PTR) && (iflag
                       & ITEM_DATA_PTR)
   pointer to data     Present if iflag & ITEM_DATA_PTR

   The size of the padding is (sizeof(void*) - (nkey % sizeof(void*))

2.5.  Interface structure

   All communication with the storage engine will go through the engine
   interface structure.  This is defined as an extendible structure to
   allow future modifications and preserve binary backwards
   compatibility.

   struct engine_interface

   typedef struct engine_interface{
     uint64_t interface;
   } ENGINE_HANDLE;


3.  Create instance

   Each shared object providing an implementation of the storage
   interface must export the following function to allow memcached to
   create a handle to the storage interface:

   ENGINE_ERROR_CODE create_instance(uint64_t interface,
                                     ENGINE_HANDLE** handle);

   interface           The highest interface level the server supports
   handle              Where to store the interface handle


4.  Interface structure v. 1

   Engine interface version 1

   typedef struct engine_interface_v1 {
      struct engine_interface interface;
      const char* (*get_info)(ENGINE_HANDLE* handle);
      ENGINE_ERROR_CODE (*initialize)(ENGINE_HANDLE* handle,
                                      const char* config_str);
      void (*destroy)(ENGINE_HANDLE* handle);
      ENGINE_ERROR_CODE (*item_allocate)(ENGINE_HANDLE* handle,
                                         const void* cookie,
                                         item **item,
                                         const void* key,



Norbye & Maesaka          Expires June 21, 2009                 [Page 5]

Internet-Draft          Memcache Engine Interface          December 2008


                                         const size_t nkey,
                                         const size_t nbytes,
                                         const int flags,
                                         const rel_time_t exptime);
      ENGINE_ERROR_CODE (*item_delete)(ENGINE_HANDLE* handle,
                                       const void* cookie,
                                       item* item);
      void (*item_release)(ENGINE_HANDLE* handle, item* item);
      ENGINE_ERROR_CODE (*get)(ENGINE_HANDLE* handle,
                               const void* cookie,
                               item** item,
                               const void* key,
                               const int nkey);
      char *(*get_stats)(ENGINE_HANDLE* handle,
                         const char *stat_key,
                         int nkey,
                         uint32_t (*callback)(char *buf,
                                              const char *key,
                                              const uint16_t klen,
                                              const char *val,
                                              const uint32_t vlen,
                                              void *cookie),
                         const void *cookie,
                         int *buflen);
      ENGINE_ERROR_CODE (*store)(ENGINE_HANDLE* handle,
                                 const void *cookie,
                                 item* item,
                                 enum operation operation);
      ENGINE_ERROR_CODE (*arithmetic)(ENGINE_HANDLE* handle,
                                      const void* cookie,
                                      const void* key,
                                      const int nkey,
                                      const bool increment,
                                      const bool create,
                                      const uint64_t delta,
                                      const uint64_t initial,
                                      const rel_time_t exptime,
                                      uint64_t *cas,
                                      uint64_t *result);
      ENGINE_ERROR_CODE (*flush)(ENGINE_HANDLE* handle,
                                 const void* cookie, time_t when);
      ENGINE_ERROR_CODE (*touch)(ENGINE_HANDLE* handle,
                                 const void* cookie,
                                 item *item,
                                 const rel_time_t newtime);
   } ENGINE_HANDLE_V1;





Norbye & Maesaka          Expires June 21, 2009                 [Page 6]

Internet-Draft          Memcache Engine Interface          December 2008


4.1.  interface

   To preserve binary compatibility this member MUST be first in the
   structure.  An engine implementing this interface MUST set the
   interface member to the value 1.

4.2.  get_info

   Get information about this storage engine (to use in the version
   command).  The core server will NOT try to release the memory
   returned, so the engine SHOULD reuse this buffer to aviod memory
   leakage.

   const char* (*get_info)(ENGINE_HANDLE* handle);

   handle              Pointer to the engine

4.3.  initialize

   Called during initialization of memcached.  The engine should do all
   of its initalization in this routine.

   ENGINE_ERROR_CODE (*initialize)(ENGINE_HANDLE* handle,
                                   const char* config_str);

   handle              Pointer to the engine
   config_str          String containing configuration options

4.4.  destroy

   Called during shutdown of memcached.  The engine should release all
   resources and perform shutdown logic in this routine.

   void (*destroy)(ENGINE_HANDLE* handle);

   handle              Pointer to the engine

4.5.  item_allocate

   Allocate space to store an item.

   ENGINE_ERROR_CODE (*item_allocate)(ENGINE_HANDLE* handle,
                                      const void* cookie,
                                      item **item,
                                      const void* key,
                                      const size_t nkey,
                                      const size_t nbytes.
                                      const int flags,



Norbye & Maesaka          Expires June 21, 2009                 [Page 7]

Internet-Draft          Memcache Engine Interface          December 2008


                                      const rel_time_t exptime);

   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   item                Where to store the result
   key                 Pointer to the key
   nkey                The length of the key
   nbytes              The number of bytes in the data
   flags               The flags to store with the item
   exptime             When the item will expire


4.6.  item_delete

   Delete an item.

   ENGINE_ERROR_CODE (*item_delete)(ENGINE_HANDLE* handle,
                                    const void* cookie,
                                    item* item);

   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   item                The item to delete

4.7.  item_release

   Release an item.  The core server will no longer reference the
   object, so the engine may release or modify it.

   void (*item_release)(ENGINE_HANDLE* handle, item* item);

   handle              Pointer to the engine
   item                The item to release

4.8.  get

   Get an item identified by a key

   ENGINE_ERROR_CODE (*get)(ENGINE_HANDLE* handle,
                            const void* cookie,
                            item** item,
                            const void* key,
                            const int nkey);






Norbye & Maesaka          Expires June 21, 2009                 [Page 8]

Internet-Draft          Memcache Engine Interface          December 2008


   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   item                Where to store the result
   key                 Pointer to the key
   nkey                The length of the key

4.9.  get_stats

   Query the engine for statistics.  This function returns a pointer to
   a buffer with series of pipelined return packets.  Each packet
   represents a single row of stats.


   char *(*get_stats)(ENGINE_HANDLE* handle,
                      const char *stat_key,
                      int nkey,
                      uint32_t (*callback)(char *buf,
                                           const char *key,
                                           const uint16_t klen,
                                           const char *val,
                                           const uint32_t vlen,
                                           void *cookie),
                      const void *cookie,
                      int *buflen);

   handle              Pointer to the engine
   stat_key            Specifies what kind of stats to fetch
   nkey                The length of stat_key
   callback            Callback to use to create the result
   cookie              Cookie for the engine that MUST be provided as
                       the cookie in the callback to the server
   buflen              Size of the result that MUST be set in bytes

   An engine SHOULD NOT implement its own result serializer and instead
   use the provided callback function.  When allocating memory for the
   result buffer, make sure to take into account the 24 byte overhead
   per stat entry and the termination packet for the binary protocol.
   The termination packet is also 24 bytes.

   Details on the callback is as follows:


   uint32_t (*callback)(char *buf,
                        const char *key,
                        const uint16_t klen,
                        const char *val,
                        const uint32_t vlen,



Norbye & Maesaka          Expires June 21, 2009                 [Page 9]

Internet-Draft          Memcache Engine Interface          December 2008


                        void *cookie);

   buf                 Buffer to write to which the engine MUST allocate
   key                 Name of the statistic
   klen                The length of the key
   val                 The value of the statistic
   vlen                The length of the value
   cookie              Cookie returned by the engin

   The callback returns the number of bytes it had written to the
   buffer.

4.10.  store

   Store an item in the cache.

   ENGINE_ERROR_CODE (*store)(ENGINE_HANDLE* handle,
                              const void *cookie,
                              item* item,
                              enum operation operation);

   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   item                The item to store
   operation           ADD / SET / REPLACE

4.11.  arithmetic

   Perform arithmetic operation (increment / decrement) on the value of
   a key.

   ENGINE_ERROR_CODE (*arithmetic)(ENGINE_HANDLE* handle,
                                   const void* cookie,
                                   const void* key,
                                   const int nkey,
                                   const bool increment,
                                   const bool create,
                                   const uint64_t delta,
                                   const uint64_t initial,
                                   const rel_time_t exptime,
                                   uint64_t *cas,
                                   uint64_t *result);








Norbye & Maesaka          Expires June 21, 2009                [Page 10]

Internet-Draft          Memcache Engine Interface          December 2008


   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   key                 The key to operate on
   nkey                The length of the key
   increment           TRUE if increment, FALSE to decrement
   create              Set to TRUE if we should create nonexisting keys
   delta               The amount to increment / decrement
   initial             The initial value if we create the key
   exptime             When the item shall expire
   cas                 IN: the current CAS value, OUT: the resulting CAS
                       value
   result              OUT: The resulting value

4.12.  flush

   Flush all items from the cache

   ENGINE_ERROR_CODE (*flush)(ENGINE_HANDLE* handle,
                              const void *cookie,
                              time_t when);

   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   when                When to flush the cache (see the protocol spec)

4.13.  touch

   Set the last referenced time for an object

   ENGINE_ERROR_CODE (*touch)(ENGINE_HANDLE* handle,
                              const void* cookie,
                              item* item,
                              const rel_time_t newtime);

   handle              Pointer to the engine
   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   item                The item to update
   newtime             The new time to use for the item


5.  Asynchronous interface

   An engine should return ENGINE_EWOULDBLOCK if it cannot return
   immediately from the request.  It may then perform all the operations
   it may want to in another thread, and call:



Norbye & Maesaka          Expires June 21, 2009                [Page 11]

Internet-Draft          Memcache Engine Interface          December 2008


   void notify_io_complete(const void* cookie,
                           ENGINE_ERROR_CODE status);

   cookie              Cookie for the engine to use with
                       ENGINE_EWOULDBLOCK
   status              The status for the IO operation

   The memcached server will then retry the operation.


6.  Security Considerations

   Memcache has no authentication or security layers whatsoever.  It is
   RECOMMENDED that memcache be deployed strictly on closed, protected,
   back-end networks within a single data center, within a single
   cluster of servers, or even on a single host, providing shared
   caching for multiple applications.  Memcache MUST NOT be made
   available on a public network.


7.  Normative References

   [KEYWORDS]
              Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.


Authors' Addresses

   Trond Norbye (editor)
   Sun Microsystems, INC
   Haakon VII g. 7B
   Trondheim  NO-7485 Trondheim
   Norway

   Email: trond.norbye@sun.com


   Toru Maesaka (editor)
   mixi, INC
   Jingumae 2-34-17
   Shibuya  Tokyo 150-0001
   Japan

   Email: dev@torum.net






Norbye & Maesaka          Expires June 21, 2009                [Page 12]


```