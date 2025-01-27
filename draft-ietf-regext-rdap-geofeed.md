%%%
Title = "An RDAP Extension for Geofeed Data"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-geofeed"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-ietf-regext-rdap-geofeed-04"
stream = "IETF"
status = "standard"
date = 2023-04-08T00:00:00Z

[[author]]
initials="J."
surname="Singh"
fullname="Jasdip Singh"
organization="ARIN"
[author.address]
email = "jasdips@arin.net"

[[author]]
initials="T."
surname="Harrison"
fullname="Tom Harrison"
organization="APNIC"
[author.address]
email = "tomh@apnic.net"

%%%

.# Abstract

This document defines a new RDAP extension "geofeed1" for including a geofeed file URL in an IP Network object.

{mainmatter}

# Introduction

[@!RFC8805] and [@?I-D.ietf-opsawg-9092-update] (obsoletes [@!RFC9092]) detail the IP geolocation feed (in short,
geofeed) concept. This document specifies how the geofeed data can be accessed through RDAP. It defines a new RDAP
extension "geofeed1" for including a geofeed file URL in an IP Network object.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14
[@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate element relationships, and are not a REQUIRED
feature of this protocol.

"..." in examples is used as shorthand for elements defined outside of this document.

# Specification

## Extension

A new RDAP extension "geofeed1" is defined for accessing the geofeed data through RDAP. It updates the IP Network object
class definition ([@!RFC9083, section 5.4]) to include a new link object for the geofeed file URL in its "links" array
([@!RFC9083, section 4.2]).

An RDAP server conforming to this specification MUST include the "geofeed1" extension string in the "rdapConformance"
array for any lookup or search response containing an IP Network object, as well as in the help response. Here is an
elided example for this inclusion:

```
{
    "rdapConformance": [ "rdap_level_0", "geofeed1", ... ],
    ...
}
```

## Geofeed Link

The IP Network object class ([@!RFC9083, section 5.4]) MAY include a link object for the geofeed file URL (also referred
to as a Geofeed link object) in its "links" array ([@!RFC9083, section 4.2]). In RDAP, the "value", "rel", and "href"
JSON members are REQUIRED for any link object. Additionally, for a Geofeed link object, the "type" JSON member is
RECOMMENDED. Pertinent details of a Geofeed link object:

* "rel" -- The link relation type is set to the "geo" string. The "geo" link relation type is new and will be registered
  in the IANA Link Relations Registry (see (#link_relations_registry)).
* "href" -- The target URI is set to the HTTPS URL of the geofeed file for an IP network.
* "type" -- See (#media_type_for_a_geofeed_link) for acceptable "type" values.

There MAY be zero or more Geofeed link objects in the "links" array of an IP Network object. Further, more than one
Geofeed link object is allowed in case a geofeed file target needs to be represented in multiple formats and/or
languages.

## Media Type for a Geofeed Link {#media_type_for_a_geofeed_link}

[@?I-D.ietf-opsawg-9092-update] requires a geofeed file to be a UTF-8 CSV file, with a series of "#" comments at the end
for the optional RPKI (Resource Public Key Infrastructure) signature. At first glance, the "text/csv" media type
([@?I-D.shafranovich-rfc4180-bis, section 4]) seems like a good candidate to represent a geofeed file since it now
supports the "#" comments needed for including the RPKI signature.

However, although the CSV geofeed data could be directly viewed by a user, the most common use case will involve it
being processed by some sort of application first, in order to facilitate subsequent address lookup operations.
Therefore, using a new “application” media type with a “geofeed” subtype under the "application" top-level type
([@!RFC6838, section 4.2.5]) for the geofeed data is preferable over the existing "text/csv" media type.

To that end, a new media type "application/geofeed+csv" will be registered in the IANA Media Types Registry (see
(#media_types_registry)), and a new suffix "+csv" will be registered in the IANA Structured Syntax Suffixes
Registry (see (#structured_syntax_suffixes_registry)).

The "type" JSON value in a Geofeed link object SHOULD be set to the "application/geofeed+csv" media type.

A server that includes the "geofeed1" extension identifier in its responses is indicating that by way of its responses,
it will provide access to geofeed data at a target site, in accordance with the text of this document. However, this
document does not restrict the use of other media types that relate to geographic information, nor the use of link
objects more generally.

Although a server may use registered media types in its link objects without any restrictions, it may be useful to
define new RDAP extensions for those media types in order for the server to communicate to clients that it will make
data for that type accessible, in the same way that the server does with the "geofeed1" extension identifier.

## Example

The following is an elided example of an IP Network object with a Geofeed link object:

```
{
    "objectClassName": "ip network",
    "handle": "XXXX-RIR",
    "startAddress": "2001:db8::",
    "endAddress": "2001:db8:0:ffff:ffff:ffff:ffff:ffff",
    "ipVersion": "v6",
    "name": "NET-RTR-1",
    "type": "DIRECT ALLOCATION",
    "country": "AU",
    "parentHandle": "YYYY-RIR",
    "status": [ "active" ],
    "links":
     [
        {
            "value": "https://example.net/ip/2001:db8::/48",
            "rel": "self",
            "href": "https://example.net/ip/2001:db8::/48",
            "type": "application/rdap+json"
        },
        {
            "value": "https://example.net/ip/2001:db8::/48",
            "rel": "geo",
            "href": "https://example.com/geofeed",
            "type": "application/geofeed+csv"
        },
        ...
    ],
    ...
}
```

# Privacy Considerations

When including a geofeed file URL in an IP Network object, an RDAP server operator SHOULD follow the guidance from
[@?I-D.ietf-opsawg-9092-update, section 7] to not accidentally expose the location of an individual.

# Security Considerations {#security_considerations}

[@?I-D.ietf-opsawg-9092-update] requires an HTTPS URL for a geofeed file, and optionally RPKI-signing the data within.
Besides that, this document does not introduce any new security considerations past those already discussed in the
RDAP protocol specifications.

# IANA Considerations

## RDAP Extensions Registry

IANA is requested to register the following value in the RDAP Extensions Registry at
https://www.iana.org/assignments/rdap-extensions/:

* Extension identifier: geofeed1
* Registry operator: Any
* Published specification: This document.
* Contact: IETF <iesg@ietf.org>
* Intended usage: This extension describes version 1 of a method to access the IP geolocation feed data through RDAP.

## Link Relations Registry {#link_relations_registry}

IANA is requested to register the following value in the Link Relations Registry at
https://www.iana.org/assignments/link-relations/:

* Relation Name: geo
* Description: Indicates that the link context has a resource with geographic information at the link target.
* Reference: This document.

## Media Types Registry {#media_types_registry}

IANA is requested to register the following value in the Media Types Registry at
https://www.iana.org/assignments/media-types/:

* Type name: application
* Subtype name: geofeed+csv
* Required parameters: N/A
* Optional parameters: N/A
* Encoding considerations: See [@?I-D.ietf-opsawg-9092-update, section 2].
* Security considerations: See (#security_considerations) of this document.
* Interoperability considerations: There are no known interoperability problems regarding this media format.
* Published specification: This document.
* Applications that use this media type: Implementations of the Registration Data Access Protocol (RDAP) Extension for
  Geofeed Data. Furthermore, any application that processes the CSV geofeed data.
* Additional information: This media type is a product of the IETF REGEXT Working Group. The REGEXT charter, information
  on the REGEXT mailing list, and other documents produced by the REGEXT Working Group can be found at
  https://datatracker.ietf.org/wg/regext/.
* Person & email address to contact for further information: IETF <iesg@ietf.org>
* Intended usage: COMMON
* Restrictions on usage: None
* Authors: Tom Harrison, Jasdip Singh
* Change controller: IETF
* Provisional Registration: No

## Structured Syntax Suffixes Registry {#structured_syntax_suffixes_registry}

IANA is requested to register the following value in the Structured Syntax Suffixes Registry at
https://www.iana.org/assignments/media-type-structured-suffix/:

* Name: Comma-Separated Values (CSV)
* +suffix: +csv
* References: [@!RFC4180], [@!RFC7111]
* Encoding Considerations: Same as "text/csv".
* Interoperability Considerations: Same as "text/csv".
* Fragment Identifier Considerations:

  The syntax and semantics of fragment identifiers specified for +csv SHOULD be as specified for "text/csv".

  The syntax and semantics for fragment identifiers for a specific "xxx/yyy+csv" SHOULD be processed as follows:

  For cases defined in +csv, where the fragment identifier resolves per the +csv rules, then as specified in +csv.

  For cases defined in +csv, where the fragment identifier does not resolve per the +csv rules, then as specified in
  "xxx/yyy+csv".

  For cases not defined in +csv, then as specified in "xxx/yyy+csv".

* Security Considerations: Same as "text/csv".
* Contact: IETF <iesg@ietf.org>

# Acknowledgements

Mark Kosters provided initial support and encouragement for this work, along with the [@!RFC9092] authors. Gavin Brown
suggested using a web link instead of a simple URI string to specify a geofeed file URL. Andy Newton, James Gould, and
Scott Hollenbeck also provided valuable feedback for this document.

# Change History

## Changes from 00 to 01

* Now using a web link instead of a simple URI string to specify a geofeed file URL.
* Renamed the extension as "geofeed1" instead of "geofeedv1".
* Introduced the new "geo" link relation type.
* Introduced the new "application/geofeed+csv" media type.

## Changes from 01 to 02

* Updated the "Requirements Language" section for examples.
* Added an example for RDAP conformance.
* Updated the rationale for using the new "application/geofeed+csv" media type.
* Updated the "Applications that use this media type" section for the "application/geofeed+csv" registration.

## Changes from 02 to 03

* Removed "value" and "hreflang" explanations from the "Geofeed Link" section. Further, clarified the cardinality of
  Geofeed link objects.
* Updated extensibility verbiage in the "Media Type for a Geofeed Link" section.
* In the "Example" section, updated the domain in "href" of the Geofeed link object to contrast with the domain in
  "value" to highlight that "href" is for a geofeed file hosted at a network operator site whereas "value" is for an IP
  network object from an RDAP server.
* Removed the "Redaction" section since the geofeed files are public to start with.
* Added URLs for various IANA registries.

## Changes from 03 to 04

* Updated the criteria for including the extension identifier in "rdapConformance".

{backmatter}
