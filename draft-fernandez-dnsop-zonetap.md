---
title: "A Discovery Mechanism for DNS Transparency Services"
abbrev: "Zonetap"
category: info

docname: draft-fernandez-dnsop-zonetap-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Domain Name System Operations"
keyword:
 - DNS
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "zluudg/draft-fernandez-dnsop-zonetap"
  latest: "https://zluudg.github.io/draft-fernandez-dnsop-zonetap/draft-fernandez-dnsop-zonetap.html"

author:
 -
    fullname: "Leon Fernandez"
    organization: The Swedish Internet Foundation
    email: "leon.fernandez@internetstiftelsen.se"

normative:

informative:

...

--- abstract

TODO Abstract


--- middle

# Introduction {#intro}

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Zone Transparency Service: A service that provides information about
    a zone such as what records have been added and deleted, if
    the zone has been transferred to a new registrar or if the
    registrant has changed.

Discoverer: Someone trying to discover a zone transparency service. This
    could be either a human that performing some kind of forensic
    investigation or troubleshooting a network issue or an automatic
    program that is looking for a connection point.


# The ZONETAP Resource Record

The ZONETAP RR is used to signal the availability of a zone transparency
service, how to connect to it and how to interpret the data it serves.
It MUST exist at the apex of a zone so a particular zone's transparency
service can be easily discovered by interested parties.

The RR Type code for the ZONETAP RR is TBD.


## The ZONETAP RDATA Format

The RDATA of the ZONETAP presentation format consists of three text
fields indicating what transport to use, the endpoint for the zone
transparency service and a what model is used for the data.

TODO some ASCII art showing the wire encoding.


### The TRANSPORT Field

The TRANSPORT field indicates how to fetch the zone data. This draft
defines the following values:

| Value | MNEMONIC | Description |
|-------|----------|-------------|
| 0     | NONE     | No zone transparency service available |
| 1     | XFR      | Zone transparency service available via an open zone transfer TODO-ref |
| 2     | HTTP     | Zone transparency service available via HTTP TODO-ref |

### The ENDPOINT Field

The ENDPOINT field shows where to connect to. For a record with an XFR
TRANSPORT field, the ENDPOINT field SHOULD be a DNS name (TODO-ref).
If it is not a valid DNS name, the discoverer MAY try to use it anyway, for
example by extracting the hostname part if the ENDPOINT value happens
to be a URL (TODO-ref).

For a record with a HTTP TRANSPORT field the value SHOULD be a URL
(TODO-ref). If it is not a valud URL, perhaps it is a plain IP address,
a discoverer MAY try to use that instead.

For a record with a TRANSPORT field set to NONE, the ENDPOINT value MUST
be set to a null byte.

### The MODEL Field

The MODEL field shows how the DNS transparency information that is being
shared is represented. For records with an XFR TRANSPORT field, the
model field MUST have one of the following values:

| Value | MNEMONIC | Description |
|-------|----------|-------------|
| 0     | AXFR     | Only full zone transfers are available |
| 1     | IXFR     | Incremental zone transfers are available (TODO-ref) |

For records with a TRANSPORT field set to HTTP, the MODEL field SHOULD
be a valid MIME type (TODO-ref). For example, it could be set to
"text/plain" for a transparency service that publishes zone data as
a text file. A discoverer MAY try to interpret values that are not valid MIME
types. For example, if the value is a URL that points to a JSON schema (TODO-ref),
the discoverer can try to use that as its data model.

For a record with a TRANSPORT field set to NONE, the MODEL value MUST
be set to a null byte.


## The ZONETAP Presentation Format

For a service that provides updates

```
example.    86400   IN  ZONETAP     XFR     zonetap.example.    IXFR
```


# Security Considerations {#security}

A zone transparency service SHOULD try to provide the data in an
authenticated manner. For services using the HTTP TRANSPORT, relying
on WebPKI (TODO-ref) is most likely sufficient. Services using XFR, can
signal the existence of XoT (TODO-ref) using an SVCB record similar to
{{!RFC9462}} and a correponding TLSA record {{!RFC6698}}:

```
_dns.zonetap.example.       IN  SVCB 1 zonetap.example. alpn=dot port=8530
_8530._tcp.zonetap.example. IN  TLSA 3 0 0 30820307308201efa003020102020...
```


# IANA Considerations {#iana}

This document has no IANA actions.


# TODO (unfinished thoughts)

Add another RDATA field if there is some kind of license related to
the zone data or "zone file"?

Add another RDATA field that indicates some kind of periodicity with
which the data is updated?


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
