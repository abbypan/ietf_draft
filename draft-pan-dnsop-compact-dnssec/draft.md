---
title: Compact DNSSEC
abbrev: EFAS
docname: draft-pan-dnsop-compact-dnssec-00
date: 2025-02-23

# stand_alone: true

stream: IETF
ipr: trust200902
area: ops
wg: dnsop
kw: Internet-Draft
cat: info

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
  RFC2119:
  RFC1034:
  RFC1035:
  RFC8499:
  RFC9539:
  RFC9250:
  RFC7858:
#  RFC8914:
#  RFC8484:
#  RFC2308:
#  RFC4592:
#  RFC7515:
#  RFC8020:
#  RFC8767:
#  RFC8198:
#  RFC9520:

informative:

  SadDNSSEC:
    title: The Sad Story of DNSSEC
    target: https://alexkelliott.github.io/dnssec/TheSadStoryOfDNSSEC.pdf
    author:
      -
        name: Elliott, A. and Moxley, J.

  AmpDNSSEC:
    title: DNSSEC fuels new wave of dns amplification.
    target: https://www.nexusguard.com/blog/dnssec-fuels-new-wave-of-dns-amplification 
    author:
      -
        name: Nexusguard

  DNSCurve:
    title: DNSCurve
    target: https://dnscurve.org/
    author:
      -
        name: Bernstein, D. J.

#entity:
#        SELF: "[RFCXXXX]"

# --- note_IESG_Note
#
# bla bla bla

--- abstract

This document describes about a compact DNSSEC scheme for resource-limited second-level domain (SLD), which is focused on NS RR.

--- middle

Background
============

DNSSEC has low adoption rate on SLD {{SadDNSSEC}}.

The operation burden of fullzone DNSSEC deployment is heavy.

DNS random subdomain attacks and amplification attacks are commonly used distributed denial-of-service (DDoS) attacks.
The DDoS amplification power of the authoritative server of SLD will be larger after deploying DNSSEC {{AmpDNSSEC}}.


Terminology
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

Basic terms used in this specification are defined in the documents {{RFC1034}}, {{RFC1035}}, {{RFC8499}}.

* Authoritative Server: Described in {{RFC8499}}.

* Recursive Resolver: Described in {{RFC8499}}. 


Compact DNSSEC Scheme
=============================

To encourge the DNSSEC deployment on resource-limited SLD, it is resonable to give it a compact DNSSEC deployment scheme.

The Resource-limited SLD Publishes Compact DNSSEC Records
-----------------------------------------------------------

Resource-limited SLD should publish these DNSSEC records: 

* the delegation signer (DS) record on TLD.

* the DNSKEY records. 

* the RRSIGs for NS/A/AAAA/CNAME/TLSA records associated with NS.


Resource-limited SLD doesn't publish other DNSSEC records on other subdomains.

Resource-limited SLD doesn't deploy NSEC/NSEC3.

For example:

        example.com. 345600 IN NS ns1.example.com.
        example.com. 345600 IN NS ns2.example.com.
        ns1.example.com. 345600 IN A 11.22.33.44
        ns1.example.com. 345600 IN AAAA ::11.22.33.44
        ns2.example.com. 345600 IN A 55.66.77.88
        ns2.example.com. 345600 IN AAAA ::55.66.77.88
        _853._tcp.ns1.example.com. 3600 IN TLSA ( 3 1 1 
          63cbfcafa3284cc46b1676a99dbc09d8acadf9050cf876de79ac1e5776bbd364 )
        _853._udp.ns1.example.com. 3600 IN TLSA ( 3 1 1
          63cbfcafa3284cc46b1676a99dbc09d8acadf9050cf876de79ac1e5776bbd364 )
        _853._tcp.ns2.example.com. 3600 IN TLSA ( 3 1 1
          63cbfcafa3284cc46b1676a99dbc09d8acadf9050cf876de79ac1e5776bbd364 )
        _853._udp.ns2.example.com. 3600 IN TLSA ( 3 1 1
          63cbfcafa3284cc46b1676a99dbc09d8acadf9050cf876de79ac1e5776bbd364 )


Therefore, the zone file size of the compact DNSSEC scheme is approximate with plain-text DNS, with few RRSIGs.


The Authoritative Server of Resource-limited SLD Deploys Secure Service
==============================================================================

{{RFC7858}} and {{RFC9250}} defined the encrypted DoT/DoQ service for client-to-recursive.

{{RFC9539}} discussed the extended deployment of encrypted recursive-to-authoritative DNS.

The authoritative server of resource-limited SLD deploys the DoQ/DoT service with self-signed PKI cerificate with TLS connection. 

* The NS records of the resource-limited SLD should be written into the subjectAltName extension field of the self-signed PKI certificate.

* The public key information of the self-signed PKI cerificate is published on associated TLSA records of the NS.

* The associated TLSA records are DNSSEC-signed.

An alternative secure channel solution is {{DNSCurve}}, embeded the raw public key into the NS records.


The Recursive Resolver Validates The Compact DNSSEC Records
==================================================================

The recursive resolver validates the DNSSEC trust chain (Root -> TLD -> SLD), and gains the trustworthy A/AAAA records of the NS records of the SLD.

The trustworthy A/AAAA records are the IP addresses of the authoritative server of the resource-limited SLD. 


Setup Secure Channel for Recursive-to-Authoritative
=======================================================================================================

The Recursive Resolver setup secure DoQ/DoT channel with the authoritative server of the resource-limited SLD:

* The recursive resolver connects to the trustworthy IP addresses of the authoritative server of the resource-limited SLD. 

* The recursive resolver receives the self-signed certificate from the authoritative server, and extract the public key from the self-signed PKI certificate. 

* The recursive resolver validates the TLSA RRSIGs of the NS records of the SLD with the DNSSEC trust chain. 

* The recursive resolver validates the digest of extracted public key match the TLSA record.

* The recursive resolver setup secure DoQ/DoT channel with the authoritative server of the resource-limited SLD successfully. 

* The recursive resolver make DNS query with the authoritative server of the resource-limited SLD on the secure DoQ/DoT channel.



Security Considerations
==========================

The compact DNSSEC scheme does not cover the entire zone and does not deploy NSEC/NSEC3.


--- back


--- fluff
