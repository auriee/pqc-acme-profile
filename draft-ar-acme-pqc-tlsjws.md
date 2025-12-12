---
title: "Post Quantum Enhancments in ACME profile"
abbrev: "PQC profile in ACME"
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
    FIPS180-4:
        title: "FIPS180-4: Secure Hash Standard (SHS)"
        target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
    


--- abstract

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) to authenticate client request payloads. Current ACME deployments rely on traditional public-key algorithms in both JWS and TLS, which are vulnerable to attacks from Cryptographically Relevant Quantum Computers (CRQCs). This document defines post-quantum (PQC) and hybrid post-quantum/traditional (PQ/T) cryptography profiles for ACME, including PQC and PQ/T JWS algorithms, as well as PQC-enabled TLS 1.3 profiles for transport security. These profiles enable ACME clients and servers to achieve quantum-resistant authentication and key establishment.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME {{RFC8555}}) automates certificate issuance, validation, and management over HTTPS using JSON Web Signatures (JWS) for authenticating client requests to the Certificate Authority (CA). ACME relies on TLS to provide confidentiality, integrity, and authentication of the ACME server, ensuring secure transport of protocol messages.

ACME defines multiple domain control validation methods, such as HTTP-01, DNS-01, and TLS-based challenges like tls-alpn-01 {{RFC8737}}. In all cases, ACME depends on traditional public-key cryptography, both in JWS for client request authentication and in TLS for server authentication and session key establishment.

The emergence of Cryptographically Relevant Quantum Computers (CRQCs) threatens the security of these mechanisms. Quantum algorithms, such as Shor’s algorithm, would render widely deployed public-key algorithms (e.g., RSA, ECDSA, and ECDH) insecure. Consequently, there is an need to transition ACME to post-quantum cryptography (PQC) to ensure long-term security.

This document defines post-quantum and hybrid PQ/T cryptographic profiles for ACME. These include PQC algorithms for JWS and PQC-enabled TLS 1.3 profiles for transport security. Together, these enhancements provide quantum-resistant authentication and key establishment.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This document also adopts terminology defined in {{RFC9794}}.

For the purposes of this document, it is useful to categorize cryptographic algorithms into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer factorization, finite field discrete logarithms, or elliptic curve discrete logarithms. In the context of TLS, an example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure against attacks from both quantum and classical computers. An example of a post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism (ML-KEM). Such algorithms rely on mathematical problems (e.g., lattices) that are believed to be hard for both classical and CRQCs to solve efficiently.

* Hybrid Algorithm: The distinction is made between key exchange algorithms and signature algorithms:

  - PQ/T Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms - one traditional algorithm and one post-quantum algorithm. The resulting shared secret remains secure as long as at least one of the component key exchange algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed of two or more component signature algorithms, where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

# Data Confidentiality in ACME

In ACME, clients may transmit personally identifiable information (PII) such as email addresses and other account-related metadata when creating or managing an account. The confidentiality of this information relies on the security of the TLS channel, which protects ACME exchanges from passive observation during transmission.

One of the primary threats to TLS in ACME is the Harvest-Now, Decrypt-Later (HNDL) attack. In this scenario, an adversary passively records encrypted TLS traffic, such as ACME account creation or order requests and stores it for future decryption once Cryptographically Relevant Quantum Computers (CRQCs) become available.

CRQCs pose a long-term confidentiality risk because they can break the traditional public-key algorithms (e.g., RSA, ECDH) used in TLS handshakes to establish session keys. Once these algorithms are broken, previously captured TLS sessions can be decrypted, exposing sensitive or personally identifiable information (PII) contained within ACME messages.

To preserve the intended privacy guarantees of TLS 1.3 and protect against HNDL, ACME deployments MUST adopt post-
quantum key exchange mechanisms, as outlined in Section 4 of {{I-D.reddy-uta-pqc-app}}.  These mechanisms ensure that even if TLS session is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

ACME servers that support 0-RTT MAY allow clients to send early data, provided that: The session resumption ticket and key exchange are derived from a quantum-resistant handshake. The ACME request includes nonce-based replay protection (Section 6.5 of RFC 8555). If these conditions are not met, clients MUST avoid using 0-RTT.

# Quantum-Ready Authentication

Although CRQCs could potentially decrypt past TLS sessions, client/server authentication based on certificates cannot be retroactively compromised. However, the multi-year process required to establish, certify, and embed new root CAs presents a significant challenge. If CRQCs emerge earlier than anticipated, responding promptly to secure authentication systems would be difficult. In practice, the availability of a CRQC may not be publicly disclosed. Similar to a zero-day vulnerability, an adversary could exploit quantum capabilities privately to
compromise traditional certificates without alerting the wider ecosystem. While the migration to PQ X.509 certificates allows for more time compared to key exchanges, delaying these preparations should be avoided.

For an ACME server in data authentication, the concern shifts to potential on-path attackers equipped with CRQCs capable of breaking certificate-based authentication mechanisms that rely on traditional algorithms. Such attackers could impersonate legitimate entities, tricking victims into connecting to the attacker’s device instead of the intended target, resulting in impersonation attacks. 

The quantum-ready authentication property becomes critical in scenarios where an on-path attacker uses CRQCs to break traditional authentication protocols. Robust authentication requires the use of either a PQC-only certificate or a PQ/T hybrid certificate for TLS server authentication, ensuring resistance to quantum attacks.

Similarly, ACME request authentication depends on the integrity of JWS objects. Therefore, the client’s JWS signature MUST be computed using a PQC-only or PQ/T hybrid signature algorithm.

To guarantee quantum-resistant authentication for both transport and application-layer mechanisms:

TLS authentication and JWS request authentication MUST use PQC-only or PQ/T hybrid signature algorithms, and any fallback to traditional-only certificate signatures or traditional-only JWS signatures MUST be rejected by the server.

## Post-Quantum X.509 Certificates

Post-quantum certificates contain only a PQC public key and are signed using a post-quantum algorithm. They are suitable for deployments capable of fully embracing post-quantum cryptography.

  * ML-DSA Certificates: Defined in {{?I-D.ietf-lamps-dilithium-certificates}}, these use the Module-Lattice Digital Signature Algorithm (ML-DSA). {{?I-D.tls-westerbaan-mldsa}} explains how ML-DSA is applied for authentication in TLS 1.3.
  * SLH-DSA Certificates: Defined in {{?I-D.ietf-lamps-x509-slhdsa}}, these use the SLH-DSA algorithm. {{?I-D.reddy-tls-slhdsa}} details how SLH-DSA is used in TLS 1.3 and compares its advantages and disadvantages with ML-DSA in Section 2 of the document.

## Post-Quantum JSON Web Signatures (JWS)

ACME clients communicate with ACME servers over HTTPS; ACME requires that client requests be integrity-protected and authenticated using JSON Web Signature (JWS) as specified in {{RFC7515}}. 

  * ML-DSA JWS: Defined in Section 5 of {{?I-D.ietf-cose-dilithium-09}}. This draft specifies JOSE algorithm identifiers and processing rules for ML-DSA signature variants, enabling the use of ML-DSA as a post-quantum signature algorithm in JWS objects for ACME client authentication..

  * SLH-DSA JWS: Defined in Section 3 of {{?I-D.ietf-cose-sphincs-plus-05}}. This draft specifies JOSE algorithm identifiers and processing rules for SLH-DSA signature variants, enabling the use of SLH-DSA as a post-quantum signature algorithm in JWS objects for ACME client authentication.

ACME implementations that intend to provide quantum-resistant client authentication will have to support one or more of the post-quantum JWS algorithm identifiers defined in these specifications. 

For ML-DSA based JSON Web Signatures:


| Name      | alg       | Description                                |
| --------- | --------- | ------------------------------------------ |
| ML-DSA-44 | ML-DSA-44 | JSON Web Signature Algorithm for ML-DSA-44 |
| ML-DSA-65 | ML-DSA-65 | JSON Web Signature Algorithm for ML-DSA-65 |
| ML-DSA-87 | ML-DSA-87 | JSON Web Signature Algorithm for ML-DSA-87 |

For SLH-DSA based JSON Web Signatures:


| Name               | alg                | Description                                          |
| ------------------ | ------------------ | ---------------------------------------------------- |
| SLH-DSA-SHA2-128s  | SLH-DSA-SHA2-128s  | JSON Web Signature Algorithm for SLH-DSA-SHA2-128s   |
| SLH-DSA-SHAKE-128s | SLH-DSA-SHAKE-128s | JSON Web Signature Algorithm for SLH-DSA-SHAKE-128s  |
| SLH-DSA-SHA2-128f  | SLH-DSA-SHA2-128f  | JSON Web Signature Algorithm for SLH-DSA-SHA2-128f   |


## DNSSEC with PQC Signatures

When the DNS-01 challenge type is used, ACME servers rely on DNSSEC to authenticate the DNS records associated with the validation. However, the DNSSEC ecosystem does not yet have standardized use of post-quantum signature algorithms, and the IETF is still evaluating the feasibility and operational impact of introducing PQC into DNSSEC. As a result, DNSSEC today remains dependent on traditional algorithms such as RSA and ECDSA, which are vulnerable to future CRQCs.

Given these limitations, ACME servers will have to assess the strength of the DNSSEC validation chain and treat DNSSEC dependencies on traditional-only algorithms as providing no long-term authentication guarantees in the presence of CRQCs. Once DNSSEC supports PQC, ACME servers and CAs are expected to migrate DNS-01 validation policies accordingly.


## URLs with PQC

The URLs used in ACME needs to use PQC-only or PQ/T hybrid key exchange and PQC-only or composite signatures. All URLs provided by the ACME directory (e.g., newAccount, newOrder, revokeCert) MUST be accessible over HTTPS endpoints that:
 
* Negotiate PQC-only or PQ/T hybrid key exchange mechanisms;
 
* Authenticate the server using a TLS certificate chain that employs PQC-only or PQ/T hybrid signature algorithms.
This includes presenting a server certificate whose public-key material and signature algorithm offer PQC-only or PQ/T hybrid signatures (for example, X25519+ML-KEM for key exchange, Ed25519+ML-DSA for signatures).

# Quantum Safe ACME JWS signature

Following is an example for ACME JWS signature:

POST /acme/authz/1234/2
Host: example.com
Content-Type: application/jose+json

{
  "protected": base64url({
    "alg": "ML-DSA44",
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

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{?I-D.ietf-uta-pqc-app}}.
