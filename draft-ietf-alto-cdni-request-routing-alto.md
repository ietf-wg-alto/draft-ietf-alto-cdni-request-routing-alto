---
docname: draft-ietf-alto-cdni-request-routing-alto-latest
title: "Content Delivery Network Interconnection (CDNI) Request Routing: CDNI Footprint and Capabilities Advertisement using ALTO"
abbrev: "CDNI FCI using ALTO"
category: std

ipr: trust200902
area: Networks
workgroup: ALTO &amp; CDNI WGs
keyword: ALTO

stand_alone: yes
pi:
  toc: yes
  tocompact: yes
  tocdepth: 3
  iprnotified: no
  sortrefs: yes
  symrefs: yes
  compact: yes
  subcompact: no

author:
  -
    ins: J. Seedorf
    name: Jan Seedorf
    org: HFT Stuttgart - Univ. of Applied Sciences
    street: Schellingstrasse 24
    city: Stuttgart
    code: 70174
    country: Germany
    phone: +49-0711-8926-2801
    email: jan.seedorf@hft-stuttgart.de
  -
    ins: Y. Yang
    name: Y. Richard Yang
    org: Yale University
    street: 51 Prospect Street
    city: New Haven
    code: CT 06511
    country: USA
    phone: +1-203-432-6400
    email: yry@cs.yale.edu
    uri: http://www.cs.yale.edu/~yry/
  -
    ins: K. Ma
    name: Kevin J. Ma
    org: Ericsson
    street: 43 Nagog Park
    city: Acton
    code: MA 01720
    country: USA
    phone: +1-978-844-5100
    email: kevin.j.ma.ietf@gmail.com
  -
    ins: J. Peterson
    name: Jon Peterson
    org: NeuStar
    street: 1800 Sutter St Suite 570
    city: Concord
    code: CA 94520
    country: USA
    email: jon.peterson@neustar.biz
  -
    ins: J. Zhang
    name: Jingxuan Jensen Zhang
    org: Tongji University
    street: 4800 Cao'an Hwy
    city: Shanghai
    code: 201804
    country: China
    email: jingxuan.zhang@tongji.edu.cn

normative:
  ISO3166-1:
    title: "ISO 3166-1: Codes for the representation of names of countries and their subdivisions -- Part 1: Country codes"
    author:
      - name: "ISO (International Organization for Standardization)"
        ins: "ISO (International Organization for Standardization)"
    date: 2020
  I-D.ietf-alto-unified-props-new:
  RFC2119:
  RFC6793:
  RFC7285:
  RFC7493:
  RFC8006:
  RFC8008:
  RFC8259:
  RFC8174:
  RFC8895:
informative:
  I-D.ietf-alto-path-vector:
  RFC5693:
  RFC6707:
  RFC7975:

--- abstract

{::include abstract.md}

--- note_Requirements_Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}}{{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

--- middle

{::include introduction.md}
{::include background.md}
{::include specification.md}
{::include others.md}

--- back
