# Overview #

Most deployments of memcached today exist within trusted networks
where clients may freely connect to any server and the servers don't
discriminate against them.

There are cases, however, where memcached is deployed in untrusted
networks or where administrators would like to exercise a bit more
control over the clients that are connecting.

This page mostly exists to describe the protocol.  If you just want to use it, check out the [SASL howto](SASLHowto.md)

# Authentication Concepts #

Authentication is abstracted from the server using the Simple
Authentication and Security Layer.

Among other things, this provides administrators with consistent
credential management that is mostly independent from the services
that are authenticating clients.

# Protocol Definitions #

## Error Codes and Conditions ##

There are two status codes provided by the SASL protocol to enable
authentication:

### Unauthorized ###

If a message is returned with a status code of `0x20`, this is
considered an authentication or authorization failure.

This may be in response to an explicit authentication command
indicating the credentials were not accepted or the authorization was
otherwise not granted to access the server.

### Continue Authentication ###

Some SASL mechanisms require multiple messages to be sent between the
client and server. If a server responds to an authentication message
with a status code of `0x21`, this will indicate your client needs to do
more work to complete the authentication negotiation.

### Authentication Not Supported ###

If a server responds to an authentication request indicating the
command is unknown (status `0x81`), it likely doesn't support
authentication. It is generally acceptable for the client to consider
authentication successful when communicating to a server that doesn't
support authentication.

## Authentication Requests ##

### List Mechanisms ###

In order to negotiate authentication, a client may need to ask the
server what authentication mechanisms it supports.

A command `0x20` with no extras, key, or value will request a mechanism
list from the server. The mechanisms are returned as a
space-separated value.

### Authentication Request ###

To begin an authentication request, send a request with command `0x21`,
the requested mechanism as the key, and the initial authentication
data as the value if any is required for the chosen mechanism.

### Authentication Continuation ###

If the authentication request responded with a continuation request
(status `0x21`), the body will contain the data needed for computing the
next value in the authentication negotiation.

The next step's data will be transmitted similarly to the initial
step, but using command `0x22`. Note that this includes the mechanism
within the key as in the initial request.

# Error Reference #

| **Status Code** | **Meaning** |
|:----------------|:------------|
| 0x20            | Authentication required / Not Successful |
| 0x21            | Further authentication steps required. |


# Command Reference #

| **Command** | **Op Code** | **Key** | **Value** |
|:------------|:------------|:--------|:----------|
|  List Mechanisms | 0x20        | None    | None      |
| Start Authentication | 0x21        | Mechanism | Auth Data |
| Authentication Step | 0x22        | Mechanism | Auth Data |

# See Also #

[memcached SASL howto](SASLHowto.md)

http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer