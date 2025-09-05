---
title: Certificate Transparency (CT) information of DNS resolver
abbrev: EFAS
docname: draft-pan-dnsop-ct-info-of-dns-resolver
date: 2025-09-05

# stand_alone: true

stream: IETF
ipr: trust200902
area: ops
wg: dnsop
kw: Internet-Draft
cat: bcp

coding: utf-8
pi:    # can use array (if all yes) or hash here
#  - toc
#  - sortrefs
#  - symrefs
  toc: yes
  sortrefs:
  symrefs: yes

author:
      -
        ins: L. Pan
        name: Lanlan Pan
        street: 
        city: Guangdong
        region:
        code:
        country: China
        email: abbypan@gmail.com
        

normative:
   RFC1034:
   RFC1035:
   RFC2119:
#  RFC2308:
#  RFC4592:
#  RFC7515:
   RFC7858:
#  RFC8020:
#  RFC8198:
   RFC8484:
#  RFC8767:
#  RFC8914:
   RFC9162:
   RFC9250:
   RFC9499:
#  RFC9520:
   RFC9606:

informative:
  MisIssuedCF:
    title: Mis-issued certificates for 1.1.1.1 DNS service pose a threat to the Internet
    target: https://arstechnica.com/security/2025/09/mis-issued-certificates-for-1-1-1-1-dns-service-pose-a-threat-to-the-internet/
    author:
      - 
         ins: 
         name: Dan Goodin
         org: 
    date: 2025


  IANA-DNS:
    title: Domain Name System (DNS) Parameters
    target: https://www.iana.org/assignments/dns-parameters/
    author:
      - 
         ins: 
         name: IANA
         org: 


    

#entity:
#        SELF: "[RFCXXXX]"

# --- note_IESG_Note
#
# bla bla bla

--- abstract

This document describes the Certificate Transparency (CT) information of the DNS resolver.

--- middle

Background
==========

DNS resolver can support any encrypted DNS scheme, such as DNS over HTTPS (DoH) {{RFC8484}}, DNS over TLS (DoT) {{RFC7858}}, or DNS over QUIC (DoQ) {{RFC9250}}.

Certificate hijacking allows attackers to impersonate a legitimate encrypted DNS resolver, see also {{MisIssuedCF}}.

Certificate Transparency (CT) is to combat the certificate hijacking issue {{RFC9162}}.
This document describes the CT information of the encrypted DNS resolver.


Terminology
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

Basic terms used in this specification are defined in the documents {{RFC1034}}, {{RFC1035}}, {{RFC9499}}, {{RFC9606}}, {{RFC9162}}.

IANA Considerations
======================

DNS Resolver Information Keys Registration
---------------------------------------------

{{RFC9606}} specifies a method for DNS resolvers to publish information about themselves.

IANA has created a new registry called "DNS Resolver Information Keys" {{IANA-DNS}}.

This document adds a new DNS Resolver Information Key: CT, to present the CT information of the encrypted DNS resolver. 


Name:
    CT

Value:
    1

Meaning:
    The value indicates that the certificate of the encrypted DNS resolver contains embedded SCTs.

Reference:
    RFC 9162


Name:
    CT

Value:
    2

Meaning:
    The value indicates that the encrypted DNS resolver supports the transparency_info TLS extension.

Reference:
    RFC 9162


Security Considerations
=======================================

DNS clients can get trustworthy DNS resolver information through DNSSEC query or out-of-band configuration.

Suppose the DNS clients find the CT value in the trustworthy DNS resolver information. In that case, they can mandate the CT validation in the encrypted communication channel setup process with the encrypted DNS resolver.


Acknowledgements
================

Thanks to all in the DNSOP mailing list.


--- back


--- fluff
