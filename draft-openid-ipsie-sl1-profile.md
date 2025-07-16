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
workgroup: IPSIE Working Group
keyword:
  - openid
  - ipsie
venue:
  group: IPSIE
  type: Working Group
  mail: openid-specs-ipsie@lists.openid.net
  arch: https://openid.net/wg/ipsie/
  github: "openid/ipsie-openid-sl1"
  latest: "https://openid.github.io/ipsie-openid-sl1/draft-openid-ipsie-sl1-profile.html"

author:
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com

normative:
  BCP195:
  RFC2119:
  RFC8174:
  RFC6749:
  RFC6750:
  RFC6797:
  RFC7636:
  RFC8252:
  RFC8414:
  RFC8725:
  RFC9126:
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
  NIST.FAL:
    title: NIST SP 800-63 Digital Identity Guidelines Federation Assurance Level (FAL)
    target: https://pages.nist.gov/800-63-4/sp800-63c/fal/
    date: August 28, 2024

informative:


--- abstract

TBD

--- middle

# Introduction

This specification defines how to implement OpenID Connect to meet IPSIE's SL1 requirements for enterprise integrations. The profile establishes security and interoperability standards for federated authentication, allowing applications to authenticate users and retrieve additional user claims from the OpenID Connect UserInfo endpoint.

This profile focuses specifically on authentication scenarios and does not cover broad API access use cases. As a result, the use of refresh tokens and / or OAuth DPoP (Demonstration of Proof of Possession) are optional.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

## Roles

This document uses the term "Identity Provider" to refer to the "OpenID Provider" in [OpenID] and the "Authorization Server" in [RFC6749].

This document uses the term "Application" to refer to the "Relying Party" in [OpenID] and the "Client" in [RFC6749].


# Profile

## Network Layer Requirements

### Requirements for all endpoints

To protect against network attacks, clients, authorization servers, and resource servers

* MUST only offer TLS protected endpoints and MUST establish connections to other servers using TLS;
* MUST set up TLS connections using TLS version 1.2 or later;
* MUST follow the recommendations for Secure Use of Transport Layer Security in [BCP195];
* SHOULD use DNSSEC to protect against DNS spoofing attacks that can lead to the issuance of rogue domain-validated TLS certificates; and
* MUST perform a TLS server certificate check, as per [RFC9525].

### Requirements for endpoints not used by web browsers

For server-to-server communication endpoints that are not used by web browsers, the following requirements apply:

* When using TLS 1.2, servers MUST only permit the cipher suites recommended in [BCP195];
* When using TLS 1.2, clients SHOULD only permit the cipher suites recommended in [BCP195].

### Requirements for endpoints used by web browsers

For endpoints that are used by web browsers, the following additional requirements apply:

* Servers MUST use methods to ensure that connections cannot be downgraded using TLS stripping attacks. A preloaded [preload] HTTP Strict Transport Security policy [RFC6797] can be used for this purpose. Some top-level domains, like .bank and .insurance, have set such a policy and therefore protect all second-level domains below them.
* When using TLS 1.2, servers MUST only use cipher suites allowed in [BCP195].
* Servers MUST NOT support [CORS] for the authorization endpoint, as clients must perform an HTTP redirect rather than access this endpoint directly.


## Cryptography and Secrets

The following requirements apply to cryptographic operations and secrets:

* Authorization servers, clients, and resource servers when creating or processing JWTs:
  * MUST adhere to [RFC8725];
  * MUST use PS256, ES256, or EdDSA (using the Ed25519 variant) algorithms; and
  * MUST NOT use or accept the `none` algorithm.
* RSA keys MUST have a minimum length of 2048 bits.
* Elliptic curve keys MUST have a minimum length of 224 bits.
* Credentials not intended for handling by end-users (e.g., access tokens, refresh tokens, authorization codes, etc.) MUST be created with at least 128 bits of entropy such that an attacker correctly guessing the value is computationally infeasible ({{Section 10.10 of RFC6749}}).

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

OpenID Providers:

* MUST distribute discovery metadata (such as the authorization endpoint) via the metadata document as specified in [OpenID.Discovery];
* MUST reject requests using the resource owner password credentials grant;
* MUST support public clients as defined in [RFC6749];
* MUST NOT expose open redirectors {{Section 4.11 of RFC9700}};
* MUST only accept its issuer identifier value (as defined in [RFC8414]) as a string in the `aud` claim received in client authentication assertions;
* MUST issue authorization codes with a maximum lifetime of 60 seconds;
* MUST require clients to be preregistered, and MUST NOT support unauthenticated Dynamic Client Registration requests (see Note 1);
* MUST require clients to pre-register their redirect URIs;

Access Tokens issued by OpenID Providers:

* MUST only be used by the RP to retrieve identity claims at the OpenID Provider;
* MUST only issue sender-constrained access tokens using DPoP [RFC9449];

ID Tokens issued by OpenID Providers:

* MUST contain the OAuth Client ID of the RP as a single audience value as a string (see Note 2);
* MUST contain the `acr` claim as a string that identifies the Authentication Context Class that the authentication performed satisfied, as described in Section 2 of [OpenID];
* MUST contain the `amr` claim as an array of strings indicating identifiers for authentication methods used in the authentication from those registered in the IANA Authentication Method Reference Values registry, as described in Section 2 of [OpenID];
* MUST indicate the expected lifetime of the RP session in the `session_lifetime` claim in seconds (see Note 3);
* MUST contain the `auth_time` claim to describe when end user authentication last occurred (see Note 4);
* MUST indicate the expected expiration time of the RP session in the `session_expiry` claim as a JSON integer that represents the Unix timestamp (seconds since epoch). (see Note 3);

Note 1: The requirement for preregistered clients corresponds to Section 3.4 "Trust Agreements" of [NIST.FAL].

Note 2: The audience value must be a single string to meet the audience restriction of [NIST.FAL].

Note 3: This claim is currently being defined in the AB Connect WG.  See the latest draft at https://openid.github.io/connect-enterprise-extensions/main.html.

Note 4: This claim is required to satisfy the requirements in Section 4.7 of [NIST.FAL].


For the authorization code flow, OpenID Providers:

* MUST require the value of `response_type` described in [RFC6749] to be `code`;
* MUST require PKCE [RFC7636] with S256 as the code challenge method (see Note 1 below);
* MUST require an exact match of a registered redirect URI as described in {{Section 2.1 of RFC9700}};
* MUST issue authorization codes with a maximum lifetime of 60 seconds;
* MUST support "Authorization Code Binding to DPoP Key" (as required by {{Section 10.1 of RFC9449}});
* MUST return an `iss` parameter in the authorization response according to [RFC9207];
* MUST NOT transmit authorization responses over unencrypted network connections, and, to this end, MUST NOT allow redirect URIs that use the `http` scheme;
* MUST reject an authorization code (Section 1.3.1 of [RFC6749]) if it has been previously used;
* MUST NOT use the HTTP 307 status code when redirecting a request that contains user credentials to avoid forwarding the credentials to a third party accidentally (see {{Section 4.12 of RFC9700}});
* SHOULD use the HTTP 303 status code when redirecting the user agent using status codes;
* MUST support `nonce` parameter values up to 64 characters in length, and MAY reject `nonce` values longer than 64 characters.
* MUST support the `max_age` parameter with a values representing the maximum number of seconds allowable since the user was authenticated by the OP. If the elapsed time since authentication is less than this value, the OP MAY choose to actively reauthenticate the user.  If the elapsed time since authentication is greater than this value, the OP MUST actively reauthenticate the user.
* MUST support the `prompt=login` parameter by requiring user-interactive reauthentication of the user when this parameter is received from the RP.

Note 1: while both nonce and PKCE can provide protection from authorization code injection, nonce relies on the client (RP) to implement and enforce the check, and the IdP is unable to verify that it has been implemented correctly, and only stops the attack after tokens have already been issued. Instead, PKCE is enforced by the IdP and stops the attack before tokens are issued.



### Requirements for OpenID Relying Parties

OpenID Relying Parties:

* MUST support third-party initiated login as defined in Section 4 of [OpenID];
* MUST use the authorization server's issuer identifier value (as defined in [RFC8414]) in the `aud` claim in client authentication assertions. The issuer identifier value shall be sent as a string not as an item in an array;
* MUST NOT expose open redirectors (see {{Section 4.11 of RFC9700}});
* MUST only use authorization server metadata (such as the authorization endpoint) retrieved from the metadata document as specified in [OpenID.Discovery] and [RFC8414];
* MUST ensure that the issuer URL used as the basis for retrieving the authorization server metadata is obtained from an authoritative source and using a secure channel, such that it cannot be modified by an attacker;
* MUST ensure that this issuer URL and the issuer value in the obtained metadata match;

OpenID Relying Parties making resource requests to the OpenID Provider:

* MUST support sender-constrined access tokens using DPoP as described in [RFC9449];
* MUST support the server-provided nonce mechanism (as defined in {{Section 8 of RFC9449}});
* MUST send access tokens in the HTTP header as described in {{Section 7.1 of RFC9449}};
* MUST support refresh tokens and their rotation;

For the authorization code flow, Relying Parties:

* MUST use the authorization code grant described in [RFC6749];
* MUST use PKCE [RFC7636] with S256 as the code challenge method;
* MUST generate the PKCE challenge specifically for each authorization request and securely bind the challenge to the client and the user agent in which the flow was started;
* MUST check the `iss` parameter in the authorization response according to [RFC9207] to prevent mix-up attacks;
* SHOULD NOT use `nonce` parameter values longer than 64 characters;
* SHOULD use the `max_age` parameter in the authentication request to specify the maximum allowable authentication age to the OP in seconds.  The value of the `max_age` parameter MAY be determined based upon the business rules of the RP.

In addition to the ID Token validation requirements described in Section 3.1.37 of [OpenID], Relying Parties:

* MUST validate that the `aud` claim is a single string and matches the OAuth Client ID of the RP;
* MUST set the session expiration of the session created to match the `session_expiry` claim (see Note 1);

Note 1: This claim is currently being defined in the AB Connect WG.  See the latest draft at https://openid.github.io/connect-enterprise-extensions/main.html.


# Security Considerations



# IANA Considerations

This document has no IANA actions.




--- back



# Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer,
implementer, or other interested party a non-exclusive, royalty free,
worldwide copyright license to reproduce, prepare derivative works from,
distribute, perform and display, this Implementers Draft, Final
Specification, or Final Specification Incorporating Errata Corrections
solely for the purposes of (i) developing specifications,
and (ii) implementing Implementers Drafts, Final Specifications,
and Final Specification Incorporating Errata Corrections based
on such documents, provided that attribution be made to the OIDF as the
source of the material, but that such attribution does not indicate an
endorsement by the OIDF.

The technology described in this specification was made available
from contributions from various sources, including members of the OpenID
Foundation and others. Although the OpenID Foundation has taken steps to
help ensure that the technology is available for distribution, it takes
no position regarding the validity or scope of any intellectual property
or other rights that might be claimed to pertain to the implementation
or use of the technology described in this specification or the extent
to which any license under such rights might or might not be available;
neither does it represent that it has made any independent effort to
identify any such rights. The OpenID Foundation and the contributors to
this specification make no (and hereby expressly disclaim any)
warranties (express, implied, or otherwise), including implied
warranties of merchantability, non-infringement, fitness for a
particular purpose, or title, related to this specification, and the
entire risk as to implementing this specification is assumed by the
implementer. The OpenID Intellectual Property Rights policy
(found at openid.net) requires
contributors to offer a patent promise not to assert certain patent
claims against other contributors and against implementers.
OpenID invites any interested party to bring to its attention any
copyrights, patents, patent applications, or other proprietary rights
that may cover technology that may be required to practice this
specification.


# Document History

[[ To be removed from the final specification ]]

-01

* Replaced keywords with uppercase IETF keywords

-00

Initial draft

# Acknowledgments
{:numbered="false"}

TODO acknowledge.


