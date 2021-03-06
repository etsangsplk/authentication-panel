# Solid-OIDC Authentication Spec - Draft

# Abstract

A key challenge on the path toward re-decentralizing user data on the
Worldwide Web is the need to access multiple potentially untrusted
resources servers securely. This document aims to address that challenge
by building on top of current and future web standards, to allow
entities to authenticate within a distributed ecosystem.

# Status of This Document

This section describes the status of this document at the time of its
publication. Other documents may supersede this document. A list of
current W3C publications and the latest revision of this technical
report can be found in the [W3C technical reports
index](https://www.w3.org/TR/) at https://www.w3.org/TR/.

This document is produced from work by the [Solid Community
Group](https://www.w3.org/community/solid/). It is a draft document that
may, or may not, be officially published. It may be updated, replaced,
or obsoleted by other documents at any time. It is inappropriate to cite
this document as anything other than work in progress. The source code
for this document is available at the following
URI: <https://github.com/solid/authentication-panel>

This document was published by the [Solid Authentication
Panel](https://github.com/solid/process/blob/master/panels.md#authentication)
as a First Draft.

[GitHub Issues](https://github.com/solid/authentication-panel/issues)
are preferred for discussion of this specification. Alternatively, you
can send comments to our mailing list. Please send them to...

> TODO: Add email address, and archives links (archives).

# Introduction

_This section is non-normative_

The [Solid project](https://solidproject.org/) aims to change the way
web applications work today to improve privacy and user control of
personal data by utilizing current standards, protocols, and tools, to
facilitate building extensible and modular decentralized applications
based on [Linked Data](https://www.w3.org/standards/semanticweb/data)
principles.

This specification is written for Authorization and Resource Server
owners intending to implement Solid-OIDC. It is also useful to Solid
application developers charged with implementing a Solid-OIDC client.

The [OAuth 2.0](https://tools.ietf.org/html/rfc6749) and [OpenID Connect
Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) web
standards were published in October 2012, and November 2014,
respectively. Since publication, they have increased with a marked pace
and have claimed wide adoption with extensive _'real-world'_ data and
experience. The strengths of the protocols are now clear, however, in a
changing eco-system where privacy and control of digital identities are
becoming more pressing concerns, it is also clear that additional
functionality is required.

The additional functionality is aimed at addressing:

1. Ephemeral clients as a common use case.
2. Resource servers with no existing trust relationship with identity
   providers.

## Out of Scope

> TBD

# Terminology

_This section is non-normative_

This specification uses the terms "access token", "authorization
server", "resource server" (RS), "authorization endpoint", "token
endpoint", "grant type", "access token reques", "access token
response", and "client" defined by The OAuth 2.0 Authorization
Framework \[[RFC6749](https://tools.ietf.org/html/rfc6749)\].

Throughout this specification, we will use the term Identity Provider
(IdP) in line with the terminology used in the [Open ID Connect Core 1.0
specification](https://openid.net/specs/openid-connect-core-1_0.html)
(OIDC). It should be noted that [The OAuth 2.0 Authorization
Framework](https://tools.ietf.org/html/rfc6749) (OAuth) refers to this
same entity as an Authorization Server.

This specification also defines the following terms:

**WebID** _as defined in the [WebID 1.0 Editors
Draft](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/identity-respec.html)_

A WebID is a URI with an HTTP or HTTPS scheme which denotes an Agent
(Person, Organization, Group, Device, etc.)

**Demonstration of Proof-of-Possession at the Application Layer (DPoP)**
_as defined in the [DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04)_

A mechanism for sender-constraining OAuth tokens via a
proof-of-possession mechanism on the application level.

**JSON Web Token (JWT)** _as defined by
[RFC7519](https://tools.ietf.org/html/rfc7519)_

A string representing a set of claims as a JSON object that is encoded
in a JWS or JWE, enabling the claims to be digitally signed or MACed
and/or encrypted.

**JSON Web Key (JWK)** _as defined by
[RFC7517](https://tools.ietf.org/html/rfc7517)_

A JSON object that represents a cryptographic key. The members of the
object represent properties of the key, including its value.

**Proof Key for Code Exchange (PKCE\*)** _as defined by
[RFC7636](https://tools.ietf.org/html/rfc7636)_

An extension to the Authorization Code flow which mitigates the risk of
an authorization code interception attack.

# Conformance

_This section is non-normative_

All authoring guidelines, diagrams, examples, and notes in this document
are non-normative. Everything else in this specification is normative
unless explicitly expressed otherwise.

The key words MAY, MUST, MUST NOT, RECOMMENDED, SHOULD, SHOULD NOT, and
REQUIRED in this document are to be interpreted as described in [BCP
14](https://tools.ietf.org/html/bcp14)
\[[RFC2119](https://www.w3.org/TR/2014/REC-cors-20140116/#refsRFC2119)\]
\[[RFC8174](https://www.w3.org/TR/2019/REC-vc-data-model-20191119/#bib-rfc8174)\]
when, and only when, they appear in all capitals, as shown here.

# Core Concepts

## Ephemeral Identity Providers

_This section is non-normative_

In a decentralized ecosystem, such as Solid, the IdP may be:

- The user
- Any client application, or,
- Preexisting IdPs and vendors

Therefore, this specification makes extensive use of OAuth and OIDC best
practices and assumes the
[Authorization](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps)
Code Flow with PKCE, as per OIDC definition. It is also assumed that
there is no preexisting trust relationship with the IdP. This means
dynamic, and static client registration is entirely optional.

## WebIDs

_This section is non-normative_

In line with Linked Data principles, a
[WebID](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/identity-respec.html)
is a HTTP URI that, when dereferenced, resolves to a profile document
that is structured data in an [RDF
format](https://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/). This
profile document allows people to link with others to grant access to
identity resources as they see fit. WebIDs are an underpinning principle
of the Solid movement and are used as the primary identifier for users
and client applications in this specification.

# Basic Flow

_This section is non-normative_

> TODO: Add diagram.

The basic authentication and authorization flow is as follows:

1. The client requests a non-public resource from the RS.
2. The RS returns a 401 with a `WWW-Authenticate` HTTP header
   containing parameters that inform the client that a DPoP-bound
   Access Token is required.
3. The client presents its WebID to the IdP and requests an
   Authorization Code.
4. The client presents the Authorization Code and a DPoP proof, to the
   token endpoint.
5. The Token Endpoint returns a DPoP-bound Access Token and OIDC ID
   Token, to the client.
6. The client presents the DPoP-bound Access Token and DPoP proof, to
   the RS.
7. The RS validates the Access Token and DPoP header, then returns the
   requested resource.

# Proof of Identity

Client registration (static or dynamic) is not required in Solid-OIDC,
thus Access Tokens need to be bound to the client, to prevent token
replay attacks. This is achieved by using a [Distributed
Proof-of-Possession](https://tools.ietf.org/html/draft-fett-oauth-dpop-04)
(DPoP) mechanism at the application layer.

# Token Instantiation

Assuming the token request and DPoP Proof are valid, the client MUST
receive two tokens from the IdP:

1. A DPoP-bound Access Token
2. An OIDC ID Token

These tokens require additional and/or modified claims for them to be
compliant with the authorization flow laid out in this document.

## Access Token

The client MUST send the IdP a DPoP proof that is valid according to the
[DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04).
The Access Token MUST be a JWT and the IdP MUST embed the client\'s
WebID in the Access Token as a custom claim. This claim MUST be named
`client_webid`.

The audience (`aud`) claim is required in OAuth, however, the
DPoP token provides the full URL of the request, making the `aud`
claim redundant, so in Solid-OIDC the `aud` claim MUST be a string
with the value of `solid`.

An example Access Token:

```js
{
    "sub": "https://janedoe.com/web#id", // Web ID of User
    "iss": "https://idp.example.com",
    "aud": "solid",
    "iat": 1541493724,
    "exp": 1573029723, // Identity credential expiration (separate from the ID token expiration)
    "cnf":{
      "jkt":"0ZcOCORZNYy-DWpqq30jZyJGHTN0d2HglBV3uiguA4I" // DPoP public key confirmation claim
    },
    "client_webid": "https://client.example.com/web#id" // WebID of client
}
```

## ID Token

The subject (`sub`) claim in the returning ID Token MUST be set to the
user's WebID.

Example:

```js
{
    "iss": "https://idp.example.com",
    "sub": "https://janedoe.com/web#id",
    "aud": "https://client.example.com",
    "nonce": "n-0S6_WzA2Mj",
    "exp": 1311281970,
    "iat": 1311280970,
}
```

# Resource Access

Ephemeral clients MUST use DPoP-bound Access Tokens, however, the RS MAY
allow registered clients to access resources using a traditional
`Bearer` tokens.

## DPoP Validation

As mentioned in Section XX, a valid DPoP token is REQUIRED to have an
`iat` claim as stated in the [DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-4.1).
The RS MUST check the `iat` claim's timestamp to ensure it is
received within the agreed-upon limits.

The encoded HTTP response and resource URL of the DPoP token MUST be
dereferenced and matched against the original request in the Access
Token.

If either the DPoP token has expired, or either the URL and the HTTP
method does not match that of the resource request then the RS MUST deny
the resource request.

## Validating the Access Token

### WebID

The `sub` claim of the Access Token MUST be a WebID. This needs to be
dereferenced and checked against the `iss` claim in the Access Token.
If the `iss` claim is different from the domain of the WebID, then the
RS MUST check the WebID document for a `solid:oidcIssuer` property to
check the token issuer is listed. This prevents a malicious identity
provider from issuing valid Access Tokens for arbitrary WebIDs.

### Public/private key pair

The public key in the fingerprint of the Access Token MUST be checked
against the DPoP fingerprint to ensure a match, as outlined in the [DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-6).
To achieve this the RS must fetch the required pubic key from the IdP to
match the Access Tokens `cnf` claim against the DPoP tokens
fingerprint to verify they are a bound pair.

An RS MUST deny any resource request that does not include a verifiable
confirmation claim that is dereferenceable from the Access Token.

### JSON Web Key Store

It is RECOMMENDED that the RS keep a list of trusted IdPs, to facilitate
the expedient lookup of JWKS through local trust stores or cached public
keys.

If the IdP does not appear on the RS's list of trusted providers, then
the RS MUST perform OIDC discovery to fetch the JWKS location by
obtaining the `jwks_uri` from the IdPs
`/.well-known/openid-configuration` page. The JWKS will contain an
undetermined number of public keys. The `kid` claim in the Access
Token header will provide the necessary location of the required public
key from the JWKS keyset.

The JWK MUST be in a standard format as defined in the JSON Web Key
specification\[[RFC7517](https://tools.ietf.org/html/rfc7517)\].

An example of a JWKS:

```js
{
// All values shorten for brevity
"keys": [{
    "p": "0Euvv-n31c26PoY...J5Zntp1uDcQ8",
    "kty": "RSA",
    "q": "sZOchpZUjWhdR...xnUwGJVwjbq2k",
    "d": "H-UIIpXTlULSBUz...3MxPlJTmX1ilW4vEQ",
    "e": "AQAB",
    "use": "sig",
    "kid": "rsa1",
    "qi": "H-Q748vgZD1sQfv...YP0cUYntrzToAFhZrCUftA",
    "dp": "FD1Gdn9ldYDn9-tTjHN...hwjgT2Rrg2055qSlbPEE",
    "alg": "RS256",
    "dq": "j7641y271hgkYncjFF5hJ...hDIk15-aW_1-BFAmPk",
    "n": "kHxvVTz_ygk5bII-7N9pZ...2WR9V8JZGJBvYhUNkJw"
    }]
}
```

# Security Considerations

> This section is incomplete

_This section is non-normative_

As this specification builds upon existing web standards, all security
considerations from OAuth, OIDC, and DPoP specifications may also apply
unless otherwise indicated. The following considerations should be
reviewed by implementors and system/s architects of this specification.

## TLS Requirements

All implementors of this specification MUST support TLS. The version(s)
that ought to be implemented will vary over time, depending on
availability and known security vulnerabilities. Current security
considerations can be found in "Recommendations for Secure Use of
Transport Layer Security (TLS) and Datagram Transport Layer Security
(DTLS)" \[[BCP195](https://tools.ietf.org/html/bcp195)\].

Whenever TLS is used, a TLS server certificate check MUST be performed
\[[RFC6125](https://tools.ietf.org/html/rfc6125)\].

All tokens, client, and user credentials MUST only be transmitted by
TLS.

## Cryptography

All randomly generated cryptographic values SHOULD have sufficient
entropy to make brute-force attacks impractical.

## Client IDs

Implementors SHOULD expire client IDs that are kept in server storage
mitigate the potential for a bad actor to fill server storage with
unexpired or otherwise useless client IDs.

## Client Secrets

Client secrets SHOULD NOT be stored in browser local storage. Doing so
will increase the risk of data leaks should an attacker gain access to
client credentials.

# Privacy considerations

> TODO:

# Acknowledgments

> TODO:

# References

> TODO:
