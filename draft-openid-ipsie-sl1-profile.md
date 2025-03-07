---
title: "IPSIE SL1 OpenID Connect Profile"
abbrev: "IPSIE SL1"
category: info

docname: draft-openid-ipsie-sl1-profile-latest
submissiontype: "independent"
number:
date:
v: 3
# area: AREA
# workgroup: IPSIE Working Group
keyword:
  - openid
  - ipsie
venue:
#  group: IPSIE
#  type: Working Group
#  mail: openid-specs-ipsie@lists.openid.net
#  arch: https://openid.net/wg/ipsie/
  github: "aaronpk/ipsie-openid-sl1"
  latest: "https://aaronpk.github.io/ipsie-openid-sl1/draft-openid-ipsie-sl1-profile.html"

author:
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com

normative:
  BCP195:
  RFC6749:
  RFC6750:
  RFC6797:
  RFC7636:
  RFC8414:
  RFC9207:
  RFC9449:
  RFC9525:
  RFC9700:
  OpenID:
    title: OpenID Connect Core 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore
  OpenID.Discovery:
    title: OpenID Connect Discovery 1.0 incorporating errata set 2
    target: https://openid.net/specs/openid-connect-discovery-1_0.html
    date: December 15, 2023
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: E. Jay

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

The keywords "shall", "shall not", "should", "should not", "may", and "can" in
this document are to be interpreted as described in ISO Directive Part 2
[ISODIR2]. These keywords are not used as dictionary terms such that any
occurrence of them shall be interpreted as keywords and are not to be
interpreted with their natural language meanings.

# Profile

## Network Layer Requirements

### Requirements for all endpoints

To protect against network attacks, clients, authorization servers, and resource servers

* shall only offer TLS protected endpoints and shall establish connections to other servers using TLS;
* shall set up TLS connections using TLS version 1.2 or later;
* shall follow the recommendations for Secure Use of Transport Layer Security in [BCP195];
* should use DNSSEC to protect against DNS spoofing attacks that can lead to the issuance of rogue domain-validated TLS certificates; and
* shall perform a TLS server certificate check, as per [RFC9525].

### Requirements for endpoints not used by web browsers

For server-to-server communication endpoints that are not used by web browsers, the following requirements apply:

* When using TLS 1.2, servers shall only permit the cipher suites recommended in [BCP195];
* When using TLS 1.2, clients should only permit the cipher suites recommended in [BCP195].

### Requirements for endpoints used by web browsers

For endpoints that are used by web browsers, the following additional requirements apply:

* Servers shall use methods to ensure that connections cannot be downgraded using TLS stripping attacks. A preloaded [preload] HTTP Strict Transport Security policy [RFC6797] can be used for this purpose. Some top-level domains, like .bank and .insurance, have set such a policy and therefore protect all second-level domains below them.
* When using TLS 1.2, servers shall only use cipher suites allowed in [BCP195].
* Servers shall not support [CORS] for the authorization endpoint, as clients must perform an HTTP redirect rather than access this endpoint directly.


## Cryptography and Secrets


## OpenID Connect

In the following, a profile of the following technologies is defined:

* OpenID Connect Core 1.0 incorporating errata set 2 [OpenID.Discovery]
* OpenID Connect Discovery [OpenID.Discovery]
* OAuth 2.0 Authorization Framework [RFC6749]
* Proof Key for Code Exchange (PKCE) [RFC7636]
* OAuth 2.0 Authorization Server Metadata [RFC8414]
* OAuth 2.0 Demonstrating Proof of Possession (DPoP) [RFC9449]
* OAuth 2.0 Authorization Server Issuer Identification [RFC9207]


### Requirements for OpenID Providers

#### General Requirements

OpenID Providers:

* shall distribute discovery metadata (such as the authorization endpoint) via the metadata document as specified in [OpenID.Discovery];
* shall reject requests using the resource owner password credentials grant;
* shall only support confidential clients as defined in [RFC6749];
* shall only issue sender-constrained access tokens using DPoP [RFC9449];
* shall authenticate clients using `private_key_jwt` as specified in Section 9 of [OpenID];
* shall not expose open redirectors {{Section 4.11 of RFC9700}};
* shall only accept its issuer identifier value (as defined in [RFC8414]) as a string in the `aud` claim received in client authentication assertions;
* shall


#### Authorization Endpoint Flows

For flows that use the authorization endpoint, OpenID Providers:

* shall require the value of `response_type` described in [RFC6749] to be `code`;
* shall require PKCE [RFC7636] with S256 as the code challenge method (see Note 1 below);
* shall issue authorization codes with a maximum lifetime of 60 seconds;
* shall support "Authorization Code Binding to DPoP Key" (as required by {{Section 10.1 of RFC9449}});
* shall return an iss parameter in the authorization response according to [RFC9207];
* shall not transmit authorization responses over unencrypted network connections, and, to this end, shall not allow redirect URIs that use the "http" scheme except for native clients that use loopback interface Redirection as described in Section 7.3 of [RFC8252];
* shall reject an authorization code (Section 1.3.1 of [RFC6749]) if it has been previously used;
* shall not use the HTTP 307 status code when redirecting a request that contains user credentials to avoid forwarding the credentials to a third party accidentally (see {{Section 4.12 of RFC9700}});
* should use the HTTP 303 status code when redirecting the user agent using status codes;
* shall support `nonce` parameter values up to 64 characters in length, may reject `nonce` values longer than 64 characters.

TBD: Should PAR be required at this level?

* shall support client-authenticated pushed authorization requests according to [RFC9126];
* shall reject authorization requests sent without [RFC9126];
* shall reject pushed authorization requests without client authentication;
* shall issue pushed authorization requests request_uri with expires_in values of less than 600 seconds;


Note 1: while both nonce and PKCE can provide protection from authorization code injection, nonce relies on the client (RP) to implement and enforce the check, and the IdP is unable to verify that it has been implemented correctly, and only stops the attack after tokens have already been issued. Instead, PKCE is enforced by the IdP and stops the attack before tokens are issued.


### Requirements for OpenID Relying Parties




# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
