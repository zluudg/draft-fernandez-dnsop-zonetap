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

The Central Zone Data Service (CZDS, TODO-ref) provides data that is
fundamental to understanding how malicious actors operate on the Internet.
However, it has a few drawbacks. Recent research shows that the frequency
with which the CZDS is updated is too low, which allows some attackers to
evade detection (TODO-ref). Furthermore, while CZDS is mandatory for generic
top-level domains (gTLD) to partake in, many country-code top-level
domains (ccTLD) lack a similar service. Lastly, the CZDS gives no
insight in how data changes within zones at the level where domains are
usually registered. For example, it enables researchers to see changes
within the .com zone (or its siblings), but not within its child zones
(i.e. example.com and its siblings).

This document proposes a mechanism that lets individual zones promote
an alternative zone transparency service that may go beyond what the
CZDS requires.


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


# Zone Transparency Discovery

A discoverer issues a query for example.com and sets the query type to
ZONETAP. Via the response, the discoverer discovers an service from
which it can download the zone contents as they look at that precise
moment. It can use that data to, for instance, look for A records that
contain IP addresses that are known to be part of a botnet as a proactive
measure.

# The ZONETAP Resource Record

The ZONETAP RR is used to signal the availability of a zone transparency
service, how to connect to it and how to interpret the data it serves.
It MUST exist at the apex of a zone so a particular zone's transparency
service can be easily discovered by interested parties.

The RR Type code for the ZONETAP RR is TBD.


## The ZONETAP RDATA Format

The RDATA of the ZONETAP presentation format consists of two text
that guides the discoverer to where it can find a zone transparency
service and what kind of content it can expect.

TODO some ASCII art showing the wire encoding.


### The SERVICE-ID Field

The SERVICE-ID field consists of a URI (TODO-ref) that tells the discoverer
where and how to fetch the zone data. This draft supports the following
schemes:

| Scheme | Description |
|--------|-------------|
| dns    | Zone transparency service available via an open zone transfer {{!RFC4501}} |
| http  | Zone transparency service available via HTTPS TODO-ref |
| https  | Zone transparency service available via HTTPS TODO-ref |

For SERVICE-ID field URIs with a "dns" scheme, the "dnstypeval" parameter
SHOULD have a value of AXFR or IXFR (or their corresponding numerical
values) to indicate that the zone is available via an open zone
transfer at the given "dnsauthority". Standard ports are assumed if not
explicitly part of the "dnsauthority". I.e. tcp port 853. The
"dnsclassval" SHOULD be "IN", the Internet class, or omitted, which is
equivalent.

For SERVICE-ID field URIs with a "http" or "https" scheme, field SHOULD
contain a URL from where the zone data can be fetched.

Furthermore, the SERVICE-ID field can be set to a period (hex value 0x2E),
to explicitly state that there is no zone transparency service available
for the zone in question.


### The FORMAT Field

The FORMAT field shows how the DNS transparency information that is being
shared is represented and its allowed values vary depending on the
scheme in the SERVICE-ID field. For records with a "dns" scheme, the
model field MUST have one of the following values:

| Value | MNEMONIC | Description |
|-------|----------|-------------|
| 1     | SIGNED   | A signed version of the zone is provided  |
| 2     | UNSIGNED | An unsigned version of the zone is provided |

For records with a "http" or "https" field, the FORMAT field SHOULD
be a valid MIME type (TODO-ref). For example, it could be set to
"text/plain" for a transparency service that publishes zone data as
a text file. A discoverer MAY try to interpret values that are not valid MIME
types. For example, if the value is a URL that points to a JSON schema (TODO-ref),
the discoverer can try to use that as its data model.

For a record with a SERVICE-ID field set to a single period to indicate
the non-existance of a zone transparency service, the FORMAT value can
be set to anything and discoverers SHOULD ignore it since it adds
no meaningful information to the fact that there is no zone transparency
service.


## The ZONETAP Presentation Format

For a service that exposes the signed zone data and supports incremental
zone transfers, the record would look like:

```
example.    86400   IN  ZONETAP     dns:zonetap.example.?TYPE=IXFR     SIGNED
```

For a service that exposes an unsigned version of the zone on a
non-standard portand only supports AXFR, the record would be:

```
example.    86400   IN  ZONETAP     dns:zonetap2.example.:20000?TYPE=AXFR     UNSIGNED
```

A service that consists of publishing a text file with a list of the
zone contents (i.e. a zone file as it often appears on disk), the
record would be:

```
example.    86400   IN  ZONETAP     https://zonetap3.example/example.zone     text/plain
```

A service that encodes the zone content and enriches it with, for instance,
WHOIS data according to a specific data model would look like:

```
example.    86400   IN  ZONETAP     https://zonetap4.example/example.json     https://schemas.example/zone-transparency.json
```

# For Zone Owners

Zone owners may wish to provide a zone transparency service for their
zone for a number of different reasons. For instance, it may offload
authoritative nameservers that are subject to zone walking, subdomain
enumeration via dictionary attacks or similar. It can also improve the
reputation of a zone by showing that it is not involved in malicious
activities such as fast-flux (TODO-ref) or, if the zone is an effective
TLD, domain-flux (TODO-ref).


# For DNS Providers

DNS providers, either dedicated ones or registrars who offer DNS services
to their registrants, could let their customers, i.e. the zone owners,
opt-in to a dedicated transparency service. If the zone owner opts in,
the provider can add the ZONETAP record to the zone and set up the
underlying infrastructure without the zone owner having to be aware of
any of the implementation details. Opting in could be as simple as
ticking a checkbox in a web GUI.


# For Registries

Registries, typically operators of TLD zones, can of course also offer
a transparency service. While a lot of malicious activity happens at
a level or two below the TLD level, fine-grained data from TLDs is still
valuable. It will, for instance, make it easier for security researchers
to detect newly registered domains or large batch registrations, both of
which are associated with malicious activity. It would also help provide
operational insights for the wider Internet community. For example, to
what extent best practices are followed when delegation records are being
updated or when KSK rollovers are performed.

Some ccTLDs already offer zone transparency services comparable to the
CZDS (for example, .se, .nu, .ee and .ch) but with no standardized way
of discovering them, they are at risk of being overlooked by security
researchers.


# Security Considerations {#security}

A zone transparency service SHOULD try to provide the data in an
authenticated manner. For services using the "https" scheme, relying
on WebPKI (TODO-ref) is most likely sufficient. Services using the "dns"
scheme, can publish a TLSA TLSA record {{!RFC6698}}. For instance, the
service:

```
example.    86400   IN  ZONETAP     dns:zonetap5.example.?TYPE=AXFR     UNSIGNED
```

Would publish the following TLSA record:

```
_853._tcp.zonetap.example. IN  TLSA 3 0 0 30820307308201efa003020102020...
```

Of course, even with authenticated access to a zone transparency service,
it can still "lie" about the zone contents. For example, it can omit
A records with low-reputation IP addresses. A ZONETAP record on its own
is not enough to guarantee that a zone is not involved in malicious
activities. But over time, if the transparency service does not disagree
with what is observed by other actors on the Internet, such as resolver
operators, then the service will eventually become trusted. That way,
DNS providers can distinguish themselves by having a reliable
transparency service (that its customers can opt-in to).


# IANA Considerations {#iana}

IANA is requested to update the "Resource Record (RR) TYPEs" registry
under the "Domain Name System (DNS) Parameters" registry group as follows:

Type: ZONETAP
Value: TBD
Meaning: Zone Transparency Service Discovery
Reference: (this document)


# TODO (unfinished thoughts)

Add another RDATA field if there is some kind of license related to
the zone data or "zone file"?

Add another RDATA field that indicates some kind of periodicity with
which the data is updated?

Worth having a "mailto" scheme?


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
