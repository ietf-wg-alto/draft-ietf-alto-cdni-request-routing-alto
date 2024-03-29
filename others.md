<!-- Skip header line -->

# IANA Considerations {#iana}

This document defines two new media types: "application/alto-cdni+json", as
described in [](#iana-cdni), and "application/cdnifilter+json", as described in
[](#iana-cdnifilter). It also defines a new CDNI metadata footprint type
([](#iana-footprint-type)), two new ALTO entity domain types
([](#iana-entity-domain-type)), and a new ALTO entity property type
([](#iana-entity-prop-type)).

## application/alto-cdni+json Media Type {#iana-cdni}

{: newline="true"}
Type name:
: application

Subtype name:
: alto-cdni+json

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: Encoding considerations are identical to those specified for the
  "application/json" media type. See {{RFC8259}}.

Security considerations:
: Security considerations related to the generation and consumption of ALTO
  Protocol messages are discussed in Section 15 of {{RFC7285}}.

Interoperability considerations:
: N/A

Published specification:
: [](#cdnifci) of RFCthis

Applications that use this media type:
: ALTO servers and ALTO clients {{RFC7285}} either stand alone or are embedded within other
  applications that provides CDNI interfaces for uCDNs or dCDNs.

Fragment identifier considerations:
: N/A

Additional information:
: Magic number(s):
  : N/A

  File extension(s):
  : N/A

  Macintosh file type code(s):
  : N/A

Person &amp; email address to contact for further information:
: See Authors' Addresses section.

Intended usage:
: COMMON

Restrictions on usage:
: N/A

Author:
: See Authors' Addresses section.

Change controller:
: Internet Engineering Task Force (mailto:iesg@ietf.org).

## application/alto-cdnifilter+json Media Type {#iana-cdnifilter}

{: newline="true"}
Type name:
: application

Subtype name:
: alto-cdnifilter+json

Required parameters:
: N/A

Optional parameters:
: N/A

Encoding considerations:
: Encoding considerations are identical to those specified for the
  "application/json" media type. See {{RFC8259}}.

Security considerations:
: Security considerations related to the generation and consumption of ALTO
  Protocol messages are discussed in Section 15 of {{RFC7285}}.

Interoperability considerations:
: N/A

Published specification:
: [](#filteredcdnifci) of RFCthis

Applications that use this media type:
: ALTO servers and ALTO clients {{RFC7285}} either stand alone or are embedded
  within other applications that provides CDNI interfaces for uCDNs or dCDNs
  and supports CDNI capability-based filtering.

Fragment identifier considerations:
: N/A

Additional information:
: Magic number(s):
  : N/A

  File extension(s):
  : N/A

  Macintosh file type code(s):
  : N/A

Person &amp; email address to contact for further information:
: See Authors' Addresses section.

Intended usage:
: COMMON

Restrictions on usage:
: N/A

Author:
: See Authors' Addresses section.

Change controller:
: Internet Engineering Task Force (mailto:iesg@ietf.org).

## CDNI Metadata Footprint Type Registry {#iana-footprint-type}

This document updates the CDNI Metadata Footprint Types Registry created by
Section 7.2 of {{RFC8006}}. A new footprint type is to be registered, listed in
[](#tbl:footprint-type).

| Footprint Type | Description         | Specification                     |
|----------------|---------------------|-----------------------------------|
| altopid        | A list of PID names | [](#cdnifcinetworkmap) of RFCthis |
{: #tbl:footprint-type title="CDNI Metadata Footprint Type"}

\[RFC Editor: Please replace RFCthis with the published RFC number for this
document.\]

## ALTO Entity Domain Type Registry {#iana-entity-domain-type}

This document updates the ALTO Entity Domain Type Registry created by Section
11.2 of {{I-D.ietf-alto-unified-props-new}}. Two new entity domain types are to
be registered, listed in [](#tbl:entity-domain).

| Identifier  | Entity Address Encoding                   | Hierarchy &amp; Inheritance | Media Type of Defining Resource |
|-------------|-------------------------------------------|-----------------------------|---------------------------------|
| asn         | See [](#asn-entity-id) of RFCthis         | None                        | None                            |
| countrycode | See [](#countrycode-entity-id) of RFCthis | None                        | None                            |
{: #tbl:entity-domain title="Additional ALTO Entity Domain Types"}

\[RFC Editor: Please replace RFCthis with the published RFC number for this
document.\]

## ALTO Entity Property Type Registry {#iana-entity-prop-type}

This document updates the ALTO Entity Property Type Registry created by Section
11.3 of {{I-D.ietf-alto-unified-props-new}}. A new entity property type is to
be registered, listed in [](#tbl:prop-type-register).

| Identifier        | Intended Semantics                     | Media Type of Defining Resource |
|-------------------|----------------------------------------|---------------------------------|
| cdni-capabilities | [](#capabilitytoproperties) of RFCthis | application/alto-cdni+json      |
{: #tbl:prop-type-register title="Additional ALTO Entity Property Type"}

\[RFC Editor: Please replace RFCthis with the published RFC number for this
document.\]

# Security Considerations {#security}

As an extension of the base ALTO protocol {{RFC7285}}, this document fits into
the architecture of the base protocol. And hence Security Considerations of the
base protocol (Section 15 of {{RFC7285}}) fully apply when this extension is
provided by an ALTO server.

In the context of CDNI Advertisement, the following security considerations should
be considered:

* For authenticity and integrity of ALTO information, an attacker may disguise
  itself as an ALTO server for a dCDN, and provide false capabilities and
  footprints to a uCDN using the CDNI Advertisement service. Such false
  information may lead a uCDN to (1) select an incorrect dCDN to serve user
  requests, or (2) skip uCDNs in good conditions.
* For potential undesirable guidance from authenticated ALTO information, a dCDN
  can provide a uCDN with limited capabilities and smaller footprint coverage so
  that the dCDN can avoid transferring traffic for a uCDN which they should have
  to transfer.
* For confidentiality and privacy of ALTO information, footprint properties
  integrated with ALTO property maps may expose network location identifiers
  (e.g., IP addresses or fine-grained PIDs).
* For availability of ALTO services, an attacker may conduct service degradation
  attacks using services defined in this document to disable ALTO services of a
  network. It may request potentially large, full CDNI Advertisement resources
  from an ALTO server in a dCDN continuously, to consume the bandwidth resources
  of that ALTO server. It may also query filtered property map services with
  many smaller individual footprints, to consume the computation resources of
  the ALTO server.

Although protection strategies as described in Section 15 of {{RFC7285}} should
be applied to address aforementioned security and privacy considerations,
two special cases need to be included as follows:

- As required by section 7 of {{RFC8008}},

      "All protocols that implement these capabilities and footprint
      advertisement objects are REQUIRED to provide integrity and
      authentication services."

  Therefore, the uCDN (ALTO Client)
  MUST be authenticated to the dCDN (ALTO Server). And the dCDN (ALTO Server)
  MUST support HTTP Digest Authentication and MAY also support TLS mutual
  authentication. The authentication method will need to be negotiated out of
  band and is out of scope for this document, as is the approach for
  provisioning and managing these credentials.
- One specific information leakage risk introduced by this document could not
  be addressed by these strategies. In particular, if a dCDN signs agreements
  with multiple uCDNs without any isolation, this dCDN may disclose extra
  information of one uCDN to another one. In that case, one uCDN may redirect
  requests which should not have to be served by this dCDN to it.

  To reduce the risk, a dCDN SHOULD isolate full/filtered CDNI Advertisement
  resources for different uCDNs. It could consider generating URIs of different
  full/filtered CDNI Advertisement resources by hashing its company ID, a
  uCDN's company ID as well as their agreements. A dCDN SHOULD avoid exposing
  all full/filtered CDNI Advertisement resources in one of its IRDs.

