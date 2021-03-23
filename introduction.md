<!-- Skip header line -->

# Introduction {#intro}

The ability to interconnect multiple content delivery networks (CDNs)
has many benefits, including increased coverage, capability, and
reliability. The Content Delivery Networks Interconnection (CDNI)
framework {{RFC6707}} defines four interfaces to
achieve the interconnection of CDNs: (1) the CDNI Request Routing
Interface; (2) the CDNI Metadata Interface; (3) the CDNI Logging
Interface; and (4) the CDNI Control Interface.

Among the four interfaces, the CDNI Request Routing Interface
provides key functions, as specified in {{RFC6707}}:
"The CDNI Request Routing interface enables a Request Routing
function in an Upstream CDN to query a Request Routing function in a
Downstream CDN to determine if the Downstream CDN is able (and
willing) to accept the delegated Content Request. It also allows the
Downstream CDN to control what should be returned to the User Agent
in the redirection message by the upstream Request Routing function."
At a high level, the scope of the CDNI Request Routing Interface,
therefore, contains two main tasks: (1) determining if the dCDN
(downstream CDN) is willing to accept a delegated content request,
and (2) redirecting the content request coming from a uCDN (upstream
CDN) to the proper entry point or entity in the dCDN.

Correspondingly, the request routing interface is broadly divided
into two functionalities: (1) the CDNI Footprint &amp; Capabilities
Advertisement interface (FCI) defined in {{RFC8008}},
and (2) the CDNI Request Routing Redirection interface (RI) defined
in {{RFC7975}}. Since this document focuses on the
first functionality (CDNI FCI), below is more details about it.

Specifically, CDNI FCI allows both an advertisement from a dCDN to a
uCDN (push) and a query from a uCDN to a dCDN (pull) so that the uCDN
knows whether it can redirect a particular user request to that dCDN.

A key component in defining CDNI FCI is defining objects describing the
footprints and capabilities of a dCDN. Such objects are already defined in
{{RFC8008}}. A protocol to transport and update such objects between a uCDN and
a dCDN, however, is not defined. Hence, the scope of this document is to define
such a protocol by introducing a new Application-Layer Traffic Optimization
(ALTO) {{RFC7285}} service called "CDNI Advertisement Service".

There are multiple benefits in using ALTO as a transport protocol, as discussed
in [](#bgALTO).

The rest of this document is organized as follows. [](#background) provides
non-normative background on both CDNI FCI and ALTO. [](#cdnifci) introduces the
most basic service, called "CDNI Advertisement Service", to realize CDNI FCI
using ALTO. [](#cdnifcinetworkmap) demonstrates a key benefit of using ALTO: the
ability to integrate CDNI FCI with ALTO network maps. Such integration provides
new granularity to describe footprints. [](#filteredcdnifci) introduces
"Filtered CDNI Advertisement Service" to allow a uCDN to get footprints with
given capabilities instead of getting the full resource, which can be large.
[](#unifiedpropertymap) further shows another benefit of using ALTO: the ability
to query footprint properties using ALTO entity property map extension. In this
way, a uCDN can effectively fetch capabilities of footprints in which it is
interested. IANA and security considerations are discussed in [](#iana) and
[](#security) respectively.
