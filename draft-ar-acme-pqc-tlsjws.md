---
title: "Quantum-Ready Profiles for ACME"
abbrev: "Quantum-Ready ACME Profiles"
category: std

docname: draft-ar-acme-pqc-tlsjws-00
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
    RFC9147:
    RFC5280:
    RFC9794:
    RFC4343:
    RFC8555:
    RFC2119:
    RFC8174:
    RFC4648:
    RFC4086:
    RFC6066:
    RFC7515:
    I-D.ietf-uta-pqc-app:
    I-D.reddy-tls-composite-mldsa:
    I-D.ietf-jose-pq-composite-sigs:
    FIPS180-4:
        title: "FIPS180-4: Secure Hash Standard (SHS)"
        target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
informative:
  SP-800-56C:
     title: "Recommendation for Key-Derivation Methods in Key-Establishment Schemes"
     target: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-56Cr2.pdf
     date: false   


--- abstract

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) to authenticate client request payloads. Current ACME deployments rely on traditional public-key algorithms in both JWS and TLS, which are vulnerable to attacks from Cryptographically Relevant Quantum Computers (CRQCs). These profiles define PQC or PQ/T TLS 1.3 key establishment and specify PQC or PQ/T authentication mechanisms for JWS and TLS where required by the selected quantum-ready profile. Together, these profiles enable ACME clients and servers to achieve quantum-resistant confidentiality and authentication.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) for authenticating client requests to the Certificate Authority (CA). ACME relies on TLS to provide confidentiality, integrity, and authentication of the ACME server, ensuring secure transport of protocol messages.

ACME defines multiple domain control validation methods, such as HTTP-01, DNS-01, and TLS-based challenges like tls-alpn-01 {{RFC8737}}. In all cases, ACME depends on traditional public-key cryptography, both in JWS for client request authentication and in TLS for server authentication and session key establishment.

The emergence of Cryptographically Relevant Quantum Computers (CRQCs) threatens the security of these mechanisms. Quantum algorithms, such as Shor’s algorithm, would render widely deployed public-key algorithms (e.g., RSA, ECDSA, and ECDH) insecure. Consequently, there is a need to transition ACME to post-quantum cryptography (PQC) to ensure long-term security.

This document defines quantum-ready ACME profiles based on post-quantum and post-quantum/traditional (PQ/T) hybrid cryptography. These profiles specify PQC-enabled TLS 1.3 key establishment and PQC or PQ/T authentication
mechanisms for JWS and TLS. Unless explicitly stated otherwise, all requirements in this document apply only to deployments operating under these quantum-ready profiles.

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

# Data Confidentiality in ACME

In ACME, clients may transmit personally identifiable information (PII) such as email addresses and other account-related metadata when creating or managing an account. The confidentiality of this information relies on the security of the TLS channel, which protects ACME exchanges from passive observation during transmission.

One of the primary threats to TLS in ACME is the Harvest-Now, Decrypt-Later (HNDL) attack. In this scenario, an adversary passively records encrypted TLS traffic, such as ACME account creation or order requests and stores it for future decryption once Cryptographically Relevant Quantum Computers (CRQCs) become available.

CRQCs pose a long-term confidentiality risk because they can break the traditional public-key algorithms (e.g., RSA, ECDH) used in TLS handshakes to establish session keys. Once these algorithms are broken, previously captured TLS sessions can be decrypted, exposing sensitive or personally identifiable information (PII) contained within ACME messages.

To preserve the intended privacy guarantees of TLS 1.3 and protect against HNDL, ACME deployments operating under a quantum-ready profile MUST adopt post-quantum or post-quantum/traditional hybrid key exchange mechanisms, as outlined in Section 4 of {{I-D.ietf-uta-pqc-app}}.  These mechanisms ensure that even if TLS session is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

ACME servers that support 0-RTT MAY allow clients to send early data only if all of the following conditions are met:

* The session resumption ticket and keying material were derived from a  post-quantum or post-quantum/traditional hybrid key exchange; and
* The ACME request includes nonce-based replay protection as specified in Section 6.5 of {{RFC8555}}.

If these conditions are not met, ACME clients MUST NOT use 0-RTT.

# Quantum-Ready Authentication

Post-quantum protection requirements differ significantly between key establishment and authentication mechanisms.

Post-quantum or post-quantum/traditional (PQ/T) hybrid key exchange is urgent due to the risk of Harvest-Now, Decrypt-Later (HNDL) attacks. Compromise of traditional key exchange algorithms would allow an adversary to retrospectively decrypt previously recorded TLS sessions, exposing sensitive ACME traffic.

In contrast, digital signature based authentication (TLS certificates and JWS signatures) is subject to a different threat model. Signature mechanisms cannot be retroactively compromised; attacks require an active, on-path adversary with real-time cryptanalytic capabilities. As a result, PQC-only or PQ/T hybrid authentication is important for long-term security and preparedness, but does not present the same immediate risk as key exchange.

Accordingly, this document mandates the use of post-quantum or PQ/T hybrid key exchange for quantum-ready ACME profiles, while treating post-quantum or PQ/T hybrid authentication mechanisms as a recommended, incrementally deployable enhancement.

### TLS Server Authentication

Although CRQCs could enable real-time forgery of traditional certificate
signatures, exploiting such capabilities would require an active on-path
attack. Migration to post-quantum X.509 certificates is operationally
complex due to the time required to establish trust anchors, update
validation infrastructure, and deploy new root certificates across clients.

For this reason, ACME servers operating under a quantum-ready profile:

- MUST negotiate post-quantum or PQ/T hybrid key exchange mechanisms for
  TLS connections.
- SHOULD support TLS server authentication using PQC-only or PQ/T hybrid
  certificate signatures.
- MAY continue to accept traditional-only certificate signatures during
  an incremental transition period.

### JWS Request Authentication

ACME request authentication relies on the integrity of JSON Web Signatures
(JWS). Similar to TLS certificates, JWS signatures cannot be retroactively
compromised and require an active attacker to exploit traditional
algorithms.

Therefore:

- ACME clients operating under a quantum-ready profile SHOULD use
  PQC-only or PQ/T hybrid signature algorithms for JWS request
  authentication.
- ACME servers MAY accept traditional-only JWS signatures during a
  defined transition period.

### Strict Quantum-Ready Operation

Deployments that explicitly require quantum-resistant authentication
(e.g., high-assurance environments or regulatory-driven profiles) MAY
impose stricter policies. In such configurations:

- TLS authentication and JWS request authentication MUST use PQC-only
  or PQ/T hybrid signature algorithms.
- Any fallback to traditional-only signature algorithms MUST be
  rejected.

## Post-Quantum X.509 Certificates

Post-quantum certificates contain only a PQC public key and are signed using a post-quantum algorithm. They are suitable for deployments capable of fully embracing post-quantum cryptography.

  * ML-DSA Certificates: Defined in {{?I-D.ietf-lamps-dilithium-certificates}}, these use the Module-Lattice Digital Signature Algorithm (ML-DSA). Support for composite ML-DSA certificates in TLS is specified in {{I-D.reddy-tls-composite-mldsa}}. {{!I-D.ietf-tls-mldsa}} explains how ML-DSA is applied for authentication in TLS 1.3.
  * SLH-DSA Certificates: Defined in {{?I-D.ietf-lamps-x509-slhdsa}}, these use the SLH-DSA algorithm. {{?I-D.reddy-tls-slhdsa}} details how SLH-DSA is used in TLS 1.3 and compares its advantages and disadvantages with ML-DSA in Section 2 of the document.

## Post-Quantum JSON Web Signatures (JWS)

ACME clients communicate with ACME servers over HTTPS; ACME requires that client requests be integrity-protected and authenticated using JSON Web Signature (JWS) as specified in {{RFC7515}}. 

  * ML-DSA JWS: Defined in Section 5 of {{?I-D.ietf-cose-dilithium-09}}. This draft specifies JOSE algorithm identifiers and processing rules for ML-DSA signature variants, enabling the use of ML-DSA as a post-quantum signature algorithm in JWS objects for ACME client authentication.

  * SLH-DSA JWS: Defined in Section 3 of {{?I-D.ietf-cose-sphincs-plus-05}}. This draft specifies JOSE algorithm identifiers and processing rules for SLH-DSA signature variants, enabling the use of SLH-DSA as a post-quantum signature algorithm in JWS objects for ACME client authentication.

  * Composite JWS: Defined in {{I-D.ietf-jose-pq-composite-sigs}}, this draft specifies JOSE algorithm identifiers and processing rules for PQ/T hybrid signature constructions in JWS objects.

ACME implementations that intend to provide quantum-resistant client authentication will have to support one or more of the post-quantum JWS algorithm identifiers defined in these specifications. 

# Post-Quantum Validation Challenges

## DNSSEC with PQC Signatures

When the DNS-01 challenge type is used, ACME servers rely on DNSSEC to authenticate the DNS records associated with the validation. However, the DNSSEC ecosystem does not yet have standardized use of post-quantum signature algorithms, and the IETF is still evaluating the feasibility and operational impact of introducing PQC into DNSSEC. As a result, DNSSEC today remains dependent on traditional algorithms such as RSA and ECDSA, which are vulnerable to future CRQCs.

Given these limitations, ACME servers will have to assess the strength of the DNSSEC validation chain and treat DNSSEC dependencies on traditional-only algorithms as providing reduced or uncertain long-term authentication guarantees in the presence of CRQCs. Once DNSSEC supports PQC, ACME servers and CAs are expected to migrate DNS-01 validation policies accordingly.

## TLS-ALPN-01 Challenge

When the tls-alpn-01 challenge type {{RFC8737}} is used, the ACME server validates domain control by initiating a TLS connection to the host identified in the certificate request. ACME servers operating under a strict quantum-ready authentication policy MUST reject tls-alpn-01 validation attempts where the challenge certificate relies solely on traditional algorithms.

## URLs with PQC

When operating under a quantum-ready ACME profile, all URLs provided by the ACME directory (e.g., `newAccount`, `newOrder`, `revokeCert`) MUST be accessible over HTTPS endpoints that use post-quantum or post-quantum/traditional (PQ/T) hybrid key exchange mechanisms.

Authentication requirements for these endpoints follow the authentication policy associated with the selected quantum-ready profile. In particular, support for PQC-only or PQ/T hybrid certificate authentication is RECOMMENDED, while continued use of traditional-only certificate signatures MAY be permitted during an explicitly defined transition period, unless a strict quantum-ready authentication policy is in effect.

## CSR Signature Requirement

When operating under a quantum-ready ACME profile, and when an ACME client explicitly requests issuance of a PQC-only or PQ/T hybrid certificate, the Certificate Signing Request (CSR) submitted during the finalize step MUST be signed using a post-quantum or PQ/T hybrid signature algorithm consistent with the requested certificate type.

For PQC-only certificates:

The CSR MUST be signed using the corresponding post-quantum private key.

For PQ/T hybrid certificates:

The CSR MUST be signed using the corresponding PQ/T hybrid signature scheme.

If a client submits a CSR that contains a PQC or hybrid public key but is signed using a traditional-only signature algorithm (e.g., RSA or ECDSA), the ACME server MUST reject the request with a badCSR error.

This requirement does not apply to CSRs requesting traditional-only certificates and does not impose additional requirements outside the quantum-ready profile.

# Quantum Safe ACME JWS signature

Following is an example for ACME JWS signature:

POST /acme/authz/1234/2
Host: example.com
Content-Type: application/jose+json

{
  "protected": base64url({
    "alg": "ML-DSA-44",
    "kid": "https://example.com/acme/acct/1",
    "nonce": "JHb54aT_KTXBWQOzGYkt9A",
    "url": "https://example.com/acme/authz/1234/1"
  }),
  "payload": base64url({}),
  "signature": "Q1bURgJoEslbD1c5...3pYdSMLio57mQNN4"
}
 
# Security Considerations

The security considerations outlined in {{?I-D.ietf-pquip-pqc-engineers}} must be carefully evaluated and taken into account.

Post-quantum algorithms selected for standardization are relatively new, and their implementations are still in the early stages of maturity. This makes them more susceptible to implementation bugs compared to the well-established and extensively tested cryptographic algorithms currently in use. Furthermore, certain deployments may need to continue using traditional algorithms to meet regulatory requirements, such as Federal Information Processing Standard (FIPS) {{SP-800-56C}} or Payment Card Industry (PCI) compliance.

Hybrid key exchange provides a practical and flexible solution, offering protection against "Harvest Now, Decrypt Later" attacks while ensuring resilience to potential catastrophic vulnerabilities in any single algorithm. This approach allows for a gradual transition to PQC, preserving the benefits of traditional cryptosystems without requiring their immediate replacement.

## MITM Attacks with CRQC in ACME

A MITM attack is possible if an adversary possesses a CRQC capable of breaking traditional public-key signatures. The attacker can generate a forged certificate and create a valid signature, enabling them to impersonate a TLS peer, whether a server or a client. This completely undermines the authentication guarantees of TLS when relying on traditional certificates.

To mitigate such attacks, several steps need to be taken:

1. Revocation and Transition: Both ACME clients and servers that use traditional certificates will have to revoke them and migrate to PQC authentication.
2. Client-Side Verification: ACME clients should avoid establishing TLS sessions with ACME servers that do not support PQC authentication.
3. Server-Side Verification: ACME servers can examine incoming requests to determine whether the client’s JWS signatures and associated JWKs are quantum-resistant. Servers that intend to operate in a quantum-ready mode will have to decline to process requests authenticated with traditional-only signatures.
4. PKI migration planning. Organizations operating PKI infrastructures can plan a transition toward issuing PQC or hybrid certificates and phase out issuance of certificates based solely on traditional algorithms as part of their quantum-readiness strategy.

# IANA Considerations

--- back

# Acknowledgements

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{I-D.ietf-uta-pqc-app}}.
