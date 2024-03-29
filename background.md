<!-- Skip header line -->

# Terminology and Background {#background}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

The design of CDNI FCI transport using ALTO assumes an understanding of both
FCI semantics and ALTO. Hence, this document starts with a non-normative review
for both.

## Terminology {#term}

The document uses the CDNI terms defined in {{RFC6707}}, {{RFC8006}} and
{{RFC8008}}. Also, the document uses the ALTO terms defined in {{RFC7285}} and
{{I-D.ietf-alto-unified-props-new}}. This document uses the following
abbreviations:

- ALTO: Application-Layer Traffic Optimization
- ASN: Autonomous System Number
- CDN: Content Delivery Network
- CDNI: CDN Interconnection
- dCDN: Downstream CDN
- FCI: CDNI FCI, CDNI Request Routing Footprint & Capabilities Advertisement interface
- IRD: Information Resource Directory in ALTO
- PID: Provider-defined Identifier in ALTO
- uCDN: Upstream CDN

## Semantics of FCI Advertisement {#bgSemantics}

{{RFC8008}} defines the semantics
of CDNI FCI, provides guidance on what Footprint and Capabilities mean in a CDNI
context, and specifies the requirements on the CDNI FCI transport protocol. The
definitions in {{RFC8008}} depend on {{RFC8006}}. Below is a non-normative
review of key related points of {{RFC8008}} and {{RFC8006}}. For detailed
information and normative specification, the reader should refer to these two
RFCs.

* Multiple types of mandatory-to-implement footprints (i.e., ipv4cidr, ipv6cidr, asn,
  and countrycode) are defined in {{RFC8006}}. A "Set of IP-prefixes" can
  contain both full IP addresses (i.e., a /32 for IPv4 or a /128 for IPv6) and
  IP prefixes with an arbitrary prefix length. There must also be support for
  multiple IP address versions, i.e., IPv4 and IPv6, in such a footprint.
* Multiple initial types of capabilities are defined in {{RFC8008}} including
  (1) Delivery Protocol, (2) Acquisition Protocol, (3) Redirection Mode, (4)
  Capabilities related to CDNI Logging, and (5) Capabilities related to CDNI
  Metadata. They are required in all cases and, therefore, considered as
  mandatory-to-implement capabilities for all CDNI FCI implementations.
* Footprint and capabilities are defined together and cannot be interpreted
  independently from each other. Specifically, {{RFC8008}} integrates footprint
  and capabilities with an approach of "capabilities with footprint
  restrictions", by expressing capabilities on a per footprint basis.
* Specifically, for all mandatory-to-implement footprint types, footprints can
  be viewed as constraints for delegating requests to a dCDN: A dCDN footprint
  advertisement tells the uCDN the limitations for delegating a request to the
  dCDN. For IP prefixes or Autonomous System Numbers (ASNs), the footprint signals to the uCDN that it
  should consider the dCDN a candidate only if the IP address of the request
  routing source falls within the prefix set or ASN, respectively. The CDNI
  specifications do not define how a given uCDN determines what address ranges
  are in a particular ASN. Similarly, for country codes, a uCDN should only
  consider the dCDN a candidate if it covers the country of the request routing
  source. The CDNI specifications do not define how a given uCDN determines the
  country of the request routing source. Different types of footprint
  constraints can be combined together to narrow the dCDN candidacy, i.e., the
  uCDN should consider the dCDN a candidate only if the request routing source
  satisfies all the types of footprint constraints in the advertisement.
* Given that a large part of Footprint and Capabilities Advertisement may
  happen in contractual agreements, the semantics of CDNI Footprint and
  Capabilities advertisement refers to answering the following question: what
  exactly still needs to be advertised by the CDNI FCI? For instance, updates
  about temporal failures of part of a footprint can be useful information to
  convey via the CDNI FCI. Such information would provide updates on information
  previously agreed in contracts between the participating CDNs. In other words,
  the CDNI FCI is a means for a dCDN to provide changes/updates
  regarding a footprint and/or capabilities that it has previously agreed to serve in
  a contract with a uCDN. Hence, server push and incremental
  encoding will be necessary techniques.

## ALTO Background and Benefits {#bgALTO}

Application-Layer Traffic Optimization (ALTO) {{RFC7285}} defines an approach
for conveying network layer (topology) information to "guide" the resource
provider selection process in distributed applications that can choose among
several candidate resources providers to retrieve a given resource. Usually, it
is assumed that an ALTO server conveys information that these applications
cannot measure or have difficulty measuring themselves {{RFC5693}}.

Originally, ALTO was motivated by optimizing cross-ISP traffic generated by P2P
applications {{RFC5693}}. However, ALTO can also be used for improving the
request routing in CDNs. In particular, Section 5 of {{RFC7971}}
explicitly mentions ALTO as a candidate protocol to improve the selection of a
CDN surrogate or origin.

The following reasons make ALTO a suitable candidate protocol for dCDN
selection as part of CDNI request routing and, in particular,
for an FCI protocol:

* Application Layer-oriented: ALTO is a protocol specifically designed to
  improve application layer traffic (and application layer connections among
  hosts on the Internet) by providing additional information to applications
  that these applications could not easily retrieve themselves. This matches the
  need of CDNI, where a uCDN wants to improve application layer CDN request routing by
  using information (provided by a dCDN) that the uCDN could not easily obtain
  otherwise. Hence, ALTO can help a uCDN to select a proper dCDN by first
  providing dCDNs' capabilities as well as footprints (see [](#cdnifci)) and
  then providing costs of surrogates in a dCDN by ALTO cost maps.
- Security: The identification between uCDNs and dCDNs is an important
  requirement (see [](#security)). ALTO maps can be signed and hence provide
  inherent origin protection. Please see Section 15.1.2 of {{RFC7285}} for
  detailed protection strategies.
* RESTful design: The ALTO protocol has undergone extensive revisions in order
  to provide a RESTful design regarding the client-server interaction specified
  by the protocol. It is flexible and extensible enough to handle existing and
  potential future data formats defined by CDNI. It can provide the consistent
  client-server interaction model for other existing CDNI interfaces or
  potential future extensions and therefore reduce the learning cost for both
  users and developers, although they are not in the scope of this
  document. A CDNI FCI interface based on ALTO would inherit this RESTful
  design. Please see [](#cdnifci).
* Error-handling: The ALTO protocol provides extensive error-handling in the
  whole request and response process (see Section 8.5 of {{RFC7285}}). A CDNI
  FCI interface based on ALTO would inherit this extensive error-handling
  framework. Please see [](#filteredcdnifci).
* Map Service: The semantics of an ALTO network map is an exact match for the
  needed information to convey a footprint by a dCDN, in
  particular, if such a footprint is being expressed by IP-prefix
  ranges. Please see [](#cdnifcinetworkmap).
* Filtered Map Service: The ALTO map filtering service would allow a uCDN to
  query only for parts of an ALTO map. For example, the ALTO filtered property
  map service can enable a uCDN to query properties of a part of footprints
  efficiently. Please see [](#unifiedpropertymap).
* Server-initiated notifications and incremental updates: When the footprint or
  the capabilities of a dCDN change (i.e., unexpectedly from the perspective of
  a uCDN), server-initiated notifications would enable a dCDN to inform a uCDN
  about such changes directly. Consider the case where - due to failure - part
  of the footprint of the dCDN is not functioning, i.e., the CDN cannot serve
  content to such clients with reasonable QoS. Without server-initiated
  notifications, the uCDN might still use a recent network and cost map from the
  dCDN, and therefore redirect requests to the dCDN which it cannot serve.
  Similarly, the possibility for incremental updates would enable efficient
  conveyance of the aforementioned (or similar) status changes by the dCDN to
  the uCDN. The newest design of ALTO supports server pushed incremental updates
  {{RFC8895}}.
* Content availability on hosts: A dCDN might want to express CDN capabilities
in terms of certain content types (e.g., codecs/ formats, or content from
certain content providers). ALTO Entity Property Map
{{I-D.ietf-alto-unified-props-new}} would enable a dCDN to make such
information available to a uCDN. This would enable a uCDN to access whether
a dCDN has the capabilities for a given type of content requested.
* Resource availability on hosts or links: The capabilities on links (e.g.,
  maximum bandwidth) or caches (e.g., average load) might be useful information
  for a uCDN for optimized dCDN selection. For instance, if a uCDN receives a
  streaming request for content with a certain bitrate, it needs to know if it
  is likely that a dCDN can fulfill such stringent application-level
  requirements (i.e., can be expected to have enough consistent bandwidth)
  before it redirects the request. In general, if ALTO could convey such
  information via ALTO Entity Property Map {{I-D.ietf-alto-unified-props-new}},
  it would enable more sophisticated means for dCDN selection with ALTO. ALTO
  Path Vector Extension {{I-D.ietf-alto-path-vector}} is designed to allow ALTO
  clients to query information such as capacity regions for a given set of
  flows.
