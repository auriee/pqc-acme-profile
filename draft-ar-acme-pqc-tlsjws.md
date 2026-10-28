---
title: "Quantum-Ready ACME Mechanism"
abbrev: "Quantum-Ready ACME Mechanism"
category: std

docname: draft-ar-acme-pqc-tlsjws-01
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

This document updates RFC 8555 (ACME) to specify post-quantum and PQ/T hybrid cryptographic requirements for quantum-ready deployments. The Automatic Certificate Management Environment (ACME) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) to authenticate client request payloads. Current ACME deployments rely on traditional public-key algorithms in both JWS and TLS, which are vulnerable to attacks from Cryptographically Relevant Quantum Computers (CRQCs). This specification defines PQC or PQ/T hybrid TLS 1.3 key establishment and specifies PQC or PQ/T authentication mechanisms for JWS and TLS under the quantum-ready ACME mechanism. This specification enables ACME clients and servers to achieve quantum-resistant confidentiality and authentication.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) {{RFC7515}} for authenticating client requests to the Certificate Authority (CA). ACME relies on TLS to provide confidentiality, integrity, and authentication of the ACME server, ensuring secure transport of protocol messages.

ACME defines multiple domain control validation methods, such as HTTP-01, DNS-01, and TLS-based challenges like tls-alpn-01 {{RFC8737}}. In all cases, ACME depends on traditional public-key cryptography, both in JWS for client request authentication and in TLS for server authentication and session key establishment.

The emergence of Cryptographically Relevant Quantum Computers (CRQCs) threatens the security of these mechanisms. Quantum algorithms, such as Shor's algorithm, would render widely deployed public-key algorithms (e.g., RSA, ECDSA, and ECDH) insecure. Consequently, there is a need to transition ACME to post-quantum cryptography (PQC) to ensure long-term security.

This document makes two distinct contributions. First, it specifies post-quantum and PQ/T hybrid signature algorithms as mandatory-to-implement for authentication in quantum-ready ACME deployments — covering both JWS request authentication (updating RFC 8555 Section 6.2, following the same precedent by which RFC 8555 mandated ES256) and TLS server authentication. Second, it enables the use of post-quantum or PQ/T hybrid key exchange for the TLS channel underlying ACME, specifically to address the Harvest-Now, Decrypt-Later (HNDL) threat to ACME traffic confidentiality. Unless explicitly stated otherwise, all requirements in this document apply only to deployments operating under these quantum-ready mechanisms.

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

* Quantum-Ready Mechanism: A deployment mechanism under which all TLS connections use post-quantum or PQ/T hybrid key exchange, and post-quantum or PQ/T hybrid authentication is supported for both TLS server authentication and JWS request authentication. Traditional-only authentication may continue to be accepted during an explicitly bounded transition period.

Selection of a Quantum-Ready Mechanism is a matter of deployment policy.

# Quantum-Ready TLS Channel Requirements {#quantum-ready-tls-channel-requirements}

In ACME, clients may transmit personally identifiable information (PII) such as email addresses and other account-related metadata when creating or managing an account. The confidentiality of this information relies on the security of the TLS 1.3 {{RFC8446}} channel, which protects ACME exchanges from passive observation during transmission.

ACME deployments operating under a quantum-ready mechanism MUST use TLS 1.3 {{RFC8446}}, or a later version of TLS, for all ACME message exchanges; earlier versions of TLS MUST NOT be used. Post-quantum and PQ/T hybrid key exchange for TLS is defined only for TLS 1.3, so this requirement is a prerequisite for the key-establishment protections described below.

One of the primary threats to TLS in ACME is the Harvest-Now, Decrypt-Later (HNDL) attack. In this scenario, an adversary passively records encrypted TLS traffic, such as ACME account creation or order requests, and stores it for future decryption once Cryptographically Relevant Quantum Computers (CRQCs) become available. CRQCs can break the traditional public-key algorithms (e.g., RSA, ECDH) used in TLS handshakes to establish session keys, exposing sensitive PII contained within ACME messages.

Protection against HNDL requires post-quantum or PQ/T hybrid key exchange for the TLS connection. The key exchange is established at the TLS layer, not by ACME; its post-quantum and PQ/T hybrid form is specified in Section 4 of {{I-D.ietf-uta-pqc-app}}. ACME deployments seeking quantum-ready confidentiality follow that guidance, which ensures that recorded TLS sessions cannot be retrospectively decrypted.

# Quantum-Ready Authentication

Post-quantum protection requirements differ between key establishment and authentication mechanisms, but both require action before a Cryptographically Relevant Quantum Computer (CRQC) becomes available.

Unlike Harvest-Now, Decrypt-Later attacks that target confidentiality retrospectively, the threat to authentication arises only from the moment a CRQC becomes available. However, as discussed in {{?I-D.reddy-pquip-pqc-signature-migration}}, the availability of a CRQC may not be publicly disclosed — similar to a zero-day vulnerability, an adversary could exploit quantum capabilities privately to forge traditional certificates without alerting the wider ecosystem. Furthermore, because ecosystem-wide upgrades across cryptographic libraries, PKI infrastructure, Hardware Security Modules (HSMs), and dependent protocols require years of planning and deployment, preparations must begin well before a CRQC is publicly known. Post-quantum security is achieved only when relying parties enforce policies that reject traditional-only authentication.

For these reasons, this document provides support for post-quantum or PQ/T hybrid authentication mechanisms for quantum-ready ACME mechanism, while permitting continued acceptance of traditional-only signatures during an explicitly bounded transition period.

## TLS Server Authentication {#tls-server-authentication}

Migration to post-quantum X.509 certificates is operationally complex due to the time required to establish new trust anchors, update validation infrastructure, and deploy new root certificates across clients. As noted in {{?I-D.reddy-pquip-pqc-signature-migration}}, these transitions require years of planning and must begin well before a CRQC is publicly known.

For TLS server authentication, the certificate signature algorithms are governed by TLS and the issuing PKI, not by ACME. Post-quantum and PQ/T hybrid certificate authentication for TLS is addressed in {{I-D.ietf-uta-pqc-app}}. ACME deployments seeking quantum-ready server authentication follow that guidance.

## JWS Request Authentication {#jws-request-authentication}

ACME request authentication relies on the integrity of JSON Web Signatures (JWS) {{RFC7515}} produced with the client's account key. While JWS signatures cannot be retroactively forged, an adversary with a CRQC could forge them in real time to impersonate ACME clients and submit fraudulent certificate requests. As with TLS authentication, the risk that a CRQC may be available without public disclosure means that migration must be underway before the threat materialises.

This document updates Section 6.2 of {{RFC8555}}.

ACME servers MAY support any JWS signature algorithm registered in the IANA "JSON Web Signature and Encryption Algorithms" registry {{RFC7518}}. These include the post-quantum and PQ/T hybrid algorithms defined in {{RFC9964}} (ML-DSA), {{!I-D.ietf-cose-sphincs-plus}} (SLH-DSA), and {{!I-D.ietf-jose-pq-composite-sigs}} (composite ML-DSA). This document does not mandate any particular algorithm or parameter set; selection is a matter of deployment and PKI policy.

RFC 8555 Section 6.2 requires an ACME server to implement the ES256 signature algorithm. This document updates that requirement: an ACME server MAY choose not to implement ES256, and MAY implement any signature algorithm registered in the JOSE registry instead. A server that still implements ES256 for backward compatibility with existing clients MAY accept traditional-only JWS signatures during a transition period of its choosing.

# Security Considerations

The security considerations outlined in {{?RFC9958}} must be carefully evaluated and taken into account.

Post-quantum algorithms selected for standardization are relatively new, and their implementations are still in the early stages of maturity. This makes them more susceptible to implementation bugs compared to the well-established and extensively tested cryptographic algorithms currently in use. Furthermore, certain deployments may need to continue using traditional algorithms to meet regulatory requirements, such as Federal Information Processing Standard (FIPS) 140-3 {{FIPS-140-3}} or Payment Card Industry (PCI) compliance. Hybrid key exchange provides a practical and flexible solution, offering protection against "Harvest Now, Decrypt Later" attacks while ensuring resilience to potential catastrophic vulnerabilities in any single algorithm. This approach allows for a gradual transition to PQC, preserving the benefits of traditional cryptosystems without requiring their immediate replacement.

SLH-DSA signatures are substantially larger than ML-DSA or traditional signatures. ACME servers that accept SLH-DSA-authenticated JWS requests will have to account for the increased message size in request-size limits and resource provisioning, so that the larger signatures do not become a denial-of-service vector.

## MITM Attacks with CRQC in ACME

A MITM attack can occur if an adversary possesses a CRQC capable of breaking traditional public-key signature schemes. In such a scenario, the adversary can generate forged certificates and produce valid signatures, thereby impersonating a TLS peer (server or client). Consequently, the authentication guarantees of TLS have to be considered compromised when relying solely on traditional certificate-based authentication once a CRQC is available.

The normative measures that mitigate this threat are specified in the mechanism sections of this document and are not restated here as independent requirements: The Quantum-Ready mechanism requires support for PQC-only or PQ/T hybrid authentication for both TLS server authentication ({{tls-server-authentication}}) and JWS request authentication ({{jws-request-authentication}}), and permits traditional-only authentication only during an explicitly bounded transition period.

Organizations operating PKI infrastructures are expected to define and implement a transition strategy toward issuing PQC or PQ/T hybrid certificates, phasing out certificates that rely exclusively on traditional cryptographic algorithms within a defined migration timeline as part of their quantum-readiness posture.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgements

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{I-D.ietf-uta-pqc-app}}.
