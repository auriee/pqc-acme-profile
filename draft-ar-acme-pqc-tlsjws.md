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
    email: "kondtir@gmail.com"

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
ACME {{8555}} leverages TLS 1.2, JWS signatures primarily for authentication and signature generation that employs the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for domain control validation. However, TLS 1.2 as well as current JWS signatures is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in ACME.

This draft aims to enhance the security of ACME protocol by defining TLS 1.3 and JWS PQC profile for ACME to make it quantum-safe {{RFC8555}}.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME) {{RFC8555}} specification describes methods for validating control of domain names via HTTP and DNS.  Deployment experience has shown it is also useful to be able to validate domain control using the TLS layer alone.  In particular, this allows hosting providers, Content Distribution Networks (CDNs), and TLS-terminating load balancers to validate domain control without modifying the HTTP handling behavior of their backends as explained in {{RFC8737}}.

When using protocols such as TLS 1.3 {{RFC8446}}, DTLS 1.3 {{RFC9147}}, or protocols built on these foundations like ACME {{RFC8555}}, clients and servers perform ephemeral public-key exchanges, such as Elliptic Curve Diffie-Hellman (ECDH), to derive a shared secret that ensures forward secrecy. Additionally, they validate each other's identities through X.509 certificates, establishing secure communication. The emergence of a Cryptographically Relevant Quantum Computers (CRQCs) would render current public-key algorithms insecure and obsolete. This is because the mathematical assumptions underpinning these algorithms, which currently offer high levels of security, would no longer hold in the presence of a CRQC.  Consequently, there is an urgent need to update protocols and infrastructure with post-quantum cryptographic (PQC) algorithms.  These algorithms are designed to remain secure against both CRQCs and classical computers. The traditional cryptographic primitives requiring replacement are discussed in {{?I-D.ietf-pquip-pqc-engineers}}, and the NIST PQC Standardization process has selected algorithms such as ML-KEM, SLH-DSA, and ML-DSA as candidates for future deployment in protocols.

Historically, the industry has successfully transitioned between cryptographic protocols, such as upgrading TLS versions and deprecating older ones (e.g., SSLv2), and shifting from RSA to Elliptic Curve Cryptography (ECC), which improved security and reduced key sizes.  However, the transition to ACME PQC presents a unique challenge, primarily due to the algorithm maturity; while NIST has finalized a set of PQC algorithms, ensuring the correctness and security of implementations remains critical.  Even the most secure algorithm is vulnerable if implementation flaws introduce security risks.

Any application transmitting messages over untrusted networks is potentially vulnerable to active or passive attacks by adversaries, including those equipped with CRQCs. The degree of vulnerability varies depending on the application, the underlying systems, the value of the data being transmitted, and the attractiveness of attacking a particular individual, device, or flow. This document outlines quantum-ready usage profiles for applications designed to protect against passive and on-path attacks leveraging CRQCs.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.
This document also adopts terminology defined in {{RFC9794}}.

For the purposes of this document, it is useful to categorize cryptographic algorithms into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer factorization, finite field discrete logarithms, or elliptic curve discrete logarithms. In the context of TLS, an example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure against attacks from both quantum and classical computers. An example of a post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism (ML-KEM). Such algorithms rely on mathematical problems (e.g., lattices) that are believed to be hard for both classical and CRQCs to solve efficiently.

* Hybrid Algorithm: We distinguish between key exchanges and signature algorithms:

  - Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms - one traditional algorithm and one post-quantum algorithm. The resulting shared secret remains secure as long as at least one of the component key exchange algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed of two or more component signature algorithms, where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

Digital signature algorithms play a critical role in X.509 certificates, Certificate Transparency Signed Certificate Timestamps, Online Certificate Status Protocol (OCSP) statements, remote attestation evidence, and any other mechanism that contributes signatures during a TLS handshake or in context of a secure communication establishment.

# Data Confidentiality in ACME

One of the primary threats to TLS in ACME is the HNDL (Harvest Now Decyrpt Later) attack. In this scenario, adversaries can passively capture TLS handshakes such as those transmitted over the air and store them for future decryption once CRQCs become available.

If the handshake is not quantum-resistant, a future CRQC could retroactively decrypt session traffic, revealing:

* The identity of the authenticated client.

* Client credentials used in certificate-based authentication (e.g., usernames, URLs, device or organization identifiers).

* Session-specific metadata that may aid surveillance or profiling, such as cipher suites and TLS extensions supported. These elements can help to fingerprint devices, or correlate ACME TLS sessions over time.

To preserve the intended privacy guarantees of TLS 1.3 and protect against HNDL, ACME TLS deployments MUST adopt post-quantum key exchange mechanisms, as outlined in Section 4 of [I-D.reddy-uta-pqc-app]. These mechanisms ensure that even if handshake data is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

## Threats to Personally Identifiable Information (PII):

For a client of the ACME protocol, an critical threat to PII from a CRQC is the ability of future quantum computers to break current public-key cryptography. This would enable a HNDL attack, where adversaries collect encrypted data today and store it until sufficiently powerful quantum computers are available to decrypt it. Following are the key threats to an ACME Client PII:

* Decryption of harvested data: Standard encryption methods like ECDH, RSA, which the ACME protocol and Transport Layer Security (TLS) rely on, will be vulnerable to quantum computers running algorithms such as Shor's algorithm.

* Compromised digital signatures: Quantum attacks could allow an adversary to forge digital signatures, enabling impersonation of the certificate authority. This would let them issue fraudulent certificates to tamper with or compromise data.

* Breach of long-lived data: PII, such as financial or health data, that is encrypted today but has a long lifespan could be exposed years from now.

* Weakened supply chain: Vulnerabilities could propagate through a system's ecosystem, particularly in Public Key Infrastructure (PKI) used by ACME.

# Quantum-Ready Authentication

Although CRQCs could potentially decrypt past TLS sessions, client/server authentication based on certificates cannot be retroactively compromised. However, the multi-year process required to establish, certify, and embed new root CAs presents a significant challenge. If CRQCs emerge earlier than anticipated, responding promptly to secure authentication systems would be difficult. While the migration to PQ X.509 certificates allows for more time compared to key exchanges, delaying these preparations should be avoided.

For data authentication, the concern shifts to potential on-path attackers equipped with CRQCs capable of breaking certificate-based authentication mechanisms that rely on traditional algorithms. Such attackers could impersonate legitimate entities, tricking victims into connecting to the attacker’s device instead of the intended target, resulting in impersonation attacks. 

While this is not as immediate a threat as "Harvest Now, Decrypt Later" attacks, it remains a significant risk that must be addressed proactively. In client/server certificate-based authentication, the security window between the generation of the signature in the CertificateVerify message and its verification by the peer during the TLS handshake is typically short.  However, the security lifetime of digital signatures on X.509 certificates, including those issued by root Certification Authorities (CAs), warrants closer scrutiny.  Root CA certificates can have validity periods of 20 years or more, while root Certificate Revocation Lists (CRLs) often remain valid for a year or longer.  Delegated credentials, such as CRL Signing Certificates or OCSP response signing certificates, generally have shorter lifetimes but still present a potential vulnerability window.

The quantum-ready authentication property becomes critical in scenarios where an on-path attacker uses network devices equipped with CRQCs to break traditional authentication protocols and ensures robust authentication through the use of either a pure Post-Quantum certificate or a PQ/T hybrid certificate.

The TLS handshake MUST negotiate a PQC-only or hybrid key exchange. The client’s JWS signature MUST be computed using a PQC-only or composite signature algorithm. Any fallback to classical-only TLS or signature mechanisms MUST be rejected by the server.

ACME servers MUST implement TLS 1.3 with support for PQC-only or hybrid key exchanges (e.g., X25519+Kyber768) and MUST NOT permit fallback to TLS 1.2 or below for certificate-management transactions. ACME servers that support 0-RTT MAY allow clients to send early data, provided that: The session resumption ticket and key exchange are derived from a quantum-resistant handshake. The ACME request includes nonce-based replay protection (Section 6.5 of RFC 8555). If these conditions are not met, clients MUST avoid using 0-RTT.

An ACME server MUST support at least one PQC-only or composite signature algorithm for signing and verifying JWS objects.

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

# PQC TLS with ALPN Challenge

The TLS with Application-Layer Protocol Negotiation (TLS ALPN) validation method proves control over a domain name by requiring the ACME client to configure a TLS server to respond to specific connection attempts using the ALPN extension with identifying information.  The ACME server validates control of the domain name by connecting to a TLS server at one of the addresses resolved for the domain name and verifying that a certificate with specific content is presented.

Once the tls-alpn-01 ACME challenge certificate has been created (as defined in {{RFC8737}}), it MUST be provisioned such that it is returned during a TLS handshake where the "acme-tls/1" application-layer protocol has been negotiated and a Server Name Indication (SNI) extension {{RFC6066}} has been provided containing the domain name being validated.

A client responds by POSTing an empty JSON object ({}) to the challenge URL to acknowledge that the challenge is ready to be validated by the server. The base64url encoding of the protected headers and payload is described in Section 6.1 of {{RFC8555}}.

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
   
On receiving this request from a client, the server constructs and stores the key authorization from the challenge "token" value and the current client account key and then the server then verifies the client's control over the domain by verifying that the TLS server was configured as expected.

# acme-tls/1 definition

The "acme-tls/1" protocol MUST only be used for validating ACME tls-alpn-01 challenges.  The protocol consists of a TLS handshake in which the required validation information is transmitted.  The "acme-tls/2" protocol does not carry application data.  Once the handshake is completed, the client MUST NOT exchange any further data with the server and MUST immediately close the connection.  While this protocol uses X.509 certificates, it does not use the authentication method described in {{RFC5280}} and, as such, does not require a valid signature on the provided certificate nor require the TLS handshake to complete successfully.

An ACME server may wish to use an off-the-shelf TLS stack where it is not simple to allow these divergences in the protocol as defined.  Because of this, an ACME server MAY choose to withhold authorization if either the certificate signature is invalid or the handshake doesn't fully complete. ACME servers that implement "acme-tls/2" MUST only negotiate TLS 1.3 {{RFC8446}} only when connecting to clients for validation as TLS 1.3 has support for post quantum enhancements.

# Security Considerations

The security considerations outlined in {{?I-D.ietf-pquip-pqc-engineers}} must be carefully evaluated and taken into account.

Post-quantum algorithms selected for standardization are relatively new, and their implementations are still in the early stages of
maturity. This makes them more susceptible to implementation bugs compared to the well-established and extensively tested cryptographic
algorithms currently in use. Furthermore, certain deployments may need to continue using traditional algorithms to meet regulatory
requirements, such as Federal Information Processing Standard (FIPS) {{SP-800-56C}} or Payment Card Industry (PCI) compliance.

Hybrid key exchange provides a practical and flexible solution, offering protection against "Harvest Now, Decrypt Later" attacks while
ensuring resilience to potential catastrophic vulnerabilities in any single algorithm. This approach allows for a gradual
transition to PQC, preserving the benefits of traditional cryptosystems without requiring their immediate replacement.

## MITM Attacks with CRQC 

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

This draft leverages text from {{RFC8555}}, {{RFC8737}} and {{?I-D.reddy-uta-pqc-app}}.
