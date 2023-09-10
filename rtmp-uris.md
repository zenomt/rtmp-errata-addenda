RTMP Family URI Schemes
=======================
Adobe’s Real-Time Messaging Protocol (RTMP) comprises a family of network
protocols for transporting streams of time-oriented video, audio, and data
messages over IP networks.  The URI schemes that are used to identify, locate,
and access streaming resources in the RTMP family follow the
[URI Generic Syntax of RFC 3986][URI], with constraints.  This memo describes
the syntax constraints and the access semantics for RTMP family URIs.

Copyright Notice
----------------
Copyright © 2023 Michael Thornburgh. All rights reserved.

    SPDX-License-Identifier: MIT

Background
----------
RTMP and its TCP-based transport (the RTMP Chunk Stream) are described in the
[RTMP Specification of December 2012][RTMP]. Errors, omissions, and
ambiguities in that specification are addressed in [RTMP Errata and Addenda][RTMP-Errata].
Coordinated interoperable enhancements to RTMP are described in
[Enhancing RTMP, FLV With Additional Video Codecs and HDR Support][E-RTMP].
Transporting RTMP messages over the
[Secure Real-Time Media Flow Protocol (RTMFP) [RFC7016]][RFC 7016]
is described in
[Adobe's RTMFP Profile for Flash Communication [RFC7425]][RFC 7425].

This memo does not address private or undocumented extensions to the RTMP
family for which interoperation is not desired by the extending parties.

The primary operation for dereferencing an RTMP family URI is to connect to
the indicated origin server and its abstract _target resource_. Neither the
connection itself nor the target resource have a "representation" or a "media
type" in the senses described in RFC 3986, nor is a "retrieval" performed by
connecting.

The target resource can have secondary resources, such as streams, shared
objects, and remote procedures, that can be accessed according their kinds
via the connection. For example, streams may be published or played, shared
objects updated or observed, and remote procedures called.

Terminology
-----------
The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to
be interpreted as described in BCP 14 \[[RFC2119][]\] \[[RFC8174][]\] when,
and only when, they appear in all capitals, as shown here.

The terms "client", "server", "origin server", "resource", "secondary resource",
"dereference", "representation", and "media type" in this document are to be
interpreted as used or described in [RFC 3986][URI].

Syntax
------
RTMP family URIs follow the [Generic Syntax of RFC 3986][URI], with constraints.
Their syntax is described here using the
[Augmented Backus-Naur Form (ABNF) \[RFC5234\]][RFC5234] rules from RFC 3986,
which are incorported by reference as though fully set forth here.  The `host`
rule is also reproduced here for clarity in the following sections.  Note
that the `rtmp-authority` rule defined here is compatible with the `authority`
rule, and `rtmp-userinfo` is compatible with the `userinfo` rule.

    rtmp-URI  = "rtmp://" rtmp-authority path-abempty [ "?" query ] [ "#" fragment ]

    rtmps-URI = "rtmps://" rtmp-authority path-abempty [ "?" query ] [ "#" fragment ]

    rtmfp-URI = "rtmfp:"
              / "rtmfp://" rtmp-authority path-abempty [ "?" query ] [ "#" fragment ]


    rtmp-authority   = [ rtmp-userinfo "@" ] host [ ":" port ]

    rtmp-userinfo    = rtmp-connect-arg *( ":" rtmp-connect-arg )

    rtmp-connect-arg = *( unreserved / pct-encoded / sub-delims )

    host = IP-literal / IPv4address / reg-name

Authority
---------
The `rtmp-authority` component, when present, identifies the location of an RTMP
origin server which governs a namespace of target and secondary resources.

Note: The bare RTMFP URI form "`rtmfp:`" does not have an `rtmp-authority`
component (or any other components).  All of the other URI forms do. The bare
"`rtmfp:`" URI is used in the APIs of some implementations to indicate
instantiation of an RTMFP client according to [RFC 7425][], but without
connecting to a server.  Such an instantiation might be used for pure
peer-to-peer communication.

### Host and Port
The `host` and optional `port` components identify the origin server in the
network. The `host` component of the `rtmp-authority` **MUST NOT** be empty.

For an `rtmp-URI`, the `host` and optional `port` identify a potential origin
server listening for TCP connections, over which RTMP messages are to be sent
using the RTMP Chunk Stream as described in the [RTMP Specification][RTMP].
If the `port` is empty or not given, TCP port 1935 is the default.

For an `rtmps-URI`, the `host` and optional `port` identify a potential origin
server listening for TCP connections and capable of establishing a [TLS][]
connection secured for [RTMP Chunk Stream communication][RTMP]. "Secured"
means a connection that is authenticated, confidential, and integrity-protected
in a manner acceptable to both the client and server.  If the `port` is empty
or not given, TCP port 443 is the default.

For an `rtmfp-URI` where the `rtmp-authority` is present, the `host` and optional
`port` identify one or more initial candidate addresses with which to initiate
an [RTMFP][RFC 7016] connection according to [RFC 7425][].  The UDP port for
the initial candidate addresses, if empty or not specified, is 1935.  If
`host` is a `reg-name`, the initial candidate address set **SHOULD** comprise
all IPv4 and IPv6 addresses to which `reg-name` resolves.

### Userinfo
When connecting to an RTMP server, the
[`connect` command](https://rtmp.veriskope.com/docs/spec/#7211connect) can
accept or expect additional arguments following the Command Object. The number
and types of the additional arguments, their interpretation, and whether they
are expected is implementation-specific and is the prerogative of the server.

Often the server expects one or more string arguments to the `connect` command
following the Command Object, for example a developer key, a user name and
password, or implementation-specific connection properties.

If the `rtmp-authority` contains an `rtmp-userinfo` component, and in the
absence of other arrangements between the client and server, the `rtmp-userinfo`
**SHOULD** be interpreted as a "`:`" (`COLON`) separated sequence of potentially
empty `rtmp-connect-arg` strings (as illustrated in the ABNF) to be sent as
string arguments to the `connect` command following the Command Object. Each
`rtmp-connect-arg` **SHOULD** be percent-decoded before being sent.

Clients **SHOULD** remove the `rtmp-userinfo` component from the URI when
sending it as the `tcUrl` property of the `connect` command’s Command Object
or in the
[Ancillary Data option of an RTMFP Endpoint Discriminator](https://rfc-editor.org/rfc/rfc7425#section-4.4.2.2).

Path and Query
--------------
The `path-abempty` and `query` components identify the abstract
_target resource_ within the origin server’s namespace. The meaning of "target
resource" is implementation-specific and is the prerogative of the origin
server.

Interoperability Note: Some client implementations incorrectly presume
the server’s interpretation of the `path-abempty` component of the URI,
specifically by presuming the number of path segments that identify the target
resource. These clients then incorrectly interpret the remaining path segments of the
original URI as identifying a secondary resource (such the name of a stream
to play or publish via the connection to the target resource).  This behavior
is not in keeping with the spirit of generic URIs (particularly that the
`authority` governs its namespace), is not interoperable, and is
**NOT RECOMMENDED**. Such secondary resources, when encoded in a URI, should
instead be identified by the fragment identifier as described in the next
section.

Fragment Identifier
-------------------
[Section 3.5 of RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html#section-3.5)
describes the `fragment` component of a generic URI as allowing "indirect
identification of a secondary resource by reference to a primary resource and
additional identifying information".

Since an RTMP connection to a target resource has neither a "representation"
nor a "media type", the semantics of the `fragment` component are unconstrained
according to that section.

Often an RTMP client connects to a server to perform a specific operation on
a specific secondary resource, such as to publish to or play from a named
stream.  In cases where the kind of secondary resource and the intended operation
are unambiguous, and in the absence of other arrangements between the client
and server, the secondary resource **MAY** be identified by the `fragment`
component. The `fragment` **SHOULD** be percent-decoded before being used in
secondary resource APIs (for example in a stream `play` command).

Clients **SHOULD** remove the `fragment` component from the URI when sending
it as the `tcUrl` property of the `connect` command’s Command Object or in the
[Ancillary Data option of an RTMFP Endpoint Discriminator](https://rfc-editor.org/rfc/rfc7425#section-4.4.2.2).

Examples
--------

    rtmp://server.example/three/segment/path
    rtmp://server.example/three/segment/path#BigBuckBunny
    rtmp://[2001:db8:1::2]:19350/something?else
    rtmps://server.example/something?else#CosmosLaundromat?aFlag
    rtmps://arg1:arg2::arg4@server.example:1943
    rtmps://name=Mike;pri=5;exi=3600:7c412a7b6e5892f9c@server.example/path#Caves
    rtmfp://:arg2@redirectors.example/two/segments?key=value&flag
    rtmfp://arg%3Aone:arg2@redirectors.example/something#BigBuck%42unny
    rtmfp:

IANA Considerations
-------------------
A future version of this memo will request the IANA to update the
[URI Scheme Registry][SCHEMES] for the following schemes.

### `rtmp`
This section will request an update to the `rtmp` provisional scheme registration.

    URI scheme name:  rtmp

    Status:  provisional

    URI scheme syntax:

      rtmp-URI  = "rtmp://" authority path-abempty [ "?" query ] [ "#" fragment ]

    URI scheme semantics: This provides location information for the RTMP
       server to which to connect, and identifies a target resource and
       an optional secondary resource in the namespace of the server.  The
       host component of the authority MUST NOT be empty.  The host and
       port components of the authority identify a potential origin server
       listening for TCP connections, over which RTMP messages are to be
       sent using the RTMP Chunk Stream as described in the RTMP Specification
       (as amended).  If port is empty or not given, TCP port 1935 is the
       default.

       See the RTMP Family URI Schemes memo for more specific information
       regarding the semantics of this URI scheme.

    Encoding considerations:  The userinfo, path-abempty, query, and fragment
       components represent textual data consisting of characters from
       the Universal Character Set. These components SHOULD be encoded
       according to Section 2.5 of RFC 3986.

    Applications/protocols that use this URI scheme name:  The Flash
       runtime (including Flash Player) from Adobe, communication servers
       such as Adobe Media Server, and interoperable clients and servers
       provided by other parties.

    Interoperability considerations:  This scheme requires use of RTMP
       and the RTMP Chunk Stream as defined by the RTMP Specification (as
       amended).

    Security considerations:  See Security Considerations sections in
       RTMP Errata and Addenda and in RTMP Family URI Schemes.

    Contact:  Michael Thornburgh, <zenomt@zenomt.com>.

    Author/Change controller:  Michael Thornburgh, <zenomt@zenomt.com>.

    References:
       Parmar, H., Ed. and M. Thornburgh, Ed., "Adobe’s Real Time Messaging
       Protocol" ("RTMP Specification"), December 2012,
       <https://rtmp.veriskope.com/docs/spec/>.

       Thornburgh, M., "RTMP Errata and Addenda", September 2023,
       <https://zenomt.github.io/rtmp-errata-addenda/>.

       Thornburgh, M., "RTMP Family URI Schemes", September 2023,
       <TBD>.

### `rtmps`
This section will request provisional registration of the `rtmps` scheme.

    URI scheme name:  rtmps

    Status:  provisional

    URI scheme syntax:

      rtmps-URI = "rtmps://" authority path-abempty [ "?" query ] [ "#" fragment ]

    URI scheme semantics: This provides location information for the secure
       RTMP server to which to connect, and identifies a target resource
       and an optional secondary resource in the namespace of the server.
       The host component of the authority MUST NOT be empty.  The host
       and optional port components of the authority identify a potential
       origin server listening for TCP connections and capable of establishing
       a TLS connection secured for RTMP Chunk Stream communication, over
       which RTMP messages are to be sent using RTMP Chunk Stream as
       described in the RTMP Specification (as amended).  If port is empty
       or not given, TCP port 443 is the default.

       See the RTMP Family URI Schemes memo for more specific information
       regarding the semantics of this URI scheme.

    Encoding considerations:  The userinfo, path-abempty, query, and fragment
       components represent textual data consisting of characters from
       the Universal Character Set. These components SHOULD be encoded
       according to Section 2.5 of RFC 3986.

    Applications/protocols that use this URI scheme name:  The Flash
       runtime (including Flash Player) from Adobe, communication servers
       such as Adobe Media Server, and interoperable clients and servers
       provided by other parties.

    Interoperability considerations:  This scheme requires use of RTMP
       and the RTMP Chunk Stream as defined by the RTMP Specification (as
       amended).

    Security considerations:  See Security Considerations sections in
       RTMP Errata and Addenda and in RTMP Family URI Schemes.

    Contact:  Michael Thornburgh, <zenomt@zenomt.com>.

    Author/Change controller:  Michael Thornburgh, <zenomt@zenomt.com>.

    References:
       Parmar, H., Ed. and M. Thornburgh, Ed., "Adobe’s Real Time Messaging
       Protocol" ("RTMP Specification"), December 2012,
       <https://rtmp.veriskope.com/docs/spec/>.

       Thornburgh, M., "RTMP Errata and Addenda", September 2023,
       <https://zenomt.github.io/rtmp-errata-addenda/>.

       Thornburgh, M., "RTMP Family URI Schemes", September 2023,
       <TBD>.

### `rtmfp`
This section will request an update to the `rtmfp` provisional scheme registration.

    URI scheme name:  rtmfp

    Status:  provisional

    URI scheme syntax:

      rtmfp-URI = "rtmfp:"
                / "rtmfp://" authority path-abempty [ "?" query ] [ "#" fragment ]

    URI scheme semantics:  The first form is used in the APIs of some
       implementations to indicate instantiation of an RTMFP client
       according to RFC 7425, but without connecting to a server.  Such
       an instantiation might be used for pure peer-to-peer communication.
   
       The second form provides location information for the server to
       which to connect, and identifies a target resource and an optional
       secondary resource in the namespace of the server.  Connections are
       made using RTMFP (RFC 7016) as described by RFC 7425.  The host
       component of the authority MUST NOT be empty.  If host is a reg-name,
       the initial candidate address set SHOULD comprise all IPv4 and IPv6
       addresses to which reg-name resolves.  The UDP port for the initial
       candidate addresses for the server, if not specified, is 1935.

       See the RTMP Family URI Schemes memo for more specific information
       regarding the semantics of this URI scheme.

    Encoding considerations:  The userinfo, path-abempty, query, and fragment
       components represent textual data consisting of characters from
       the Universal Character Set. These components SHOULD be encoded
       according to Section 2.5 of RFC 3986.

    Applications/protocols that use this URI scheme name:  The Flash
       runtime (including Flash Player) from Adobe, communication servers
       such as Adobe Media Server, and interoperable clients and servers
       provided by other parties, using RTMFP according to RFC 7425.

    Interoperability considerations:  This scheme requires use of RTMFP
       as defined by RFC 7016 in the manner described by RFC 7425.

    Security considerations:  See Security Considerations (Section 7) in
       RFC 7425, and Section 9 of RTMP Errata and Addenda.

    Contact:  Michael Thornburgh, <zenomt@zenomt.com>.

    Author/Change controller:  Michael Thornburgh, <zenomt@zenomt.com>.

    References:
       Thornburgh, M., "Adobe's Secure Real-Time Media Flow Protocol",
       RFC 7016, November 2013.

       Thornburgh, M., "Adobe's RTMFP Profile for Flash Communication", 
       RFC 7425, December 2014.

       Thornburgh, M., "RTMP Errata and Addenda", September 2023,
       <https://zenomt.github.io/rtmp-errata-addenda/>.

       Thornburgh, M., "RTMP Family URI Schemes", September 2023,
       <TBD>.

Security Considerations
-----------------------
Several. See the Security Considerations sections of [RFC 7016][],
[RFC 7425][], and [RTMP Errata and Addenda][RTMP-Errata].

Many of the
[security considerations of RFC 9110](https://www.rfc-editor.org/rfc/rfc9110#section-17)
also apply to RTMP.

Author’s Address
----------------
**Michael C. Thornburgh**<br/>
Santa Cruz, CA 95060-1950<br/>
United States of America<br/>
Email: [zenomt@zenomt.com](mailto:zenomt@zenomt.com)<br/>
URI: [https://zenomt.zenomt.com/card.ttl#me](https://zenomt.zenomt.com/card.ttl#me)<br/>

  [RTMP]:        https://rtmp.veriskope.com/docs/spec/
  [E-RTMP]:      https://github.com/veovera/enhanced-rtmp
  [RTMP-Errata]: https://zenomt.github.io/rtmp-errata-addenda/
  [RFC2119]:     https://www.rfc-editor.org/rfc/rfc2119.html
  [RFC5234]:     https://www.rfc-editor.org/rfc/rfc5234.html
  [RFC8174]:     https://www.rfc-editor.org/rfc/rfc8174.html
  [RFC 7016]:    https://www.rfc-editor.org/rfc/rfc7016.html
  [RFC 7425]:    https://www.rfc-editor.org/rfc/rfc7425.html
  [SCHEMES]:     https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml
  [TLS]:         https://www.rfc-editor.org/rfc/rfc8446.html
  [URI]:         https://www.rfc-editor.org/rfc/rfc3986.html
