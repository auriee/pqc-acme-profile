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

--- abstract
ACME TLS {{RFC8737}} leverages TLS 1.2 that employs the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for domain control validation. However, TLS 1.2 it is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in ACME.

This draft aims to enhance the security of ACME TLS protocol by defining TLS 1.3 PQC profile to make it quantum-safe {{RFC8737}}.

--- middle


--- back