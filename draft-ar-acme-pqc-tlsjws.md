---
title: "Quantum-Ready Profiles for ACME"
abbrev: "Quantum-Ready ACME Profiles"
category: std

docname: draft-ar-acme-pqc-tlsjws
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "ACME"
keyword:
 - PQC
 - ACME

updates: RFC8555

venue:
  group: "acme"
  type: "Working Group"
  mail: "acme@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/acme/"


stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Aritra Banerjee
    organization: Nokia
    city: London
    country: United Kingdom
    email: "aritra.banerjee@nokia.com"
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "k.tirumaleswar_reddy@nokia.com"

normative:
    RFC8737:
    RFC8446:
    RFC9794:
    RFC8555:
    RFC2119:
    RFC8174:
    RFC7515:  
    RFC7518:  
    RFC9964:   
    I-D.ietf-uta-pqc-app:
    I-D.ietf-jose-pq-composite-sigs:
    I-D.ietf-cose-sphincs-plus:
informative:
  FIPS-140-3:
     title: "Security Requirements for Cryptographic Modules"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-3.pdf
     date: false
     author:
        - org: National Institute of Standards and Technology (NIST)
  RFC9958:   
  I-D.reddy-pquip-pqc-signature-migration:


--- abstract

This document updates RFC 8555 (ACME) to specify post-quantum and PQ/T hybrid cryptographic requirements for quantum-ready deployments. The Automatic Certificate Management Environment (ACME) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) to authenticate client request payloads. Current ACME deployments rely on traditional public-key algorithms in both JWS and TLS, which are vulnerable to attacks from Cryptographically Relevant Quantum Computers (CRQCs). These profiles define PQC or PQ/T TLS 1.3 key establishment and specify PQC or PQ/T authentication mechanisms for JWS and TLS where required by the selected quantum-ready profile. Together, these profiles enable ACME clients and servers to achieve quantum-resistant confidentiality and authentication.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) {{RFC7515}} for authenticating client requests to the Certificate Authority (CA). ACME relies on TLS to provide confidentiality, integrity, and authentication of the ACME server, ensuring secure transport of protocol messages.

ACME defines multiple domain control validation methods, such as HTTP-01, DNS-01, and TLS-based challenges like tls-alpn-01 {{RFC8737}}. In all cases, ACME depends on traditional public-key cryptography, both in JWS for client request authentication and in TLS for server authentication and session key establishment.

The emergence of Cryptographically Relevant Quantum Computers (CRQCs) threatens the security of these mechanisms. Quantum algorithms, such as Shor's algorithm, would render widely deployed public-key algorithms (e.g., RSA, ECDSA, and ECDH) insecure. Consequently, there is a need to transition ACME to post-quantum cryptography (PQC) to ensure long-term security.

This document makes two distinct contributions. First, it specifies post-quantum and PQ/T hybrid signature algorithms as mandatory-to-implement for authentication in quantum-ready ACME deployments — covering both JWS request authentication (updating RFC 8555 Section 6.2, following the same precedent by which RFC 8555 mandated ES256) and TLS server authentication. Second, it mandates the use of post-quantum or PQ/T hybrid key exchange for the TLS channel underlying ACME, specifically to address the Harvest-Now, Decrypt-Later (HNDL) threat to ACME traffic confidentiality. Unless explicitly stated otherwise, all requirements in this document apply only to deployments operating under these quantum-ready profiles.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document also adopts terminology defined in {{RFC9794}}.

For the purposes of this document, it is useful to categorize cryptographic algorithms into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer factorization, finite field discrete logarithms, or elliptic curve discrete logarithms. In the context of TLS, an example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure against attacks from both quantum and classical computers. An example of a post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism (ML-KEM). Such algorithms rely on mathematical problems (e.g., lattices) that are believed to be hard for both classical and CRQCs to solve efficiently.

* Hybrid Algorithm: The distinction is made between key exchange algorithms and signature algorithms:

  - PQ/T Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms - one traditional algorithm and one post-quantum algorithm. The resulting shared secret remains secure as long as at least one of the component key exchange algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed of two or more component signature algorithms, where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

* For clarity, this document uses the term PQC-only to refer exclusively to cryptographic mechanisms that contain no traditional component, and distinguishes them from PQ/T hybrid mechanisms that combine post-quantum and traditional algorithms.

* Quantum-Ready Profile: A deployment profile under which all TLS connections use post-quantum or PQ/T hybrid key exchange, and post-quantum or PQ/T hybrid authentication is supported for both TLS server authentication and JWS request authentication. Traditional-only authentication may continue to be accepted during an explicitly bounded transition period.

* Strict Quantum-Ready Profile: A Quantum-Ready Profile that additionally rejects traditional-only authentication, requiring post-quantum or PQ/T hybrid signature algorithms for both TLS server authentication and JWS request authentication with no traditional-only fallback.

Selection of a Quantum-Ready Profile or Strict Quantum-Ready Profile is a matter of deployment policy.

# Quantum-Ready TLS Channel Requirements {#quantum-ready-tls-channel-requirements}

In ACME, clients may transmit personally identifiable information (PII) such as email addresses and other account-related metadata when creating or managing an account. The confidentiality of this information relies on the security of the TLS 1.3 {{RFC8446}} channel, which protects ACME exchanges from passive observation during transmission.

ACME deployments operating under a quantum-ready profile MUST use TLS 1.3 {{RFC8446}}, or a later version of TLS, for all ACME message exchanges; earlier versions of TLS MUST NOT be used. Post-quantum and PQ/T hybrid key exchange for TLS is defined only for TLS 1.3, so this requirement is a prerequisite for the key-establishment protections specified below.

One of the primary threats to TLS in ACME is the Harvest-Now, Decrypt-Later (HNDL) attack. In this scenario, an adversary passively records encrypted TLS traffic, such as ACME account creation or order requests, and stores it for future decryption once Cryptographically Relevant Quantum Computers (CRQCs) become available. CRQCs can break the traditional public-key algorithms (e.g., RSA, ECDH) used in TLS handshakes to establish session keys, exposing sensitive PII contained within ACME messages.

To protect against HNDL, ACME deployments operating under a quantum-ready profile MUST adopt post-quantum or PQ/T hybrid key exchange mechanisms for all TLS connections, as outlined in Section 4 of {{I-D.ietf-uta-pqc-app}}. These mechanisms ensure that recorded TLS sessions cannot be retrospectively decrypted.

# Quantum-Ready Authentication

Post-quantum protection requirements differ between key establishment and authentication mechanisms, but both require action before a Cryptographically Relevant Quantum Computer (CRQC) becomes available.

Unlike Harvest-Now, Decrypt-Later attacks that target confidentiality retrospectively, the threat to authentication arises only from the moment a CRQC becomes available. However, as discussed in {{?I-D.reddy-pquip-pqc-signature-migration}}, the availability of a CRQC may not be publicly disclosed — similar to a zero-day vulnerability, an adversary could exploit quantum capabilities privately to forge traditional certificates without alerting the wider ecosystem. Furthermore, because ecosystem-wide upgrades across cryptographic libraries, PKI infrastructure, Hardware Security Modules (HSMs), and dependent protocols require years of planning and deployment, preparations must begin well before a CRQC is publicly known. Post-quantum security is achieved only when relying parties enforce policies that reject traditional-only authentication.

For these reasons, this document mandates support for post-quantum or PQ/T hybrid authentication mechanisms for quantum-ready ACME profiles, while permitting continued acceptance of traditional-only signatures during an explicitly bounded transition period.

## TLS Server Authentication {#tls-server-authentication}

Migration to post-quantum X.509 certificates is operationally complex due to
the time required to establish new trust anchors, update validation
infrastructure, and deploy new root certificates across clients. As noted in
{{?I-D.reddy-pquip-pqc-signature-migration}}, these transitions require years
of planning and must begin well before a CRQC is publicly known.

For TLS server authentication, ACME servers operating under a quantum-ready profile:

- MUST support TLS server authentication using PQC-only or PQ/T hybrid
  certificate signatures, following the authentication guidance of
  {{I-D.ietf-uta-pqc-app}}.
- MAY continue to accept traditional-only certificate signatures during
  an explicitly bounded transition period.

## JWS Request Authentication {#jws-request-authentication}

ACME request authentication relies on the integrity of JSON Web Signatures
(JWS) {{RFC7515}}. While JWS signatures cannot be retroactively forged, an 
adversary with a CRQC could forge them in real time to impersonate ACME clients 
and submit fraudulent certificate requests. As with TLS authentication, the 
risk that a CRQC may be available without public disclosure means that migration 
must be underway before the threat materialises.

This document updates Section 6.2 of {{RFC8555}}.

ACME servers operating under a quantum-ready profile MUST implement ML-DSA-44, ML-DSA-65, and ML-DSA-87 {{RFC9964}} and the composite ML-DSA algorithm pairing ML-DSA-44 with ECDSA-P256 {{!I-D.ietf-jose-pq-composite-sigs}}.
Such servers SHOULD also implement SLH-DSA-SHAKE-128s, SLH-DSA-SHAKE-192s, and
SLH-DSA-SHAKE-256s {{!I-D.ietf-cose-sphincs-plus}} and the composite ML-DSA
algorithms pairing ML-DSA-65 with Ed25519 and ML-DSA-87 with Ed448
{{!I-D.ietf-jose-pq-composite-sigs}}. ES256 remains mandatory-to-implement
for backward compatibility during the transition period. ACME servers MAY
accept traditional-only JWS signatures during an explicitly bounded
transition period, except when operating under the Strict Quantum-Ready
Profile.

The JWS "alg" values named in this section are registered in the IANA "JSON
Web Signature and Encryption Algorithms" registry established by {{RFC7518}}.

This profile uses SLH‑DSA with SHAKE rather than SHA‑2. ML‑DSA and SLH‑DSA‑SHAKE both rely only on SHAKE, so this choice lets every post‑quantum signature in the profile use a single hash primitive, avoiding a separate SHA‑2 implementation needed only by SLH‑DSA‑SHA2 and reducing the number of distinct primitives. Composite ML‑DSA complements this by combining a post‑quantum signature with a traditional one (e.g., ECDSA or RSA), so a certificate remains secure as long as either component holds. This provides hybrid assurance during migration and lowers deployment risk for ACME clients and existing PKI that are not yet ready to rely on a post‑quantum algorithm alone.

The security properties of these composite algorithms, including their
strong-unforgeability (SUF-CMA) characteristics, are described in the
Security Considerations of {{I-D.ietf-jose-pq-composite-sigs}}.

## Strict Quantum-Ready Profile {#strict-quantum-ready-profile}

The Strict Quantum-Ready Profile is intended for deployments that require
quantum-resistant authentication (e.g., high-assurance environments or
regulatory-driven profiles). Under this profile:

- TLS server authentication and JWS request authentication MUST use PQC-only
  or PQ/T hybrid signature algorithms.
- Any fallback to traditional-only signature algorithms MUST be rejected.

# Additional Quantum-Ready Requirements for ACME

## DNSSEC with PQC Signatures

When the DNS-01 challenge type is used, ACME servers rely on DNSSEC to authenticate the DNS records associated with the validation. However, the DNSSEC ecosystem does not yet have standardized use of post-quantum signature algorithms, and the IETF is still evaluating the feasibility and operational impact of introducing PQC into DNSSEC. As a result, DNSSEC today remains dependent on traditional algorithms such as RSA and ECDSA, which are vulnerable to future CRQCs.

Given these limitations, ACME servers SHOULD assess the strength of the DNSSEC validation chain and treat DNSSEC dependencies on traditional-only algorithms as providing reduced or uncertain long-term authentication guarantees in the presence of CRQCs. Once DNSSEC supports PQC, ACME servers and CAs are expected to migrate DNS-01 validation policies accordingly.

## URLs with PQC

When operating under a quantum-ready ACME profile, all URLs provided by the
ACME directory (e.g., `newAccount`, `newOrder`, `revokeCert`) MUST be
accessible over HTTPS endpoints that use post-quantum or PQ/T hybrid key
exchange mechanisms, consistent with the TLS channel requirements of
{{quantum-ready-tls-channel-requirements}}.

The certificate authentication presented on these endpoints is the TLS
server authentication described in {{tls-server-authentication}} and is
governed by the authentication policy of the selected quantum-ready
profile. This section imposes no separate or additional authentication
requirement.

# Quantum Safe ACME JWS signature

Following is an example for ACME JWS signature:

~~~
POST /acme/authz/1234/2
Host: example.com
Content-Type: application/jose+json

{
  "protected": base64url({
    "alg": "ML-DSA-44",
    "kid": "https://example.com/acme/acct/1",
    "nonce": "JHb54aT_KTXBWQOzGYkt9A",
    "url": "https://example.com/acme/authz/1234/2"
  }),
  "payload": base64url({}),
  "signature": "Q1bURgJoEslbD1c5...3pYdSMLio57mQNN4"
}
~~~

# Security Considerations

The security considerations outlined in {{?RFC9958}} must be carefully evaluated and taken into account.

Post-quantum algorithms selected for standardization are relatively new, and their implementations are still in the early stages of maturity. This makes them more susceptible to implementation bugs compared to the well-established and extensively tested cryptographic algorithms currently in use. Furthermore, certain deployments may need to continue using traditional algorithms to meet regulatory requirements, such as Federal Information Processing Standard (FIPS) 140-3 {{FIPS-140-3}} or Payment Card Industry (PCI) compliance.
Hybrid key exchange provides a practical and flexible solution, offering protection against "Harvest Now, Decrypt Later" attacks while ensuring resilience to potential catastrophic vulnerabilities in any single algorithm. This approach allows for a gradual transition to PQC, preserving the benefits of traditional cryptosystems without requiring their immediate replacement.

SLH-DSA signatures are substantially larger than ML-DSA or traditional
signatures. ACME servers that accept SLH-DSA-authenticated JWS requests
SHOULD account for the increased message size in request-size limits and
resource provisioning, so that the larger signatures do not become a
denial-of-service vector.

## MITM Attacks with CRQC in ACME

A MITM attack can occur if an adversary possesses a CRQC capable of breaking traditional public-key signature schemes. In such a scenario, the adversary can generate forged certificates and produce valid signatures, thereby impersonating a TLS peer (server or client). Consequently, the authentication guarantees of TLS have to be considered compromised when relying solely on traditional certificate-based authentication once a CRQC is available.

The normative measures that mitigate this threat are specified in the profile
sections of this document and are not restated here as independent
requirements:

* Under the Quantum-Ready Profile, ACME servers MUST support PQC-only or PQ/T
  hybrid authentication for both TLS server authentication
  ({{tls-server-authentication}}) and JWS request authentication
  ({{jws-request-authentication}}). Traditional-only authentication MAY
  continue to be accepted only during an explicitly bounded transition
  period.

* Under the Strict Quantum-Ready Profile, ACME servers MUST reject any
  authentication — TLS server authentication or JWS request authentication —
  that relies solely on traditional signature algorithms, with no fallback
  ({{strict-quantum-ready-profile}}).

Independent of the selected profile, organizations operating PKI
infrastructures are expected to define and implement a transition strategy
toward issuing PQC or PQ/T hybrid certificates, phasing out certificates that
rely exclusively on traditional cryptographic algorithms within a defined
migration timeline as part of their quantum-readiness posture.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgements

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{I-D.ietf-uta-pqc-app}}.
