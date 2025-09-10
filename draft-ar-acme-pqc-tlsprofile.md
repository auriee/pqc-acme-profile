---
title: "Post Quantum Enhancments in ACME TLS profile"
abbrev: "PQC TLS profile in ACME"
category: std

docname: draft-ar-acme-pqc-tlsprofile-00
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
ACME TLS {{RFC8737}} leverages TLS 1.2 that employs the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for domain control validation. However, TLS 1.2 it is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in ACME.

This draft aims to enhance the security of ACME TLS protocol by defining TLS 1.3 PQC profile to make it quantum-safe {{RFC8737}}.

--- middle

# Introduction

The Automatic Certificate Management Environment (ACME) {{RFC8555}} specification describes methods for validating control of domain names via HTTP and DNS.  Deployment experience has shown it is also useful to be able to validate domain control using the TLS layer alone.  In particular, this allows hosting providers, Content Distribution Networks (CDNs), and TLS-terminating load balancers to validate domain control without modifying the HTTP handling behavior of their backends as explained in {{RFC8737}}.

When using protocols such as TLS 1.3 {{RFC8446}}, DTLS 1.3 {{RFC9147}}, or protocols built on these foundations like ACME {{RFC8555}}, clients and servers perform ephemeral public-key exchanges, such as Elliptic Curve Diffie-Hellman (ECDH), to derive a shared secret that ensures forward secrecy. Additionally, they validate each other's identities through X.509 certificates, establishing secure communication. The emergence of a Cryptographically Relevant Quantum Computer (CRQC) would render current public-key algorithms insecure and obsolete. This is because the mathematical assumptions underpinning these algorithms, which currently offer high levels of security, would no longer hold in the presence of a CRQC.  Consequently, there is an urgent need to update protocols and infrastructure with post-quantum cryptographic (PQC) algorithms.  These algorithms are designed to remain secure against both CRQCs and classical computers. The traditional cryptographic primitives requiring replacement are discussed in {{?I-D.ietf-pquip-pqc-engineers}}, and the NIST PQC Standardization process has selected algorithms such as ML-KEM, SLH-DSA, and ML-DSA as candidates for future deployment in protocols.

Historically, the industry has successfully transitioned between cryptographic protocols, such as upgrading TLS versions and deprecating older ones (e.g., SSLv2), and shifting from RSA to Elliptic Curve Cryptography (ECC), which improved security and reduced key sizes.  However, the transition to PQC presents unique challenges, primarily due to the following:

   1.  Algorithm Maturity: While NIST has finalized a set of PQC
       algorithms, ensuring the correctness and security of
       implementations remains critical.  Even the most secure algorithm
       is vulnerable if implementation flaws introduce security risks.

   2.  Key and Signature Sizes: Many PQC algorithms require
       significantly larger key and signature sizes, which can inflate
       handshake packet sizes and impact network performance.  For
       example, ML-KEM public keys are substantially larger than ECDH
       keys (see Table 5 in {{?I-D.ietf-pquip-pqc-engineers}}). Similarly,
       public keys for SLH-DSA and ML-DSA are much larger than those for
       P256 (see Table 6 in {{?I-D.ietf-pquip-pqc-engineers}}). Signature
       sizes for algorithms like SLH-DSA and ML-DSA are also
       considerably larger compared to traditional options like Ed25519
       or ECDSA-P256, posing challenges for constrained environments
       (e.g., IoT) and increasing handshake times in high-latency or
       lossy networks.

   3.  Performance Trade-Offs: While some PQC algorithms exhibit slower
       operations compared to traditional algorithms, others provide
       specific advantages.  For instance, ML-KEM requires less CPU than
       X25519, and ML-DSA offers faster signature verification times
       compared to Ed25519, although its signature generation process is
       slower.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.
This document also adopts terminology defined in {{RFC9794}}.

# PQC TLS with ALPN Challenge

The TLS with Application-Layer Protocol Negotiation (TLS ALPN) validation method proves control over a domain name by requiring the ACME client to configure a TLS server to respond to specific connection attempts using the ALPN extension with identifying information.  The ACME server validates control of the domain name by connecting to a TLS server at one of the addresses resolved for the domain name and verifying that a certificate with specific content is presented.
The tls-alpn-02 ACME challenge object has the following format:

   type (required, string):  The string "tls-alpn-02"

   token (required, string):  A random value that uniquely identifies
      the challenge.  This value MUST have at least 128 bits of entropy.
      It MUST NOT contain any characters outside the base64url alphabet
      as described in Section 5 of {{RFC4648}}.  Trailing '=' padding
      characters MUST be stripped.  See {{RFC4086}} for additional
      information on randomness requirements.

The client prepares for validation by constructing a self-signed certificate that MUST contain an acmeIdentifier extension and a subjectAlternativeName extension {{RFC5280}}.  The subjectAlternativeName extension MUST contain a single dNSName entry where the value is the domain name being validated.  The acmeIdentifier extension MUST contain the SHA-256 digest {{FIPS180-4}} of the key authorization {{RFC8555}} for the challenge.  The acmeIdentifier extension MUST be critical so that the certificateisn't inadvertently used by non-ACME software.

   The acmeIdentifier extension is identified by the id-pe-
   acmeIdentifier object identifier (OID) in the id-pe arc {{RFC5280}}:

   id-pe-acmeIdentifier OBJECT IDENTIFIER ::=  { id-pe 31 }

   The extension has the following ASN.1 [X.680] format :

   Authorization ::= OCTET STRING (SIZE (32))

   The extnValue of the id-pe-acmeIdentifier extension is the ASN.1 DER
   encoding [X.690] of the Authorization structure, which contains the
   SHA-256 digest of the key authorization for the challenge.

Once this certificate has been created, it MUST be provisioned such that it is returned during a TLS handshake where the "acme-tls/2" application-layer protocol has been negotiated and a Server Name Indication (SNI) extension {{RFC6066}} has been provided containing the domain name being validated.

A client responds by POSTing an empty JSON object ({}) to the challenge URL to acknowledge that the challenge is ready to be validated by the server.  The base64url encoding of the protected headers and payload is described in Section 6.1 of {{RFC8555}}.

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

   On receiving this request from a client, the server constructs and stores the key authorization from the challenge "token" value and the current client account key.

   The server then verifies the client's control over the domain by verifying that the TLS server was configured as expected using the following steps:

   1.  The ACME server computes the expected SHA-256 digest of the key
       authorization.

   2.  The ACME server resolves the domain name being validated and
       chooses one of the IP addresses returned for validation (the
       server MAY validate against multiple addresses if more than one
       is returned).

   3.  The ACME server initiates a TLS connection to the chosen IP
       address.  This connection MUST use TCP port 443.  The ACME server
       MUST provide an ALPN extension with the single protocol name
       "acme-tls/2" and an SNI extension containing only the domain name
       being validated during the TLS handshake.

   4.  The ACME server verifies that during the TLS handshake the
       application-layer protocol "acme-tls/2" was successfully
       negotiated (and that the ALPN extension contained only the value
       "acme-tls/2") and that the certificate returned contains:

       *  a subjectAltName extension containing the dNSName being
          validated and no other entries

       *  a critical acmeIdentifier extension containing the expected
          SHA-256 digest computed in step 1

   The comparison of dNSNames MUST be case insensitive {{RFC4343}}.  Note that as ACME doesn't support Unicode identifiers, all dNSNames MUST be encoded using the rules of [RFC3492].

   If all of the above steps succeed, then the validation is successful. Otherwise, it fails.

# acme-tls/2 definition

The "acme-tls/2" protocol MUST only be used for validating ACME tls-alpn-02 challenges.  The protocol consists of a TLS handshake in which the required validation information is transmitted.  The "acme-tls/2" protocol does not carry application data.  Once the handshake is completed, the client MUST NOT exchange any further data with the server and MUST immediately close the connection.  While this protocol uses X.509 certificates, it does not use the authentication method described in {{RFC5280}} and, as such, does not require a valid signature on the provided certificate nor require the TLS handshake to complete successfully.

An ACME server may wish to use an off-the-shelf TLS stack where it is not simple to allow these divergences in the protocol as defined.  Because of this, an ACME server MAY choose to withhold authorization if either the certificate signature is invalid or the handshake doesn't fully complete. ACME servers that implement "acme-tls/2" MUST only negotiate TLS 1.3 {{RFC8446}} only when connecting to clients for validation as TLS 1.3 has support for post quantum enhancements.

# IANA Considerations

--- back

# Acknowledgements

This draft leverages text from {{RFC8737}} and {{?I-D.reddy-uta-pqc-app}}.