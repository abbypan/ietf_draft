---
title: Explicit Forged Answer Signal
abbrev: EFAS
docname: draft-pan-dnsop-explicit-forged-answer-signal-00
date: 2024-01-10

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
  RFC8914:
  RFC8484:
#  RFC2308:
#  RFC4592:
#  RFC7515:
#  RFC8020:
#  RFC8767:
#  RFC8198:
#  RFC9520:

informative:
  NXRedierct:
    title: NXDOMAIN Redirection Using DLZ in BIND 9.10 and later
    target: https://kb.isc.org/docs/aa-01150
    author:
      -
#        ins: 
#        name: 
        org: ISC
#    date: 2000

  ISPRedirect:
    title: DNS Redirect Use by Service Providers
    target: https://datatracker.ietf.org/doc/draft-livingood-dns-redirect/
    author:
      -
        name: J. Livingood, R. Weber
    
  DNSFirewall:
    title: Response Policy Zones (RPZ)
    target: https://www.isc.org/rpz/
    author:
      -
        org: ISC

  LegalRedirect:
    title: Oups! French Government Mistakenly Blocks Telegram Access for Millions
    target: https://pulse.internetsociety.org/blog/oups-french-government-mistakenly-blocks-telegram-access-for-millions
    author:
      -
        name: Dan York
    
  NXDamageControl:
    title: What DNS Is Not
    target: https://queue.acm.org/detail.cfm?id=1647302
    author:
      -
        name: Paul Vixie
    
  NXDNSLies:
    title: NXDOMAIN?
    target: https://www.potaroo.net/ispcol/2009-12/nxdomain.pdf
    author:
      -
        name: Geoff Huston

#entity:
#        SELF: "[RFCXXXX]"

# --- note_IESG_Note
#
# bla bla bla

--- abstract

This document describes that recursive resolver should give explict signal in the forged answer.

Client could react more clearly based on the explict forged answer signal, to protect user on security and privacy.

--- middle

Background and Motivation
=========================

Recursive server may replace a forged answer to a query with a configured answer of the authoritative server
in some specific scenarios, 
such as NXDOMAIN, phishing, fraud, malware, ransomware, botnet DDoS attack, and legal requirement, etc.
See also {{NXRedierct}} {{ISPRedirect}} {{DNSFirewall}} {{LegalRedirect}}.

The RCODE of faked answer is NOERROR, which make client hard to distinguish it with honest answer, if client doesn't make iterative dns query by itself, or make DNSSEC validation.

At least, the client has the right to know that it has received a forged answer and it could make clearer reaction by itself.


Terminology
===========

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

Basic terms used in this specification are defined in the documents {{RFC1034}}, {{RFC1035}}, {{RFC8499}}.

* Authoritative Server: Described in {{RFC8499}}.

* Recursive Resolver: Described in {{RFC8499}}. 


Attack Surface
==============

Faked answer can avoid user to visit malicious website, however, it may also increase the security and privacy risk.

HTTP Cookies Leakage
--------------------

The HTTP cookies risk has been well discussed in {{NXDamageControl}} and {{NXDNSLies}}. Furthermore, the risk is not only occured on NXDOMAIN scenario, but also on other faked answer scenarios.

Imagine that user visits "abc.example.com" in browser.

Recursive resolver return a faked answer to browser.

Browser will visit the faked server, and leak the HTTP cookies in "example.com" of the user to it.

With the leaked HTTP cookies, the faked server may pretend as the user to visit "abc.example.com", result in user's security issue and privacy leakage.


Explicit Forged Answer Signal
=============================

Recursive resolver should give explict forged answer signal to client. 

Format 1: Use Extended DNS Errors
---------------------------------

{{RFC8914}} defined Extended DNS Errors (EDE) extension.

Recursive resolver could give the signal by include additional EDE information in DNS response: 

* INFO-Code is 4. 

* EXTRA-TEXT is the specific scenario desciption, for example, malware.


Format 2: Use TXT RR
--------------------

{{RFC1035}} defined TXT RDATA format.

Recursive resolver could give the signal by include additional TXT RR in DNS response, such as: 

    abc.example.com  300 IN  A  1.2.3.4
    abc.example.com  300 IN  TXT  "faked=malware" 


Client Reaction
===============

Client could make its own reaction when it received an explict forged answer signal from recursive resolver. 

Reaction 1: Use DNSSEC
----------------------

Client could make DNSSEC query by itself. 

If the domain has deployed DNSSEC, the client could validate the honest answer from authoritative server.


Reaction 2: Change Recursive Resolver
-------------------------------------

Client could change to another recursive resolver which is not lying.


Reaction 3: Stop Visit
----------------------

Client could stop to visit on the website, since it knows that the answer is faked.


Reaction 4: Limited Visit
-------------------------

Client could make limited visit on the website, prevent HTTP cookies from being send to the faked server.

For example, browser should not send user's HTTP cookies to the faked server, if it gets an explict faked answer signal in the DoH response {{RFC8484}}.


Security Considerations
=======================

Faked answer is unauthenticated by authoritative server, just offered by recursive resolver on some specific scenarios.

Ideally, with the DNSSEC deployed on second level domain, client would not trust any faked answer if it makes all RRSIG validation by itself.

Explicit faked answer signal is to help client to make clearer reaction on faked answer, with the help of recursive resolver.

As a trade-off, explict faked answer signal could help browser to mitigate the http cookies leaked to faked server, protect user security and privacy in conditional limited environment.


Acknowledgements
================

Thanks to all in the DNSOP mailing list.


--- back


--- fluff

<!--  LocalWords:  CoAP datagram CoRE WG RESTful IP ETag reassembler
-->
<!--  LocalWords:  blockwise idempotence statelessly keepalive SZX
-->
<!--  LocalWords:  acknowledgement retransmissions ACKs ACK untrusted
-->
<!--  LocalWords:  acknowledgements interoperability retransmission
-->
<!--  LocalWords:  BCP atomicity NUM WebDAV IANA
-->
