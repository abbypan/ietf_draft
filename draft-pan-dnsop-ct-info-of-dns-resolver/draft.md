---
title: Certificate Transparency (CT) information of  DNS resolver
abbrev: EFAS
docname: draft-pan-dnsop-ct-info-of-dns-resolver
date: 2025-09-04

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
  sortrefs:   # defaults to yes
  symrefs: yes

author:
      -
        ins: L. Pan
        name: Lanlan Pan
        #org: 
        # abbrev: TZI
        # street:
        # - Postfach 330440
        # - Bibliothekstr. 1
        #street: 
        city: Guangdong
        #code: D-28359
        country: China
        #phone: 
        #facsimile: 
        email: abbypan@gmail.com
        

normative:
#        - rfc2119
#        - rfc2616
#  RFC1034:
#  RFC1035:
#  RFC2119:
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
      - Dan Goodin
#        ins: 
#        name: 
#        org: ISC
    date: 2025


  IANA-DNS:
    title: Domain Name System (DNS) Parameters
    target: https://www.iana.org/assignments/dns-parameters/
    author:
      - IANA
#        ins: 
#        name: 
#        org: ISC


    

#entity:
#        SELF: "[RFCXXXX]"

# --- note_IESG_Note
#
# bla bla bla

--- abstract

This document describes about the Certificate Transparency (CT) information of the encrypted DNS resolver.

--- middle

Background
==========

DNS resolver can supports any encrypted DNS scheme, such as DNS over HTTPS (DoH) {{RFC8484}}, DNS over TLS (DoT) {{RFC7858}}, or DNS over QUIC (DoQ) {{RFC9250}}.

Certificate hijacking allows attackers to impersonate legitimate encrypted DNS resolver, see also {{MisIssuedCF}}.
Certificate Transparency (CT) is to combat the certificate hijacking issue {{RFC9162}}.

This document describes about the CT information of the encrypted DNS resolver.


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

IANA has created a new registry called "DNS Resolver Information Keys" {{IANA-DNS}}.

This document adds a new DNS Resolver Information Key : CT, to present the CT information of the encrypted DNS resolver. 


Name:
    CT
Value:
    1
Meaning:
    The value indicates that the certifcate of encrypted DNS resolver contains embeded SCTs.
Reference:
    RFC 9162


Name:
    CT
Value:
    2
Meaning:
    The value indicates that the encrypted DNS resolver supports transparency_info TLS extension.
Reference:
    RFC 9162


Security Considerations
=======================================

The DNS clients can received trustable DNS resolver information through DNSSEC or out-of-band configuration. 

If the DNS clients find that CT is supported in the trustable DNS resolver information, they can mandate the CT validation in encrypted communication channel setup process.


Acknowledgements
================

Thanks to all in the DNSOP mailing list.


--- back


--- fluff
