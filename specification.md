<!-- Skip header line -->

# CDNI Advertisement Service {#cdnifci}

The ALTO protocol is based on the ALTO Information Service Framework which
consists of multiple services, where all ALTO services are "provided through a
common transport protocol, messaging structure and encoding, and transaction
model" {{RFC7285}}. The ALTO protocol specification {{RFC7285}} defines multiple
initial services, e.g., the ALTO network map service and cost map service.

This document defines a new ALTO service called "CDNI Advertisement Service"
which conveys JSON {{RFC8259}} objects of media type "application/alto-cdni+json". These
JSON objects are used to transport BaseAdvertisementObject objects defined in
{{RFC8008}}; this document specifies how to transport such
BaseAdvertisementObject objects via the ALTO protocol with the ALTO "CDNI
Advertisement Service". Similar to other ALTO services, this document defines
the ALTO information resource for the "CDNI Advertisement Service" as follows.

Note that the encoding of BaseAdvertisementObject reuses the one
defined in {{RFC8008}} and therefore also follows the recommendations of I-JSON
(Internet JSON) {{RFC7493}}, which is required by {{RFC8008}}.

## Media Type {#cdnifcimediatype}

The media type of the CDNI Advertisement resource is
"application/alto-cdni+json".

## HTTP Method {#cdnifcimethod}

A CDNI Advertisement resource is requested using the HTTP GET method.

## Accept Input Parameters {#cdnifciinput}

None.

## Capabilities {#cdnifcicap}

None.

## Uses {#cdnifciuses}

The `uses` field MUST NOT appear unless the CDNI Advertisement resource
depends on other ALTO information resources. If the CDNI Advertisement
resource has dependent resources, the resource IDs of its
dependent resources MUST be included into the `uses` field. This
document only defines one potential dependent resource for the CDNI
Advertisement resource. See [](#cdnifcinetworkmap) for details
of when and how to use it. Future documents may extend the CDNI Advertisement
resource and allow other dependent resources.

## Response {#cdnifciencoding}

The `meta` field of a CDNI Advertisement response MUST include the `vtag`
field defined in Section 10.3 of {{RFC7285}}. This
field provides the version of the retrieved CDNI FCI resource.

If a CDNI Advertisement response depends on other ALTO information resources, it
MUST include the `dependent-vtags` field, whose value is an array to indicate
the version tags of the resources used, where each resource is specified in
`uses` of its Information Resource Directory (IRD) entry.

The data component of an ALTO CDNI Advertisement response is named
`cdni-advertisement`, which is a JSON object of type CDNIAdvertisementData:

~~~
    object {
      CDNIAdvertisementData cdni-advertisement;
    } InfoResourceCDNIAdvertisement : ResponseEntityBase;

    object {
      BaseAdvertisementObject capabilities-with-footprints<0..*>;
    } CDNIAdvertisementData;
~~~

Specifically, a CDNIAdvertisementData object is a JSON object that includes
only one property named `capabilities-with-footprints`, whose value is an array
of BaseAdvertisementObject objects. It provides capabilities with footprint
restrictions for uCDN to decide the dCDN selection. If the value of this
property is an empty array, it means the corresponding dCDN cannot provide any
mandatory-to-implement CDNI capabilities for any footprints.

The syntax and semantics of BaseAdvertisementObject are well defined in Section
5.1 of {{RFC8008}}. A BaseAdvertisementObject object includes multiple
properties, including capability-type, capability-value, and footprints, where
footprints are defined in Section 4.2.2.2 of {{RFC8006}}.

To be self-contained, below is an equivalent specification of
BaseAdvertisementObject described in the ALTO-style notation (see Section 8.2
of {{RFC7285}}). As mentioned above, the normative specification of
BaseAdvertisementObject is in {{RFC8008}}.

~~~
    object {
      JSONString capability-type;
      JSONValue capability-value;
      Footprint footprints<0..*>;
    } BaseAdvertisementObject;

    object {
      JSONString footprint-type;
      JSONString footprint-value<1..*>;
    } Footprint;
~~~

For each BaseAdvertisementObject, the ALTO client MUST interpret footprints
appearing multiple times as if they appeared only once. If footprints in a
BaseAdvertisementObject is null or empty or not appearing, the ALTO client MUST
understand that the capabilities in this BaseAdvertisementObject have the
"global" coverage, i.e., the corresponding dCDN can provide them for any
request routing source.

Note: Further optimization of BaseAdvertisement objects to effectively provide
the advertisement of capabilities with footprint restrictions is certainly
possible. For example, these two examples below both describe that the dCDN can
provide capabilities \["http/1.1", "https/1.1"\] for the same footprints. However,
the latter one is smaller in its size.

~~~
EXAMPLE 1
    {
      "meta" : {...},
      "cdni-advertisement": {
        "capabilities-with-footprints": [
          {
            "capability-type": "FCI.DeliveryProtocol",
            "capability-value": {
              "delivery-protocols": [
                "http/1.1"
              ]
            },
            "footprints": [
              <Footprint objects>
            ]
          },
          {

            "capability-type": "FCI.DeliveryProtocol",
            "capability-value": {
              "delivery-protocols": [
                "https/1.1"
              ]
            },
            "footprints": [
              <Footprint objects>
            ]
          }
        ]
      }
    }

EXAMPLE 2
    {
      "meta" : {...},
      "cdni-advertisement": {
        "capabilities-with-footprints": [
          {
            "capability-type": "FCI.DeliveryProtocol",
            "capability-value": {
              "delivery-protocols": [
                "https/1.1",
                "http/1.1"
              ]
            },
            "footprints": [
              <Footprint objects>
            ]
          }
        ]
      }
    }
~~~

Since such optimizations are not required for the basic interconnection of CDNs,
the specifics of such mechanisms are outside the scope of this document.

This document only requires the ALTO server to provide the initial FCI-specific
CDNI Payload Types defined in {{RFC8008}} as the mandatory-to-implement CDNI
capabilities. There may be other documents extending BaseAdvertisementObject and
additional CDNI capabilities. They are outside the scope of this document. To
support them, future documents can extend the specification defined in this
document.

## Examples {#cdnifciexamples}

### IRD Example {#IRDexample}

Below is the IRD of a simple, example ALTO
server. The server provides both base ALTO information resources (e.g., network
maps) and CDNI FCI related information resources (e.g., CDNI Advertisement
resources), demonstrating a single, integrated environment.

Specifically, the IRD announces two network maps, one CDNI Advertisement
resource without dependency, one CDNI Advertisement resource depending on a
network map, one filtered CDNI Advertisement resource to be defined in
[](#filteredcdnifci), one property map including `cdni-capabilities` as its
entity property, one filtered property map including `cdni-capabilities` and
`pid` as its entity properties, and two update stream services (one for updating
CDNI Advertisement resources, and the other for updating property maps).

~~~
 GET /directory HTTP/1.1
 Host: alto.example.com
 Accept: application/alto-directory+json,application/alto-error+json

 HTTP/1.1 200 OK
 Content-Length: 3571
 Content-Type: application/alto-directory+json

 {
   "meta" : {
     "default-alto-network-map": "my-default-network-map"
   },
   "resources": {
     "my-default-network-map": {
       "uri" : "https://alto.example.com/networkmap",
       "media-type" : "application/alto-networkmap+json"
     },
     "my-eu-netmap" : {
       "uri" : "https://alto.example.com/myeunetmap",
       "media-type" : "application/alto-networkmap+json"
     },
     "my-default-cdnifci": {
       "uri" : "https://alto.example.com/cdnifci",
       "media-type": "application/alto-cdni+json"
     },
     "my-cdnifci-with-pid-footprints": {
       "uri" : "https://alto.example.com/networkcdnifci",
       "media-type" : "application/alto-cdni+json",
       "uses" : [ "my-eu-netmap" ]
     },
     "my-filtered-cdnifci" : {
       "uri" : "https://alto.example.com/cdnifci/filtered",
       "media-type" : "application/alto-cdni+json",
       "accepts" : "application/alto-cdnifilter+json"
     },
     "cdnifci-property-map" : {
       "uri" : "https://alto.example.com/propmap/full/cdnifci",
       "media-type" : "application/alto-propmap+json",
       "uses": [ "my-default-cdni" ],
       "capabilities" : {
         "mappings": {
           "ipv4": [ "my-default-cdni.cdni-capabilities" ],
           "ipv6": [ "my-default-cdni.cdni-capabilities" ],
           "countrycode": [
             "my-default-cdni.cdni-capabilities" ],
           "asn": [ "my-default-cdni.cdni-capabilities" ]
         }
       }
     },
     "filtered-cdnifci-property-map" : {
       "uri" : "https://alto.example.com/propmap/lookup/cdnifci-pid",
       "media-type" : "application/alto-propmap+json",
       "accepts" : "application/alto-propmapparams+json",
       "uses": [ "my-default-cdni", "my-default-network-map" ],
       "capabilities" : {
         "mappings": {
           "ipv4": [ "my-default-cdni.cdni-capabilities",
                     "my-default-network-map.pid" ],
           "ipv6": [ "my-default-cdni.cdni-capabilities",
                     "my-default-network-map.pid" ],
           "countrycode": [
             "my-default-cdni.cdni-capabilities" ],
           "asn": [ "my-default-cdni.cdni-capabilities" ]
         }
       }
     },
     "update-my-cdni-fci" : {
       "uri": "https:///alto.example.com/updates/cdnifci",
       "media-type" : "text/event-stream",
       "accepts" : "application/alto-updatestreamparams+json",
       "uses" : [
         "my-default-network-map",
         "my-eu-netmap",
         "my-default-cdnifci",
         "my-filtered-cdnifci",
         "my-cdnifci-with-pid-footprints"
       ],
       "capabilities" : {
         "incremental-change-media-types" : {
          "my-default-network-map" : "application/json-patch+json",
          "my-eu-netmap" : "application/json-patch+json",
          "my-default-cdnifci" :
          "application/merge-patch+json,application/json-patch+json",
          "my-filtered-cdnifci" :
          "application/merge-patch+json,application/json-patch+json",
          "my-cdnifci-with-pid-footprints" :
          "application/merge-patch+json,application/json-patch+json"
         }
       }
     },
     "update-my-props": {
       "uri" : "https://alto.example.com/updates/properties",
       "media-type" : "text/event-stream",
       "uses" : [
         "cdnifci-property-map",
         "filtered-cdnifci-property-map"
       ],
       "capabilities" : {
         "incremental-change-media-types": {
          "cdnifci-property-map" :
          "application/merge-patch+json,application/json-patch+json",
          "filtered-cdnifci-property-map":
          "application/merge-patch+json,application/json-patch+json"
         }
       }
     }
   }
 }
~~~

### Basic Example {#fullcdnifciexample}

This basic example demonstrates a simple CDNI Advertisement resource, which does
not depend on other resources. There are three BaseAdvertisementObjects in this
resource and these objects' capabilities are http/1.1 delivery protocol,
\[http/1.1, https/1.1\] delivery protocol, and https/1.1 acquisition protocol,
respectively.

~~~
  GET /cdnifci HTTP/1.1
  Host: alto.example.com
  Accept: application/alto-cdni+json,
          application/alto-error+json

  HTTP/1.1 200 OK
  Content-Length: 1235
  Content-Type: application/alto-cdni+json

  {
    "meta" : {
      "vtag": {
        "resource-id": "my-default-cdnifci",
        "tag": "da65eca2eb7a10ce8b059740b0b2e3f8eb1d4785"
      }
    },
    "cdni-advertisement": {
      "capabilities-with-footprints": [
        {
          "capability-type": "FCI.DeliveryProtocol",
          "capability-value": {
            "delivery-protocols": [
              "http/1.1"
            ]
          },
          "footprints": [
            {
              "footprint-type": "ipv4cidr",
              "footprint-value": [ "192.0.2.0/24" ]
            }
          ]
        },
        {
          "capability-type": "FCI.DeliveryProtocol",
          "capability-value": {
            "delivery-protocols": [
              "https/1.1",
              "http/1.1"
            ]
          },
          "footprints": [
            {
              "footprint-type": "ipv4cidr",
              "footprint-value": [ "198.51.100.0/24" ]
            }
          ]
        },
        {
          "capability-type": "FCI.AcquisitionProtocol",
          "capability-value": {
            "acquisition-protocols": [
              "https/1.1"
            ]
          },
          "footprints": [
            {
              "footprint-type": "ipv4cidr",
              "footprint-value": [ "203.0.113.0/24" ]
            }
          ]
        }
      ]
    }
  }
~~~

### Incremental Updates Example

A benefit of using ALTO to provide CDNI Advertisement resources is that such
resources can be updated using ALTO incremental updates {{RFC8895}}. Below is
an example that also shows the benefit of having both JSON merge patch and JSON
patch to encode updates.

At first, an ALTO client requests updates for "my-default-cdnifci", and the ALTO
server returns the `control-uri` followed by the full CDNI Advertisement
response. Then when there is a change in the delivery-protocols in that http/1.1
is removed (from \[http/1.1, https/1.1\] to only https/1.1) due to maintenance of
the http/1.1 clusters, the ALTO server regenerates the new CDNI Advertisement
resource and pushes the full replacement to the ALTO client. Later on, the ALTO
server notifies the ALTO client that "192.0.2.0/24" is added into the "ipv4"
footprint object for delivery-protocol https/1.1 by sending the change encoded
by JSON patch to the ALTO client.

~~~
 POST /updates/cdnifci HTTP/1.1
 Host: alto.example.com
 Accept: text/event-stream,application/alto-error+json
 Content-Type: application/alto-updatestreamparams+json
 Content-Length: 92

 { "add": {
     "my-cdnifci-stream": {
       "resource-id": "my-default-cdnifci"
     }
   }
 }

 HTTP/1.1 200 OK
 Connection: keep-alive
 Content-Type: text/event-stream

 event: application/alto-updatestreamcontrol+json
 data: {"control-uri":
 data: "https://alto.example.com/updates/streams/3141592653589"}

 event: application/alto-cdni+json,my-cdnifci-stream
 data: { ... full CDNI Advertisement resource ... }

 event: application/alto-cdni+json,my-cdnifci-stream
 data: {
 data:   "meta": {
 data:     "vtag": {
 data:       "tag": "dasdfa10ce8b059740bddsfasd8eb1d47853716"
 data:     }
 data:   },
 data:   "cdni-advertisement": {
 data:     "capabilities-with-footprints": [
 data:       {
 data:         "capability-type": "FCI.DeliveryProtocol",
 data:         "capability-value": {
 data:           "delivery-protocols": [
 data:             "https/1.1"
 data:           ]
 data:         },
 data:         "footprints": [
 data:           { "footprint-type": "ipv4cidr",
 data:             "footprint-value": [ "203.0.113.0/24" ]
 data:           }
 data:         ]
 data:       },
 data:       { ... other CDNI advertisement object ... }
 data:     ]
 data:   }
 data: }

 event: application/json-patch+json,my-cdnifci-stream
 data: [
 data:   { "op": "replace",
 data:     "path": "/meta/vtag/tag",
 data:     "value": "a10ce8b059740b0b2e3f8eb1d4785acd42231bfe"
 data:   },
 data:   { "op": "add",
 data:     "path": "/cdni-advertisement/capabilities-with-footprints
 /0/footprints/0/footprint-value/-",
 data:     "value": "192.0.2.0/24"
 data:   }
 data: ]
~~~

# CDNI Advertisement Service using ALTO Network Map {#cdnifcinetworkmap}

## Network Map Footprint Type: altopid

The ALTO protocol defines a concept called PID to represent a group of IPv4 or
IPv6 addresses which can be applied the same management policy. The PID is an
alternative to the pre-defined CDNI footprint types (i.e., ipv4cidr, ipv6cidr,
asn, and countrycode).

To leverage this concept, this document defines a new CDNI Footprint Type called
"altopid". A CDNI Advertisement resource can depend on an ALTO network map
resource and use "altopid" footprints to compress its CDNI Footprint Payload.

Specifically, the "altopid" footprint type indicates that the corresponding
footprint value is a list of PIDNames as defined in {{RFC7285}}.
These PIDNames are references of PIDs in a network map resource. Hence a CDNI
Advertisement resource using "altopid" footprints depends on a network map. For
such a CDNI Advertisement resource, the resource id of its dependent network map
MUST be included in the `uses` field of its IRD entry, and the `dependent-vtags`
field with a reference to this network map MUST be included in its response (see
the example in [](#networkmapfootprint)).

## Examples

### IRD Example

The examples below use the same IRD given in [](#IRDexample).

### ALTO Network Map for CDNI Advertisement Example {#networkmapexample}

Below is an example network map whose resource id is "my-eu-netmap", and this
map is referenced by the CDNI Advertisement example in [](#networkmapfootprint).

~~~
 GET /myeunetmap HTTP/1.1
 Host: alto.example.com
 Accept: application/alto-networkmap+json,application/alto-error+json

 HTTP/1.1 200 OK
 Content-Length: 309
 Content-Type: application/alto-networkmap+json

 {
   "meta": {
     "vtag": [
       { "resource-id": "my-eu-netmap",
         "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"
       }
     ]
   },
   "network-map": {
     "south-france" : {
       "ipv4": [ "192.0.2.0/24", "198.51.100.0/25" ]
     },
     "germany": {
       "ipv4": [ "203.0.113.0/24" ]
     }
   }
 }
~~~

### ALTO PID Footprints in CDNI Advertisement {#networkmapfootprint}

This example shows a CDNI Advertisement resource that depends on a network map
described in [](#networkmapexample).

~~~
 GET /networkcdnifci HTTP/1.1
 Host: alto.example.com
 Accept: application/alto-cdni+json,application/alto-error+json

 HTTP/1.1 200 OK
 Content-Length: 738
 Content-Type: application/alto-cdni+json

 {
   "meta" : {
     "dependent-vtags" : [
       {
         "resource-id": "my-eu-netmap",
         "tag": "3ee2cb7e8d63d9fab71b9b34cbf764436315542e"
       }
     ]
   },
   "cdni-advertisement": {
     "capabilities-with-footprints": [
       { "capability-type": "FCI.DeliveryProtocol",
         "capability-value": [ "https/1.1" ],
         "footprints": [
           { "footprint-type": "altopid",
             "footprint-value": [ "south-france" ]
           }
         ]
       },
       { "capability-type": "FCI.AcquisitionProtocol",
         "capability-value": [ "https/1.1" ],
         "footprints": [
           { "footprint-type": "altopid",
             "footprint-value": [ "germany", "south-france" ]
           }
         ]
       }
     ]
   }
 }
~~~

### Incremental Updates Example

In this example, the ALTO client is interested in changes of
"my-cdnifci-with-pid-footprints" and its dependent network map "my-eu-netmap".
Considering two changes, the first one is to change footprints of the https/1.1
delivery protocol capability, and the second one is to remove the
"south-france" PID from the footprints of the https/1.1 acquisition protocol
capability.

~~~
  POST /updates/cdnifci HTTP/1.1
  Host: alto.example.com
  Accept: text/event-stream,application/alto-error+json
  Content-Type: application/alto-updatestreamparams+json
  Content-Length: 183

  { "add": {
      "my-eu-netmap-stream": {
        "resource-id": "my-eu-netmap"
      },
      "my-netmap-cdnifci-stream": {
        "resource-id": "my-cdnifci-with-pid-footprints"
      }
    }
  }

  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Type: text/event-stream

  event: application/alto-updatestreamcontrol+json
  data: {"control-uri":
  data: "https://alto.example.com/updates/streams/3141592653590"}

  event: application/alto-networkmap+json,my-eu-netmap-stream
  data: { ... full Network Map of my-eu-netmap ... }

  event: application/alto-cdnifci+json,my-netmap-cdnifci-stream
  data: { ... full CDNI Advertisement resource ... }

  event: application/json-patch+json,my-netmap-cdnifci-stream
  data: [
  data:   { "op": "replace",
  data:     "path": "/meta/vtag/tag",
  data:     "value": "dasdfa10ce8b059740bddsfasd8eb1d47853716"
  data:   },
  data:   { "op": "add",
  data:     "path":
  data:     "/cdni-advertisement/capabilities-with-footprints
  /0/footprints/0/footprint-value/-",
  data:     "value": "germany"
  data:   }
  data: ]

  event: application/json-patch+json,my-netmap-cdnifci-stream
  data: [
  data:   { "op": "replace",
  data:     "path": "/meta/vtag/tag",
  data:     "value": "a10ce8b059740b0b2e3f8eb1d4785acd42231bfe"
  data:   },
  data:   { "op": "remove",
  data:     "path":
  data:     "/cdni-advertisement/capabilities-with-footprints
  /1/footprints/0/footprint-value/1"
  data:   }
  data: ]
~~~

# Filtered CDNI Advertisement using CDNI Capabilities {#filteredcdnifci}

[](#cdnifci) and [](#cdnifcinetworkmap) describe CDNI Advertisement Service
which can be used to enable a uCDN to get capabilities with footprint
restrictions from dCDNs. However, since always getting full CDNI Advertisement
resources from dCDNs is inefficient, this document introduces a new service
named "Filtered CDNI Advertisement Service", to allow a client to filter a CDNI
Advertisement resource using a client-given set of CDNI capabilities. For each
entry of the CDNI Advertisement response, an entry will only be returned to the
client if it contains at least one of the client given CDNI capabilities. The
relationship between a filtered CDNI Advertisement resource and a CDNI
Advertisement resource is similar to the relationship between a filtered
network/cost map and a network/cost map.

## Media Type

A filtered CDNI Advertisement resource uses the same media type defined for the
CDNI Advertisement resource in [](#cdnifcimediatype).

## HTTP Method

A filtered CDNI Advertisement resource is requested using the HTTP POST method.

## Accept Input Parameters {#filteredcdnifciinputs}

The input parameters for a filtered CDNI Advertisement resource are supplied in
the entity body of the POST request. This document specifies the input
parameters with a data format indicated by the media type
"application/alto-cdnifilter+json" which is a JSON object of type
ReqFilteredCDNIAdvertisement, where:

~~~
   object {
       JSONString capability-type;
       JSONValue capability-value;
   } CDNICapability;

   object {
       CDNICapability cdni-capabilities<0..*>;
   } ReqFilteredCDNIAdvertisement;

~~~

with fields:

capability-type:
: The same as Base Advertisement Object's capability-type defined in Section 5.1
  of {{RFC8008}}.

capability-value:
: The same as Base Advertisement Object's capability-value defined in Section
  5.1 of {{RFC8008}}.

cdni-capabilities:
: A list of CDNI capabilities defined in Section 5.1 of {{RFC8008}} for which
  footprints are to be returned. If this list is empty, the ALTO
  server MUST interpret it as a request for the full CDNI Advertisement
  resource. The ALTO server MUST interpret entries appearing in this list multiple
  times as if they appeared only once. If the ALTO server does not define any
  footprints for a CDNI capability, it MUST omit this capability from the
  response.

## Capabilities

None.

## Uses

Same to the `uses` field of the CDNI Advertisement resource (see
[](#cdnifciuses)).

## Response

If the request is invalid, the response MUST indicate an error, using ALTO
protocol error handling specified in Section 8.5 of {{RFC7285}}.

Specifically, a filtered CDNI Advertisement request is invalid if:

* the value of `capability-type` is null;
* the value of `capability-value` is null;
* the value of `capability-value` is inconsistent with `capability-type`.

When a request is invalid, the ALTO server MUST return an
`E_INVALID_FIELD_VALUE` error defined in Section 8.5.2 of {{RFC7285}}, and the
`value` field of the error message SHOULD indicate this CDNI capability.

The ALTO server returns a filtered CDNI Advertisement resource for a valid
request. The format of a filtered CDNI Advertisement resource is the same as a
full CDNI Advertisement resource (See [](#cdnifciencoding).)

<!--
The returned CDNI Advertisement resource MUST contain only
BaseAdvertisementObject objects whose CDNI capability object is the superset of
one of CDNI capability object in `cdni-fci-capabilities`. Specifically, that a
CDNI capability object A is the superset of another CDNI capability object B
means that these two CDNI capability objects have the same capability type and
mandatory properties in capability value of A MUST include mandatory properties
in capability value of B semantically.
-->

The returned filtered CDNI Advertisement resource MUST contain all the
BaseAdvertisementObject objects satisfying the following condition: The CDNI
capability object of each included BaseAdvertisementObject object MUST follow
two constraints:

- The "cdni-capabilities" field of the input includes a CDNI capability object
  X having the same capability type as it.
- All the mandatory properties in its capability value is a superset of
  mandatory properties in capability value of X semantically.

See [](#filteredcdnifciexample) for a concrete example.

The version tag included in the `vtag` field of the response MUST correspond to
the full CDNI Advertisement resource from which the filtered CDNI Advertisement
resource is provided. This ensures that a single, canonical version tag is used
independently of any filtering that is requested by an ALTO client.

## Examples

### IRD Example

The examples below use the same IRD example as in [](#IRDexample).

### Basic Example {#filteredcdnifciexample}

This example filters the full CDNI Advertisement resource in
[](#fullcdnifciexample) by selecting only the http/1.1 delivery protocol
capability. Only the second BaseAdvertisementObjects in the full resource will
be returned because the second object's capability is http/1.1 and https/1.1
delivery protocols which is the superset of https/1.1 delivery protocol.

~~~
  POST /cdnifci/filtered HTTP/1.1
  Host: alto.example.com
  Accept: application/alto-cdni+json
  Content-Type: application/cdnifilter+json
  Content-Length: 176

  {
    "cdni-capabilities": [
      {
        "capability-type": "FCI.DeliveryProtocol",
        "capability-value": {
          "delivery-protocols": [ "https/1.1" ]
        }
      }
    ]
  }

  HTTP/1.1 200 OK
  Content-Length: 571
  Content-Type: application/alto-cdni+json

  {
    "meta" : {
      "vtag": {
        "resource-id": "my-filtered-cdnifci",
        "tag": "da65eca2eb7a10ce8b059740b0b2e3f8eb1d4785"
      }
    },
    "cdni-advertisement": {
      "capabilities-with-footprints": [
        {
          "capability-type": "FCI.DeliveryProtocol",
          "capability-value": {
            "delivery-protocols": [
              "https/1.1",
              "http/1.1"
            ]
          },
          "footprints": [
            {
              "footprint-type": "ipv4cidr",
              "footprint-value": [ "198.51.100.0/24" ]
            }
          ]
        }
      ]
    }
  }
~~~

### Incremental Updates Example

In this example, the ALTO client only cares about the updates of one
advertisement object for delivery protocol capability whose value includes
"https/1.1". So it adds its limitation of capabilities in `input` field of the
POST request.

~~~
  POST /updates/cdnifci HTTP/1.1
  Host: fcialtoupdate.example.com
  Accept: text/event-stream,application/alto-error+json
  Content-Type: application/alto-updatestreamparams+json
  Content-Length: 346

  {
    "add": {
      "my-filtered-fci-stream": {
        "resource-id": "my-filtered-cdnifci",
        "input": {
          "cdni-capabilities": [
            {
              "capability-type": "FCI.DeliveryProtocol",
              "capability-value": {
                "delivery-protocols": [ "https/1.1" ]
              }
            }
          ]
        }
      }
    }
  }

  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Type: text/event-stream

  event: application/alto-updatestreamcontrol+json
  data: {"control-uri":
  data: "https://alto.example.com/updates/streams/3141592653590"}

  event: application/alto-cdni+json,my-filtered-fci-stream
  data: { ... filtered CDNI Advertisement resource ... }

  event: application/json-patch+json,my-filtered-fci-stream
  data: [
  data:   {
  data:     "op": "replace",
  data:     "path": "/meta/vtag/tag",
  data:     "value": "a10ce8b059740b0b2e3f8eb1d4785acd42231bfe"
  data:   },
  data:   { "op": "add",
  data:     "/cdni-advertisement/capabilities-with-footprints
  /0/footprints/0/footprint-value/-",
  data:     "value": "192.0.2.0/24"
  data:   }
  data: ]
~~~

# Query Footprint Properties using ALTO Property Map Service {#unifiedpropertymap}

Besides the requirement of retrieving footprints of given capabilities, another
common requirement for uCDN is to query CDNI capabilities of given footprints.

Considering each footprint as an entity with properties including CDNI
capabilities, a natural way to satisfy this requirement is to use the ALTO
property map as defined in {{I-D.ietf-alto-unified-props-new}}. This section
describes how ALTO clients look up properties for individual footprints. First,
it describes how to represent footprint objects as entities in the ALTO property
map. Then it describes how to represent footprint capabilities as entity
properties in the ALTO property map. Finally, it provides examples of the full
property map and the filtered property map supporting CDNI capabilities, and
their incremental updates.

## Representing Footprint Objects as Property Map Entities {#footprinttoentities}

A footprint object has two properties: footprint-type and footprint-value. A
footprint-value is an array of footprint values conforming to the specification
associated with the registered footprint type ("ipv4cidr", "ipv6cidr", "asn",
"countrycode", and "altopid"). Considering each ALTO entity defined in
{{I-D.ietf-alto-unified-props-new}} also has two properties: entity domain type
and domain-specific identifier, a straightforward approach to represent a
footprint as an ALTO entity is to represent its footprint-type as an entity
domain type, and its footprint value as a domain-specific identifier.

Each existing footprint type can be represented as an entity domain type as
follows:

* According to {{I-D.ietf-alto-unified-props-new}}, "ipv4" and "ipv6" are two
  predefined entity domain types, which can be used to represent "ipv4cidr" and
  "ipv6cidr" footprints respectively. Note that both "ipv4" and "ipv6" domains
  can include not only hierarchical addresses but also individual addresses.
  Therefore, a "ipv4cidr" or "ipv6cidr" footprint with the longest prefix can
  also be represented by an individual address entity. When the uCDN receives a
  property map with individual addresses in an "ipv4" or "ipv6" domain, it can
  translate them as corresponding "ipv4cidr" or "ipv6cidr" footprints with the
  longest prefix.
* "pid" is also a predefined entity domain type, which can be used to represent
  "altopid" footprints. Note that "pid" is a resource-specific entity domain. To
  represent an "altopid" footprint, the specifying information resource of the
  corresponding "pid" entity domain MUST be the dependent network map used by
  the CDNI Advertisement resource providing this "altopid" footprint.
* However, no existing entity domain type can represent "asn" and "countrycode"
  footprints. To represent footprint-type "asn" and "countrycode", this document
  registers two new domains in [](#iana) in addition to the ones in
  {{I-D.ietf-alto-unified-props-new}}.

Here is an example of representing a footprint object of "ipv4cidr" type as a
set of "ipv4" entities in the ALTO property map. The representation of the
footprint object of "ipv6cidr" type is similar.

~~~
{ "footprint-type": "ipv4cidr",
  "footprint-value": ["192.0.2.0/24", "198.51.100.0/24"]
} --> "ipv4:192.0.2.0/24", "ipv4:198.51.100.0/24"
~~~

And here is an example of corresponding footprint object of "ipv4cidr" type
represented by an individual address in an "ipv4" domain in the ALTO property
map. The translation of the entities in an "ipv6" domain is similar.

~~~
"ipv4:203.0.113.100" --> {
  "footprint-type": "ipv4cidr",
  "footprint-value": ["203.0.113.100/32"]
}
~~~

### ASN Domain

The ASN domain associates property values with Autonomous Systems in the
Internet.

#### Entity Domain Type

asn

#### Domain-Specific Entity Identifiers {#asn-entity-id}

The entity identifier of an entity in an asn domain is encoded as a string
consisting of the characters "as" (in lowercase) followed by the Autonomous
System Number {{RFC6793}}.

#### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with ASN.

### COUNTRYCODE Domain

The COUNTRYCODE domain associates property values with countries.

#### Entity Domain Type

countrycode

#### Domain-Specific Entity Identifiers {#countrycode-entity-id}

The entity identifier of an entity in a countrycode domain is encoded as an ISO
3166-1 alpha-2 code {{ISO3166-1}} in lowercase.

#### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with country
codes.

## Representing CDNI Capabilities as Property Map Entity Properties {#capabilitytoproperties}

This document defines a new entity property type called "cdni-capabilities". An
ALTO server can provide a property map resource mapping the "cdni-capablities"
entity property type for a CDNI Advertisement resource that it provides to an
"ipv4", "ipv6", "asn" or "countrycode" entity domain.

### Defining Information Resource Media Type for Property Type cdni-capabilities

The entity property type "cdni-capabilities" allows to define resource-specific
entity properties. When resource-specific entity properties are defined with
entity property type "cdni-capabilities", the defining information resource for
a "cdni-capabilities" property MUST be a CDNI Advertisement resource provided by
the ALTO server. The media type of the defining information resource for a
"cdni-capabilities" property is therefore:

application/alto-cdni+json

### Intended Semantics of Property Type cdni-capabilities

A "cdni-capabilities" property for an entity is to indicate all the CDNI
capabilities that a corresponding CDNI Advertisement resource provides for the
footprint represented by this entity. Thus, the value of a "cdni-capabilities"
property MUST be a JSON array. Each element in a "cdni-capabilities" property
MUST be an JSON object as format of CDNICapability (see
[](#filteredcdnifciinputs)). The value of a "cdni-capabilities" property for an
"ipv4", "ipv6", "asn", "countrycode" or "altopid" entity MUST include all the
CDNICapability objects satisfying the following conditions: (1) they are
provided by the defining CDNI Advertisement resource; and (2) the represented
footprint object of this entity is in their footprint restrictions.

## Examples

### IRD Example

The examples use the same IRD example given by [](#IRDexample).

### Property Map Example

This example shows a full property map in which entities are footprints and
entities' property is "cdni-capabilities".

~~~
 GET /propmap/full/cdnifci HTTP/1.1
 Host: alto.example.com
 Accept: application/alto-propmap+json,application/alto-error+json

 HTTP/1.1 200 OK
 Content-Length: 1522
 Content-Type: application/alto-propmap+json

 {
   "property-map": {
     "meta": {
       "dependent-vtags": [
         { "resource-id": "my-default-cdnifci",
           "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf62"}
       ]
     },
     "countrycode:us": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.DeliveryProtocol",
           "capability-value": {
             "delivery-protocols": ["http/1.1"]}}]
     },
     "ipv4:192.0.2.0/24": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.DeliveryProtocol",
           "capability-value": {
             "delivery-protocols": ["http/1.1"]}}]
     },
     "ipv4:198.51.100.0/24": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.DeliveryProtocol",
           "capability-value": {
             "delivery-protocols": ["https/1.1", "http/1.1"]}}]
     },
     "ipv4:203.0.113.0/24": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.AcquisitionProtocol",
           "capability-value": {
             "acquisition-protocols": ["http/1.1"]}}]
     },
     "ipv6:2001:db8::/32": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.DeliveryProtocol",
           "capability-value": {
             "delivery-protocols": ["http/1.1"]}}]
     },
     "asn:as64496": {
       "my-default-cdnifci.cdni-capabilities": [
         { "capability-type": "FCI.DeliveryProtocol",
           "capability-value": {
             "delivery-protocols": ["https/1.1", "http/1.1"]}}]
     }
   }
 }
~~~

### Filtered Property Map Example

This example uses the filtered property map service to get "pid" and
"cdni-capabilities" properties for two footprints "ipv4:192.0.2.0/24" and
"ipv6:2001:db8::/32".

~~~
   POST /propmap/lookup/cdnifci-pid HTTP/1.1
   Host: alto.example.com
   Content-Type: application/alto-propmapparams+json
   Accept: application/alto-propmap+json,application/alto-error+json
   Content-Length: 181

   {
     "entities": [
       "ipv4:192.0.2.0/24",
       "ipv6:2001:db8::/32"
     ],
     "properties": [ "my-default-cdnifci.cdni-capabilities",
                     "my-default-networkmap.pid" ]
   }

 HTTP/1.1 200 OK
 Content-Length: 796
 Content-Type: application/alto-propmap+json

 {
   "property-map": {
     "meta": {
       "dependent-vtags": [
          {"resource-id": "my-default-cdnifci",
            "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf62"},
          {"resource-id": "my-default-networkmap",
            "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf63"}
       ]
     },
     "ipv4:192.0.2.0/24": {
       "my-default-cdnifci.cdni-capabilities": [
         {"capability-type": "FCI.DeliveryProtocol",
          "capability-value": {"delivery-protocols": ["http/1.1"]}}],
       "my-default-networkmap.pid": "pid1"
     },
     "ipv6:2001:db8::/32": {
       "my-default-cdnifci.cdni-capabilities": [
         {"capability-type": "FCI.DeliveryProtocol",
          "capability-value": {"delivery-protocols": ["http/1.1"]}}],
       "my-default-networkmap.pid": "pid3"
     }
   }
 }
~~~

### Incremental Updates Example

In this example, the client is interested in updates for the properties
"cdni-capabilities" and "pid" of two footprints "ipv4:192.0.2.0/24" and
"countrycode:fr".

~~~
  POST /updates/properties HTTP/1.1
  Host: alto.example.com
  Accept: text/event-stream,application/alto-error+json
  Content-Type: application/alto-updatestreamparams+json
  Content-Length: 337

  { "add": {
      "fci-propmap-stream": {
        "resource-id": "filtered-cdnifci-property-map",
        "input": {
          "properties": [ "my-default-cdnifci.cdni-capabilities",
                          "my-default-networkmap.pid" ],
          "entities": [ "ipv4:192.0.2.0/24",
                        "ipv6:2001:db8::/32" ]
        }
      }
    }
  }

  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Type: text/event-stream

  event: application/alto-updatestreamcontrol+json
  data: {"control-uri":
  data: "https://alto.example.com/updates/streams/1414213562373"}

  event: application/alto-cdni+json,fci-propmap-stream
  data: { ... filtered property map ... }

  event: application/merge-patch+json,fci-propmap-stream
  data: {
  data:   "property-map": {
  data:     "meta": {
  data:       "dependent-vtags": [
  data:         { "resource-id": "my-default-cdnifci",
  data:           "tag": "2beeac8ee23c3dd1e98a73fd30df80ece9fa5627"},
  data:         { "resource-id": "my-default-networkmap",
  data:           "tag": "7915dc0290c2705481c491a2b4ffbec482b3cf63"}
  data:       ]
  data:     },
  data:     "ipv4:192.0.2.0/24": {
  data:       "my-default-cdnifci.cdni-capabilities": [
  data:         { "capability-type": "FCI.DeliveryProtocol",
  data:           "capability-value": {
  data:             "delivery-protocols": ["http/1.1", "https/1.1"]}}]
  data:     }
  data:   }
  data: }

  event: application/json-patch+json,fci-propmap-stream
  data: [
  data:   { "op": "replace",
  data:     "path": "/meta/dependent-vtags/0/tag",
  data:     "value": "61b23185a50dc7b334577507e8f00ff8c3b409e4"
  data:   },
  data:   { "op": "replace",
  data:     "path":
  data:     "/property-map/countrycode:fr/my-default-networkmap.pid",
  data:     "value": "pid5"
  data:   }
  data: ]
~~~
