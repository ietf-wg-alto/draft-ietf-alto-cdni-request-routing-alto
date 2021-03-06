<!-- Skip header line -->

# IANA Considerations {#iana}

## application/alto-* Media Types

This document updates the IANA Media Types Registry by registering two
additional ALTO media types, listed in [](#TableMediaTypes).

| Type        | Subtype              | Specification        |
|-------------|----------------------|----------------------|
| application | alto-cdni+json       | [](#cdnifci)         |
| application | alto-cdnifilter+json | [](#filteredcdnifci) |
{: #TableMediaTypes title="Additional ALTO Media Types."}

{: newline="true"}
Type name:
: application

Subtype name:
: This document registers multiple subtypes, as listed in [](#TableMediaTypes).

Required parameters:
: n/a

Optional parameters:
: n/a

Encoding considerations:
: Encoding considerations are identical to those specified for the
  `application/json` media type. See {{RFC8259}}.

Security considerations:
: Security considerations related to the generation and consumption of ALTO
  Protocol messages are discussed in Section 15 of {{RFC7285}}.

Interoperability considerations:
: This document specifies formats of conforming messages and the interpretation
  thereof.

Published specification:
: This document is the specification for these media types; see
  [](#TableMediaTypes) for the section documenting each media type.

Applications that use this media type:
: ALTO servers and ALTO clients either stand alone or are embedded within other
  applications.

Additional information:
: Magic number(s):
  : n/a

  File extension(s):
  : This document uses the mime type to refer to protocol messages and thus does
  not require a file extension.

  Macintosh file type code(s):
  : n/a

Person &amp; email address to contact for further information:
: See Authors' Addresses section.

Intended usage:
: COMMON

Restrictions on usage:
: n/a

Author:
: See Authors' Addresses section.

Change controller:
: Internet Engineering Task Force (mailto:iesg@ietf.org).

## CDNI Metadata Footprint Type Registry

This document updates the CDNI Metadata Footprint Types Registry created by
Section 7.2 of {{RFC8006}}. A new footprint type is to be registered, listed in
[](#tbl:footprint-type).

| Footprint Type | Description         | Specification                     |
|----------------|---------------------|-----------------------------------|
| altopid        | A list of PID names | [](#cdnifcinetworkmap) of RFCthis |
{: #tbl:footprint-type title="CDNI Metadata Footprint Type"}

\[RFC Editor: Please replace RFCthis with the published RFC number for this
document.\]

## ALTO Entity Domain Type Registry

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

## ALTO Entity Property Type Registry

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

As an extension of the base ALTO protocol ({{RFC7285}}), this document fits into
the architecture of the base protocol. And hence Security Considerations of the
base protocol (Section 15 of {{RFC7285}}) fully apply when this extension is
provided by an ALTO server.

In the context of CDNI Advertisement, additional security considerations should
be included as follows:

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
be applied to address aforementioned security considerations, one additional
information leakage risk introduced by this document could not be addressed by
these strategies. In particular, if a dCDN signs agreements with multiple uCDNs
without any isolation, this dCDN may disclose extra information of one uCDN to
another one. In that case, one uCDN may redirect requests which should not have
to be served by this dCDN to it.

To reduce the risk, a dCDN should isolate full/filtered CDNI Advertisement
resources for different uCDNs. It could consider generating URIs of different
full/filtered CDNI Advertisement resources by hashing its company ID, a uCDN's
company ID as well as their agreements. A dCDN should avoid exposing all
full/filtered CDNI Advertisement resources in one of its IRDs.

# Acknowledgments {#ack}

The authors thank Matt Caulfield, Danny Alex Lachos Perez, Daryl Malas and
Sanjay Mishra for their timely reviews and invaluable comments.

Jan Seedorf has been partially supported by the GreenICN project (GreenICN:
Architecture and Applications of Green Information Centric Networking), a
research project supported jointly by the European Commission under its 7th
Framework Program (contract no. 608518) and the National Institute of
Information and Communications Technology (NICT) in Japan (contract no. 167).
The views and conclusions contained herein are those of the authors and should
not be interpreted as necessarily representing the official policies or
endorsements, either expressed or implied, of the GreenICN project, the European
Commission, or NICT.

This document has also been supported by the Coordination Support Action
entitled 'Supporting European Experts Presence in lnternational Standardisation
Activities in ICT' ("StandlCT.eu") funded by the European Commission under the
Horizon 2020 Programme with Grant Agreement no. 780439. The views and
conclusions contained herein are those of the authors and should not be
interpreted as necessarily representing the official policies or endorsements,
either expressed or implied, of the European Commission.

# Contributors

Mr. Xiao Shawn Lin is an author of an early version of this document, with many
contributions.
