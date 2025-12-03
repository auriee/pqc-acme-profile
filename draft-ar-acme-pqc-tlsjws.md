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

ACME defines multiple domain control validation methods, such as HTTP-01, DNS-01, and TLS-based challenges like tls-alpn-01 {{RFC8737}}. In all cases, ACME depends on classical public-key cryptography, both in JWS for client request authentication and in TLS for server authentication and session key establishment.

The emergence of Cryptographically Relevant Quantum Computers (CRQCs) threatens the security of these mechanisms. Quantum algorithms, such as Shor’s algorithm, would render widely deployed public-key algorithms (e.g., RSA, ECDSA, and ECDH) insecure. Consequently, there is an need to transition ACME to post-quantum cryptography (PQC) to ensure long-term security.

This document defines post-quantum and hybrid PQ/T cryptographic profiles for ACME. These include PQC algorithms for JWS and PQC-enabled TLS 1.3 profiles for transport security. Together, these enhancements provide quantum-resistant authentication and key establishment while maintaining interoperability with existing ACME deployments.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.
This document also adopts terminology defined in {{RFC9794}}.

For the purposes of this document, it is useful to categorize cryptographic algorithms into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer factorization, finite field discrete logarithms, or elliptic curve discrete logarithms. In the context of TLS, an example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure against attacks from both quantum and classical computers. An example of a post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism (ML-KEM). Such algorithms rely on mathematical problems (e.g., lattices) that are believed to be hard for both classical and CRQCs to solve efficiently.

* Hybrid Algorithm: We distinguish between key exchanges and signature algorithms:

  - Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms - one traditional algorithm and one post-quantum algorithm. The resulting shared secret remains secure as long as at least one of the component key exchange algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed of two or more component signature algorithms, where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

# Data Confidentiality in ACME

In ACME, clients may transmit contact information** such as email addresses and other metadata when creating or managing an account. The confidentiality of this information therefore relies entirely on the security of the TLS channel, which provides protection against passive observation during transmission.

One of the primary threats to TLS in ACME is the Harvest-Now, Decrypt-Later (HNDL) attack. In this scenario, an adversary passively records encrypted TLS traffic, such as ACME account creation or order requests and stores it for future decryption once Cryptographically Relevant Quantum Computers (CRQCs) become available.

CRQCs pose a long-term confidentiality risk because they can break the traditional public-key algorithms (e.g., RSA, ECDH) used in TLS handshakes to establish session keys. Once these algorithms are broken, previously captured TLS sessions can be decrypted, exposing sensitive or personally identifiable information (PII) contained within ACME messages.

To preserve the intended privacy guarantees of TLS 1.3 and protect against HNDL, ACME deployments MUST adopt post-
quantum key exchange mechanisms, as outlined in Section 4 of {{I-D.reddy-uta-pqc-app}}.  These mechanisms ensure that even if TLS session is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

ACME servers that support 0-RTT MAY allow clients to send early data, provided that: The session resumption ticket and key exchange are derived from a quantum-resistant handshake. The ACME request includes nonce-based replay protection (Section 6.5 of RFC 8555). If these conditions are not met, clients MUST avoid using 0-RTT.

An ACME server MUST support at least one PQC-only or composite signature algorithm for signing and verifying JWS objects.

# Quantum-Ready Authentication

Although CRQCs could potentially decrypt past TLS sessions, client/server authentication based on certificates cannot be retroactively compromised. However, the multi-year process required to establish, certify, and embed new root CAs presents a significant challenge. If CRQCs emerge earlier than anticipated, responding promptly to secure authentication systems would be difficult. While the migration to PQ X.509 certificates allows for more time compared to key exchanges, delaying these preparations should be avoided.

For an ACME server in data authentication, the concern shifts to potential on-path attackers equipped with CRQCs capable of breaking certificate-based authentication mechanisms that rely on traditional algorithms. Such attackers could impersonate legitimate entities, tricking victims into connecting to the attacker’s device instead of the intended target, resulting in impersonation attacks. 

The quantum-ready authentication property becomes critical in scenarios where an on-path attacker uses network devices equipped with CRQCs to break traditional authentication protocols and ensures robust authentication through the use of either a pure Post-Quantum certificate or a PQ/T hybrid certificate.

The client’s JWS signature MUST be computed using a PQC-only or composite signature algorithm. Any fallback to classical-only TLS or signature mechanisms MUST be rejected by the server.

## Post-Quantum X.509 Certificates

Post-quantum certificates contain only a PQC public key and are signed using a post-quantum algorithm. They are suitable for deployments capable of fully embracing post-quantum cryptography.

  * ML-DSA Certificates: Defined in {{?I-D.ietf-lamps-dilithium-certificates}}, these use the Module-Lattice Digital Signature Algorithm (ML-DSA). {{?I-D.tls-westerbaan-mldsa}} explains how ML-DSA is applied for authentication in TLS 1.3.
  * SLH-DSA Certificates: Defined in {{?I-D.ietf-lamps-x509-slhdsa}}, these use the SLH-DSA algorithm. {{?I-D.reddy-tls-slhdsa}} details how SLH-DSA is used in TLS 1.3 and compares its advantages and disadvantages with ML-DSA in Section 2 of the document.

## Post-Quantum JSON Web Signatures (JWS)

Communications between an ACME client and an ACME server are done over HTTPS, using JSON Web Signature (JWS) [RFC7515] to provide some additional security properties for messages sent from the client to the server.

  * ML-DSA JWS: Defined in Section 5 of {{?I-D.ietf-cose-dilithium-09}}, these employ ML-DSA updates to JWS updates to quantum safe JOSE signatures. ACME needs to meet the requirements and updates to JWS to generate quantum safe signatures over HTTPS using JSON.
  * SLH-DSA JWS: Defined in Section 3 of {{?I-D.ietf-cose-sphincs-plus-05}}, these employ SLH-DSA updates to JWS updates to quantum safe JOSE signatures. ACME needs to meet the requirements and updates to JWS to generate quantum safe signatures over HTTPS using JSON.

ACME needs to either leverage ML-DSA or SLH-DSA specifications for JWS instead of RFC7515 for quantum safe signatures and message transport.

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

When the DNS-01 challenge type is used, the validation chain SHOULD rely on DNSSEC records signed using PQC-only or composite signature algorithms. CAs and ACME servers SHOULD reject DNSSEC validations that depend solely on RSA or ECDSA algorithms without PQC augmentation, as these signatures will be vulnerable once CRQCs are available.

ACME servers MAY publish policy metadata in their directory object indicating which DNSSEC algorithms are acceptable.

## URLs with PQC

The URLs used in ACME needs to use PQC-only or hybrid key exchange and PQC-only or composite signatures. All URLs provided by the ACME directory (e.g., newAccount, newOrder, revokeCert) MUST be accessible over HTTPS endpoints that:
 
* Negotiate PQC-only or hybrid key exchange mechanisms;
 
* Authenticate using PQC-only or composite signatures in the server’s TLS certificate chain; Present a server certificate whose key material is quantum-safe or composite (e.g., X25519+Kyber, Ed25519+ML-DSA).

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

The "alg" algorithm can be any of the following tables based on the requirements and security level desired as defined {{?I-D.ietf-pquip-pqc-engineers}}.
 
 
# Security Considerations

The security considerations outlined in {{?I-D.ietf-pquip-pqc-engineers}} must be carefully evaluated and taken into account.

Post-quantum algorithms selected for standardization are relatively new, and their implementations are still in the early stages of
maturity. This makes them more susceptible to implementation bugs compared to the well-established and extensively tested cryptographic
algorithms currently in use. Furthermore, certain deployments may need to continue using traditional algorithms to meet regulatory
requirements, such as Federal Information Processing Standard (FIPS) {{SP-800-56C}} or Payment Card Industry (PCI) compliance.

Hybrid key exchange provides a practical and flexible solution, offering protection against "Harvest Now, Decrypt Later" attacks while
ensuring resilience to potential catastrophic vulnerabilities in any single algorithm. This approach allows for a gradual
transition to PQC, preserving the benefits of traditional cryptosystems without requiring their immediate replacement.

## MITM Attacks with CRQC in ACME

A MITM attack is possible if an adversary possesses a CRQC capable of breaking traditional public-key signatures. The attacker can generate
a forged certificate and create a valid signature, enabling them to impersonate a TLS peer, whether a server or a client. This completely undermines the authentication
guarantees of TLS when relying on traditional certificates.

To mitigate such attacks, several steps need to be taken:

1. Revocation and Transition: Both clients and servers that use traditional certificates will have to revoke them and migrate to PQC authentication.
2. Client-Side Verification:  Clients should avoid establishing TLS sessions with servers that do not support PQC authentication
3. Server-Side Verification: Server should avoid establishing JSON based certificate sessions from clients that do not support quantum safe JWS/JWK authentication
4. PKI Migration: Organizations should transition their PKI to post-quantum-safe certification authorities and discontinue issuing certificates based on traditional cryptographic methods.

# IANA Considerations

--- back

# Acknowledgements

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{?I-D.ietf-uta-pqc-app}}.
