---
title: ISP Location in DNS Queries
abbrev: EIL
docname: draft-pan-dnsop-edns-isp-location-06
date: 2024-01-15

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
      -
        ins: Y. Fu
        name: Yu Fu
        org: China Unicom
#         abbrev: 
        # street:
        # - 
        # - 
        #street: 
        city: Beijing
        #code: D-28359
        country: China
        #phone: 
        #facsimile: 
        email: fuy186@chinaunicom.cn
      -
        ins: C. Wang
        name: Cuicui Wang
        org: China Unicom
#         abbrev: 
        # street:
        # - 
        # - 
        #street: 
        city: Beijing
        #code: D-28359
        country: China
        #phone: 
        #facsimile: 
        email: wangcc107@chinaunicom.cn

normative:
  RFC2119:
  RFC1034:
  RFC1035:
  RFC1700:
  RFC6891:
  RFC8499:
  RFC7871:

informative:
  BIND-GeoIP:
    title: Using the GeoIP Features in BIND 9.10
    target: https://kb.isc.org/article/AA-01149/0
    author:
      -
#        ins: 
#        name: 
        org: ISC
#    date: 2000

  PowerDNS-GeoIP:
    title: PowerDNS GeoIP backend
    target: https://doc.powerdns.com/md/authoritative/backend-geoip/
    author:
      -
#        name: J. Livingood, R. Weber
        org: PowerDNS
    
  Amazon-GeoIP:
    title: Amazon Route 53 Geolocation Routing
    target: http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-geo
    author:
      -
        org: Amazon

  DYN-GeoIP:
    title: Understanding How Traffic Director Makes Decisions
    target: https://help.dyn.com/understanding-td-decisions/
    author:
      -
        org: DYN
    
  gdnsd-GeoIP:
    title: GdnsdPluginGeoip
    target: https://github.com/gdnsd/gdnsd/wiki/GdnsdPluginGeoip
    author:
      -
        name: gdnsd
    
  WindowsServer-GeoIP:
    title: Use DNS Policy for Geo-Location Based Traffic Management with Primary Servers
    target: https://docs.microsoft.com/en-us/windows-server/networking/dns/deploy/primary-geo-location
    author:
      -
        org: Microsoft

  ISO3166:
    title: Country Codes
    target: http://www.iso.org/iso/country_codes
    author:
      -
        org: ISO

  ClientSubnet-Bis:
    title: CLIENT-SUBNET bis appetite?
    target: https://www.ietf.org/mail-archive/web/dnsop/current/msg21616.html
    author:
      -
        name: Mukund Sivaraman
    
  ECS-Privacy:
    title: Understanding the Privacy Implications of ECS
    target: https://link.springer.com/chapter/10.1007/978-3-319-40667-1_17
    author: 
      -
        name: Panagiotis Kintis

  EIL-PST:
    title: Mitigating Client Subnet Leakage in DNS Queries
    target: https://ieeexplore.ieee.org/document/8514164
    author: 
      -
        name: Lanlan Pan

  EIL-Qshine:
    title: Improving Privacy for GeoIP DNS Traffic
    target: https://eudl.eu/doi/10.1007/978-3-030-14413-5_1
    author: 
      -
        name: Lanlan Pan

#entity:
#        SELF: "[RFCXXXX]"

# --- note_IESG_Note
#
# bla bla bla

--- abstract

Nowadays, many authoritative servers support GeoIP feature, 
they guess the client's geolocation by the client subnet of EDNS Client Subnet (ECS) or by the source IP address of DNS query, return tailor DNS response based on the client's geolocation. 
However, ECS raises some privacy concerns because it leaks client subnet information on the resolution path to the authoritative server.

This document describes an improved GeoIP solution,
defines an EDNS ISP Location (EIL) extension to address the privacy problem of ECS, 
tries to find the right balance between privacy improvement and user experience optimization.

EIL is defined to convey isp location \< COUNTRY, AREA, ISP \> information that is relevant to the DNS message. 
It will directly provide sufficient information for the GeoIP-enabled authoritative server as ECS, decide the response without guessing client's geolocation.

--- middle

Introduction
============

Nowadays, many authoritative servers support GeoIP feature, such as {{BIND-GeoIP}}, {{PowerDNS-GeoIP}}, {{Amazon-GeoIP}}, {{DYN-GeoIP}}, {{gdnsd-GeoIP}}, {{WindowsServer-GeoIP}} (More details are given in Appendix A). These geographically aware authoritative servers guess the client's geolocation by the client subnet of ECS or by the source IP address of DNS query, return tailor DNS response based on the client's geolocation.

ECS is an EDNS0 option {{RFC6891}}, described in {{RFC7871}}, carries client subnet information in DNS queries for authoritative server. Compared to source IP address of DNS query, ECS will help authoritative server to guess the client's geolocation more precisely because of the DNS forwarding query structure.

GeoIP-enabled authoritative servers use ECS for client geolocation detecting. However, ECS raises some privacy concerns because it leaks client subnet information on the resolution path to the authoritative server {{ECS-Privacy}}.

This document describes an improved GeoIP solution,
defines an EDNS ISP Location (EIL) extension to address the privacy problem of ECS, 
tries to find the right balance between privacy improvement and user experience optimization.

EIL is defined to convey isp location < COUNTRY, AREA, ISP > information that is relevant to the DNS message. 
It will directly provide the same sufficient information for the GeoIP-enabled authoritative server as ECS, decide the response without guessing client's geolocation.

EIL is intended for those local forwarding resolvers, recursive resolvers and authoritative servers that would benefit from the extension and not for general purpose deployment. EIL could be applied for tailor DNS response for GeoIP scenario. EIL can safely be ignored by servers that choose not to implement or enable it.


# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in {{RFC2119}} when they appear in ALL CAPS. When these words are not in ALL CAPS (such as "should" or "Should"), they have their usual English meanings, and are not to be interpreted as {{RFC2119}} keywords.


Basic terms used in this specification are defined in the documents {{RFC1034}}, {{RFC1035}}, {{RFC8499}} and {{RFC7871}}.

* EIL: EDNS ISP Location.

* ECS: EDNS Client Subnet, described in {{RFC7871}}.

* Stub Resolver: A resolver that cannot perform all resolution itself.

* Authoritative Server: It is a server that knows the content of a DNS zone from local knowledge, and thus can answer queries about that zone without needing to query other servers.

* Intermediate Nameserver: Any nameserver in between the stub resolver and the authoritative server, such as a recursive resolver or a forwarding resolver.

* Local Forwarding Resolver: It is the first forwarding resolver which receives DNS queries from stub resolver, usually deployed nearby the first-hop router such as public Wi-Fi hotspot routers and home routers.

* Recursive Resolver: It is the last-hop before authoritative server in the DNS query path.



# Problem of ECS

As mentioned in {{RFC7871}}'s abstract section, since ECS has some known operational and privacy shortcomings, a revision will be worked through the IETF for improvement.

## Client

Common clients have little power to defense passive monitoring, expecially in the plain-text traffic.

ECS's client subnet leakage has rise some user privacy concerns.

## Recursive Resolver

Recursive resolver must deal with ECS's cache problem, such as low cache hitrate, rise response time, redundant cache size, etc. 

Mukund Sivaraman described some scenarios in {{ClientSubnet-Bis}}.

ECS is precise because it is based on client subnet. But IPv6 addresses will boom, we can foresee it to increase more burden on global recursive resolvers.

## GeoIP-enabled Authoritative Server

Tranditional recursive resolver's IP can on behalf of many client subnets because they are network topological close.
But this scenario has been varied by public recursive resolver. ECS push client subnets to authoritative server, wants to solve the "public recursive resolver's IP is network topological far from client subnet" problem.

Therefore, ECS rises GeoIP-enabled authoritative server's dependence on IP2Geo database quality, because authoritative server should guess geolocation for huge amounts of client subnet.
Every GeoIP-enabled authoritative server must operate IP2Geo database carefully and catch up with network topology change. 
The work is inevitable, but ECS aggravate this, because the number of client subnets is far greater than the number of recursive resolvers. 
GeoIP-enabled authoritative server needs a more precise IP2Geo database, updates it more frequent than before, to catch up with the huge client subnet network topology, but not the recursive resolver's IP network topology.
Every GeoIP-enabled authoritative server should cost more on IP2Geo database.


# EIL Overview

EIL is an EDNS0 option to allow local forwarding resolvers and recursive resolvers, if they are willing, to forward details about the isp location of client when talking to other nameservers. EIL can be added in queries sent by local forwarding resolvers or recursive resolvers in a way that is transparent to stub resolvers and end users.

Authoritative servers could provide a better answer by using precise isp location in EIL. Intermediate Nameservers could send EIL query and cache the EIL response. This document also provides a mechanism to signal Intermediate Nameservers that they do not want EIL treatment for specific queries.

EIL is only defined for the Internet (IN) DNS class.

## The EIL EDNS0 option

The EIL is an EDNS0 option to include the < COUNTRY, AREA, ISP > isp location of client in DNS messages.

It is 16 octets which is structured as follows:

                    +0 (MSB)                            +1 (LSB)
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       0: |                         OPTION-CODE                           |
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       2: |                         OPTION-LENGTH                         |
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       4: |                         COUNTRY                               |
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
       6: |                         AREA                                  |
          |                                                               |
          |                                                               |
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
      12: |                         ISP                                   |
          |                                                               |
          +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

          Total: 16 octets.

   *  OPTION-CODE, 2 octets, defined in {{RFC6891}}. EDNS option code should be assigned by the IANA.

   *  OPTION-LENGTH, 2 octets, defined in {{RFC6891}}, contains the length of the payload (everything after OPTION-LENGTH) in octets.

   *  COUNTRY, 2 octets, uppercase, defined in {{ISO3166}}, indicates the country information of the client's IP. For example, China's COUNTRY is CN.

   *  AREA, 6 octets, uppercase, defined in {{ISO3166}} country subdivision code, indicates the area information of the client's IP. For example, The AREA of Fujian Province in China is 35.

   *  ISP, 4 octets, uppercase, indicates the ISP information of the client's IP, using shortcut names. ISP shortcut names are unique within the context of the COUNTRY. For example, the shortcut name of China Telecommunications Corporation is TEL, the shortcut name of China United Network Communications is UNI, the shortcut name of China Mobile is MOB, etc.

All fields are in network byte order ("big-endian", per {{RFC1700}}, Data Notation).

The aim to use shortcut names in the ISP field is to limit the data size of EIL, decrease the DDoS risk.

The null value 0x20 signifies that the field is unknown. If all fields in EIL are set to null value, it means that client doesn't want to use EIL.

Authoritative servers can send EIL response with the * value 0x2A in AREA field or ISP field (not COUNTRY field), which signifies that the field is wildcard match. For example,  < CN, *, TEL > indicates "all area in China, Telecom ISP", < CN, *, * > indicates "all area in China".


# Protocol Description

## Originating the Option

The EIL can be initialized by public recursive resolver, ISP recursive resolver, or local forwarding resolver.

Examples are given in Appendix B.

### P-Model: Public Recursive Resolver

Public recursive resolvers are not close to many clients because the service providers couldn't deploy servers in every country and every ISP's network, which will affect the response accuracy of authoritative servers. 
To address this problem, ECS shifts the client subnet information to authoritative server, but rises user privacy concerns.

Therefore, to keep balance between precise and privacy, when a public recursive resolver receives a DNS query, it can guess isp location of client's IP and generate the EIL OPT data, then send EIL query to the authoritative server. This will move the "guess location of client's IP" work from authoritative server back to public recursive resolver, lighten the burden of authoritative server, but increase DDoS risk on public recursive resolver.

In order to improve the user's privacy, if a recursive resolver receives a DNS query with ECS, it can guess the isp location of SOURCE-PREFIX from the ECS OPT data, and make a new DNS query with EIL, then send the query to authoritative server which supports EIL.

P-model is the most recommended and close to the ECS.

### I-Model: ISP Recursive Resolver

ISP recursive resolver only serves its customers, each of whom has a static isp location. ISP recursive resolver can add EIL transparent to end client, and then authoritative server doesn't need to "guess location of client's IP".

EIL will be benefit if the authoritative server could not find the approximate isp location of ISP recursive resolver, which is crucial to DNS response accuracy in ECS.

### L-Model: Local Forwarding Resolver

Local forwarding resolver is usually on the first-hop router, such as public Wi-Fi hotspot routers and Cisco/Linksys/Netgear/TP-LINK home routers.

When a local forwarding resolver that implements EIL receives a DNS query from an end client, it surely can know about the isp location of client's IP, and generate the EIL OPT data, then send the EIL query to the recursive resolver. Recursive resolver sends the EIL query to the authoritative server.

In this scenario, both public recursive resolver and authoritative server don't need to "guess location of client's IP", 
because the local forwarding resolver supplies the isp location precisely. 
That is, EIL can reduce the dependence on the IP geolocation database quality, which is crucial to DNS response accuracy in ECS.

If a local forwarding resolver had sent a query with EIL, and receives a REFUSE response, it MUST regenerate a query with no EIL.

## Generating a Response

### Path Calculation and Tailored DNS Response

Separate the consideration of path calculation (data provider) and tailored DNS response (authoritative server).

Data providers make path calculations to optimize content delivery on the Internet based on the network topology, considering many factors such as IP, RIPs, FIBs, AS Path hops, system load, content availability, path latency, etc. Note that, data providers have the full details of the clients, they can make any complex path calculations without ECS and EIL.
Data Providers can make path calculations based on network topology, decide network topological close datacenter for each IP address.

Authoritative servers allocate tailored DNS response to each IP address based on the "network topological close" result of path calculations.
Based on the result of path calculation, data providers could build up their GeoIP configuration for their domains. Usually, clients from the same < COUNTRY, AREA, ISP > isp location are allocated to the same best "network topologically close" target ip addresses. For example, client IP addresses from < China, Beijing, Telecom > are allocated to Target-IP-addresses-1 (ip1, ..., ip4), client IP addresses from < China, Beijing, Unicom > are allocated to Target-IP-addresses-2 (ip5, ..., ip8), etc. 

Data providers publish their GeoIP configuration to Authoritative servers. Authoritative servers load the GeoIP configuration, and return the GeoIP-based tailored DNS response based on client's geolocation.
If the GeoIP-enabled authoritative servers support ECS, they can use the client subnet information of ECS for client's geolocation detecting.
Alternative, if the GeoIP-enabled authoritative servers support EIL, they can use the < COUNTRY, AREA, ISP > information of EIL directly, without client's geolocation detecting.

EIL tell authoritative server like that, "I want to know what is best IP address for clients from < China, Beijing, Telecom > at network topology path calculations result", but not "I want to know what is the nearest IP address for clients from < China, Beijing, Telecom > at physical topology path calculations result".

EIL is satisfied if authoritative servers aggregate the IP addresses from the same < COUNTRY, AREA, ISP > isp location to visit the same datacenters, we call that GeoIP-based tailored DNS responses, and these tailored responses have the best "network topological close" distance to the clients which are generated from network topology path calculations result.

ECS is satisfied if authoritative servers make tailored DNS response down to subnet precise level. For example, (subnet-1, ..., subnet-10) are from the same < COUNTRY, AREA, ISP > isp location, Data Provider can apply (subnet-1, ..., subnet-5) visit Target-IP-addresses-1 (ip1, ..., ip2), (subnet-6, ..., subnet-10) visit Target-IP-addresses-1 (ip3, ..., ip4).

### Whitelist

EIL contains a whitelist for < COUNTRY, AREA and ISP >, which can be discussed and maintained by the DNSOP working group. 
Authoritative servers that supporting EIL must only response the EIL queries matched the whitelist. 
Recursive resolver that supporting EIL must only cache the EIL responses matched the whitelist.

### Authoritative Server

Using the < COUNTRY, AREA and ISP > isp location specified in the EIL option of DNS query, an authoritative server can generate a tailored response.

Authoritative servers that have not implemented or enabled support for the EIL ought to safely ignore it within incoming queries, response the query as a normal case without EDNS0 option. Such a server MUST NOT include an EIL option within replies to indicate lack of support for it.

An authoritative server that has implemented this protocol and receives an EIL option MUST include an EIL option in its response to indicate that it SHOULD be cached accordingly.

An authoritative server will return a more appropriate tailored response for the query with an EIL option containing more precisely AREA.

### Intermediate Nameserver

Like ECS, intermediate nameserver passes a DNS response with an EIL option to its client when the client indicates support EIL.

If an intermediate nameserver receives a response that has a larger area than the AREA provided in its query, it SHOULD still provide the result as the answer to the triggering client request even if the client is in a smaller area.

## Handling EIL Responses and Caching

If an intermediate nameserver had sent a query with EIL, and receives a NOERROR response without EIL option, it SHOULD treat this answer as suitable for all clients.

Other handling considerations are similar with {{RFC7871}}, SECTION 7.3.

In the cache, all resource records in the answer section MUST be tied to the isp location specified in the response. The answer section is valid for all areas which the EIL option covered. For example, an EIL option < CN, 35, TEL > covers all 9 cities in Fujian province of China Telecommunications ISP.

Same with ECS, the additional and authority sections are excluded.

Enabling support for EIL in an intermediate nameserver will increase the size of the cache, and prevent "client subnet leak" privacy concern of ECS.

### Answering from Cache

Cache lookups are first done as usual for a DNS query, using the query tuple of < name, type, class >. Then, the appropriate RRset MUST be chosen based on the isp location matching.

If there was an EIL option, the intermediate nameserver will lookup for < same COUNTRY, same AREA, same ISP > of the same query tuple in the cache.

If no EIL option was provided, the safest choice of the intermediate nameserver is dealing the query as a normal case without EDNS0 option.

If no EIL option was provided, but the intermediate nameserver want to be more aggressive, it can guess the isp location from the source IP of the query, then respond as if there was an EIL option with the guessed information. Clients can be benefit when the intermediate nameserver has a more precise IP location database than the authoritative server, especially in global public DNS service like GoogleDNS(8.8.8.8).

Otherwise, if no matching is found, the intermediate nameserver MUST perform resolution as usual.

### Delegations and Negative Answers

EIL's delegation case is similar with ECS, Additional and Authority Sections SHOULD ignore EIL.

For negative answers, authoritative servers return traditional negative answers without EIL.

## Deploy

### Transitivity
 
EIL's transitivity concerns are similar with ECS.

Name servers should only enable EIL where it is expected to benefit the end clients, such as dealing with some latency-sensitive CDN domain queries in a complex network environment.

### Compatibility with non-EDNS and ECS

GeoIP-enabled authoritative servers can simply add EIL support.

Recursive Resolvers can add EIL support, and make the compatible policy with ECS and EIL. 

The indicator that authoritative servers used to generate tailor response is showed as follows:

* RRIP: Recursive Resolver's IP

* ECS: Client Subnet

* EIL: Client Isp Location


        +--------------------+----------------------=-------------------------------------+
        | Recursive Resolver |                                    AUTH                    |
        |                    | non-EDNS | ECS but non-EIL | EIL but non-ECS | ECS and EIL |
        +--------------------+----------+-----------------+-----------------+-------------+
        | non-EDNS           | RRIP     | RRIP            | RRIP            | RRIP        | 
        +-------------------------------+-----------------+-----------------+-------------+
        | ECS but non-EIL    | RRIP     | ECS             | RRIP            | ECS         |
        +-------------------------------+-----------------+-----------------+-------------+
        | EIL but non-ECS    | RRIP     | RRIP            | EIL             | EIL         |
        +--------------------+----------+-----------------+-----------------+-------------+
        | ECS and EIL        | RRIP     | ECS             | EIL             | ECS/EIL     |
        +--------------------+----------+-----------------+-----------------+-------------+
        

### Intermediate Servers Support ECS and EIL at the Same Time

Intermediate nameservers can support ECS and EIL at the same time. However, ECS and EIL can't be both initiated at the same DNS packet. 

To make more effort to protect user's privacy, we suggest that intermediate nameservers could initiate EIL query prior to ECS query. Alternative, they could send both ECS and EIL queries if not match in the cache.

    Receive EIL query: 
        Search in EIL cache.
        If cache is matched, return EIL response.
        Otherwise, 
            Send EIL query.

    Receive ECS query: 
        Search in ECS cache.
        If cache is matched, return ECS response.
        Otherwise, 
            Send ECS query.

    Receive plain DNS query without EDNS option: 
        Search in ECS cache.
        If cache is matched, return ECS response.
        Otherwise,  
            Guess the isp location information of the client's IP, 
            Search in EIL cache.
            If cache is matched, return EIL response.
            Otherwise, 
                Send EIL query.
                If authoritative server supports EIL, return EIL response.
                Otherwise, 
                    Send ECS query.
                    If authoritative server supports ECS, return ECS response.
                    Otherwise, 
                        Send plain DNS query.


    Receive plain DNS query with not-ECS/not-EIL option: 
        Search in not-EDNS cache.
        If cache is matched, return response.
        Otherwise, 
            Send plain DNS query.

    Receive ECS query, improve user privacy with EIL: 
        Guess the isp location information of the client's IP, 
        Search in EIL cache.
        If cache is matched, return EIL response RR with origin ECS option.
        Otherwise, 
             Send EIL query.

## Why not use AS number to build EIL

AS number is not an ideal object to balance between response accuracy and user privacy, for example:

* User privacy: AS24151 can directly guide to China Internet Network Infomation Center, it is not good for user privacy.

* Response accurancy: AS4134 contains a huge amount of IP prefixes whose geolocation covers from South China to North China, AS number can not afford the response accuracy consideration.

Maybe < COUNTRY-CODE, AREA-CODE, ISP, AS-NUMBER > is a considerable trade-off choice.

# Benefit and Cost

## Client

EIL is transparent to client.

EIL is to help mitigate client subnet leakage on the resolution path, improve user privacy.

## Recursive Resolver

ECS sends the query with client subnet, which means that recursive resolvers have to send a new query to authoritative servers with client_subnet_b, even it has known the response about network topological close client_subnet_a. In fact, thousands of subnets visit only a few servers, there are many redundacy queries, the recursive's cache hitrate is low.

Because of ECS's low cache hitrate, recursive resolvers's ECS tailored response latency will be longer, the average of response time will rise with the redundacy queries rate from recursive resolvers to authoritative servers.

Recursive resolver's ECS cache size grows up with the number of client subnets, see also {{EIL-Qshine}} and {{EIL-PST}}.

To sum it up, above problems all rise with the client subnet amount, especially when IPv6 addresses boom. Extend the subnet range in the ECS response may be mitigating, but not work for wide range client subnets. Recursive can make some guess optimization, if it has known response for client_subnet_a, then guess to return the same response for toplogical close client_subnet_b without send the redundancy query.

Therefore, if the ECS revision wants to make more effective client subnets aggregation for recursive resolver, then EIL can be an considerable choice.
EIL wants to summary network toplogical close client subnets into < COUNTRY, AREA, ISP > for GeoIP-enabled authoritative server.
With EIL response cache, recursive resolvers can directly response for many ECS client subnets queries, which will rise cache hitrate and reduce response latency.
The cache size of EIL is related to the row count in the < COUNTRY, AREA, ISP > isp location whitelist. Therefore, under IPv6 environment, the cache size of EIL will be much smaller than ECS.

Note that, the EIL's IP2Geo mapping work will make recursive resolver to more computational cost.

## GeoIP-enabled Authoritative Server

Client subnet is the best factor if the company has good network topology monitor ability, offen is for big company.
However, for many authoritative servers that only deployed GeoDNS, the accuracy limitation is commonly because of the IP2Geo database quality, and the small ISPs change to another next-hop big ISP suddenly.

For the GeoIP-enabled authoritative server, the response accurancy depends on the IP geolocation database quality. If authoritative server can not find approximate isp location of ECS's client subnet, they can not return best tailored response.

Even though GeoIP-enabled authoritative servers know about the precise isp location of ECS's client subnet, they may not know about the latest toplogical path change of the isp to update the tailored response in time.
In the case of "small ISP -> big ISP (change frequency) -> ...  -> website", both small ISP's client ip/resolver ip is not good factor for GeoDNS. 
Big companies work hard to catch up with the client ip's connect topology change, and adjust their authoritative servers' tailored response, but smaller companies only deploy IP2Geo may not afford.

EIL wants to give downstream a chance to tell authoritative server its best path quickly and proactively, help to rise the response accuracy, avoid cross-isp visit, save IP transit cost for Data Provider.
EIL directly provide sufficient information for the GeoIP-enabled authoritative server.
Compared to ECS, EIL can reduce GeoIP-enabled authoritative server's dependence on the IP geolocation database quality. 

# Security Considerations

## DNSSEC

EIL is not signed.

## Privacy

The biggest privacy concern on ECS is that client subnet information is personally identifiable. The more domains publish their zones on a third-party authoritative server, the more end user privacy information can be gathered by the authoritative server according to the ECS queries.

EIL is to improve user privacy which is inspired by ECS, prevented leaks in the client subnet information.

Like ECS, EIL will leak the global zonefile configurations of the authoritative servers more easily than normal case.

## Target Censorship 

DNS traffic is plain text by default. It is easily to be blocked or poisoned by internet target censorship. To bypass the censorship, it is better to encrypt the DNS traffic or use some proxy tunnel.

EIL's isp location information covers bigger area than ECS's client subnet information. Therefore, compared to ECS in plain text condition, EIL is weaker at blocking record attack, but stronger at targeted DNS poisoning attack.

## DDoS

To migrate the DDoS problem:

   *  If an Authority Server receives a DNS query with unknown data in EIL option, it SHOULD return the default response whose EIL option with null value.

   *  Nameservers OPTIONAL only implement EIL when the query is from a TCP connection.

More migration techniques described in {{RFC7871}}, Section 11.3.

# IANA Considerations

This document defines EIL, need request IANA to assign a new EDNS0 option code to EIL.

# Acknowledgements

EIL is inspired by ECS, the authors especially thanks to C. Contavalli, W. van der Gaast, D. Lawrence, and W. Kumari.

Thanks comments for Barry Raveendran Greene, Paul Vixie, Petr Špaček, Brian Hartvigsen, Ask Bjørn Hansen, Dave Lawrence.

Thanks a lot to all in the DNSOP, DNSPRIV mailing list.

# Appendix A. GeoIP-enabled Authoritative Servers Example

## BIND

As described in {{BIND-GeoIP}}, BIND 9.10 is able to use data from MaxMind GeoIP databases to achieve restrictions based on the (presumed) geographic location of that address. The ACL itself is still address-based, but the GeoIP-based specification mechanisms can easily populate an ACL with addresses in a certain geographic location. 

    acl "example" {
      geoip country US;
      geoip region CA;
      geoip city "Redwood City"; /* names, etc., must be quoted if they contain spaces */
    };

## PowerDNS

As described in {{PowerDNS-GeoIP}}, PowerDNS supports many geolocation placeholders, such as %co = 3-letter country, %cn = continent, %re = region, %ci = city.

    domains:
    - domain: geo.example.com
      ttl: 30
      records:
        geo.example.com:
          - soa: ns1.example.com hostmaster.example.com 2014090125 7200 3600 1209600 3600
          - ns:
               content: ns1.example.com
               ttl: 600
          - ns: ns2.example.com
          - mx: 10 mx.example.com
        fin.eu.service.geo.example.com:
          - a: 192.0.2.2
          - txt: hello world
          - aaaa: 2001:DB8::12:34DE:3
        # this will result first record being handed out 30% of time
        swe.eu.service.geo.example.com:
          - a:
               content: 192.0.2.3
               weight: 50
          - a: 192.0.2.4
      services:
        # syntax 1
        service.geo.example.com: '%co.%cn.service.geo.example.com'
        # syntax 2
        service.geo.example.com: [ '%co.%cn.service.geo.example.com', 
                                            '%cn.service.geo.example.com']
        # alternative syntax
      services:
        service.geo.example.com:
          default: [ '%co.%cn.service.geo.example.com', '%cn.service.geo.example.com' ]
          10.0.0.0/8: 'internal.service.geo.example.com'

## Amazon

As described in {{Amazon-GeoIP}}, Amazon Route 53 lets you choose the resources that serve your traffic based on the geographic location of your users, meaning the location that DNS queries originate from. It allows you to route some queries for a continent to one resource and to route queries for selected countries on that continent to a different resource.

When a browser or other viewer uses a DNS resolver that does support edns-client-subnet, the DNS resolver sends Amazon Route 53 a truncated version of the client's IP address. Amazon Route 53 determines the location of the user based on the truncated IP address rather than the source IP address of the DNS resolver; this typically provides a more accurate estimate of the client's location. Amazon Route 53 then responds to geolocation queries with the DNS record for the client's location.

## DYN

As described in {{DYN-GeoIP}}, Dyn provides the ability to control DNS responses on a granular/customized geographical rule set. Part of the rulesets will be the identification of the global regions, countries, or states and provinces that use a specific DNS server group. DYN uses the ECS information for the geolocation lookup. Once a geolocation is found and a response is selected, it will provide a DNS response back to the source IP address.

## gdnsd

As described in {{gdnsd-GeoIP}}, gdnsd uses MaxMind's GeoIP binary databases to map address and CNAME results based on geography and monitored service availability. gdnsd supports geolocation codes, such as continent, country, region/subdivision, city.

## Windows Server

As described in {{WindowsServer-GeoIP}}, Windows server can be configured DNS Policy to respond to DNS client queries based on the geographical location of both the client and the resource to which the client is attempting to connect, providing the client with the IP address of the closest resource.

# Appendix B. EIL Example

authoritative server of www.example.com has enabled EIL.

Stub DNS query A resource record of www.example.com .

## P-Model

    Stub DNS 
    -> local forwarding resolver (61.48.7.2) 
    -> Public Forwarding Resolver(AliDNS, 223.5.5.5) 
    -> Public recursive resolver(AliDNS, 202.108.250.231) 
    -> authoritative server

Public Forwarding Resolver 223.5.5.5 could enable EIL and generate the EIL OPT data < CN, 11, UNI > based on 61.48.7.2.

P-Model will not leak client subnet to authoritative server.

## I-Model

    Stub DNS 
    -> local forwarding resolver 
    -> ISP Forwarding Resolver(202.106.196.115) 
    -> ISP recursive resolver(61.135.23.92) 
    -> authoritative server

ISP recursive resolver 61.135.23.92 could enable EIL and generate the EIL OPT data < CN, 11, UNI > based on 61.135.23.92.

If authoritative server doesn't know much about 61.135.23.92, EIL will be helpful.

ISP recursive resolver generates static EIL query, simply manages response cache as tranditionl non-ECS/non-EIL scenario.

EIL helps ISP recursive resolver to give upstream an explicit correct isp location information.

## L-Model

    Stub DNS 
    -> Local Fowarding Resolver(58.60.109.234) 
    -> ... 
    -> authoritative server

Local Fowarding Resolver 58.60.109.234 could enable EIL and generate the option data is < CN, 44, TEL > based on 58.60.109.234.

L-Model can give the most precisely isp location information for DNS resolution.

# Appendix C. Frequent GeoIP-enabled Authoritative Server's Response Accuracy Problem

## Public Recursive Resolver with non-ECS Authoritative Server

If authoritative server doesn't support ECS, the clients that use public recursive resolver(such as 8.8.8.8) may receive disaster latency IP.

In this scenario, we must pray that public recursive resolver's IP is network topological close to client's IP.

## IP2Geo Database Quality

If authoritative server's IP2Geo database misidentify client IP's information, then the client may be assigned to some high letency cross-isp IP address.

With EIL, public recursive resolver and ISP recursive resolver can help to give more precise information for GeoIP-enabled authoritative servers.

## Unstable ISP Network Topology

Some small ISPs may change their upstreams frequently. authoritative servers offen can not catch up the variation in time.

EIL gives downstream a chance to proactively tell authoritative servers the latest best topological close response itself wants now. Downstream can assure itself has got explicit tailored response with EIL.

For example, 218.247.200.100's isp location information is < China, Beijing, PengBoShi >. In I-Model, PengBoShi's resolver can send EIL < CN, 11, TEL > to authoritative servers, indicates that the best topological close response forclient 218.247.200.100 is from China Beijing Telecom.


--- back


--- fluff

<!--  LocalWords:  ECS Privacy EIL GeoIP Geolocation
-->
