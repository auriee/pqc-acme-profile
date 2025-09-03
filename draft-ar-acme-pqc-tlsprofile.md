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

--- back

# Acknowledgements

This draft leverages text from {{RFC8737}} and {{?I-D.reddy-uta-pqc-app}}.