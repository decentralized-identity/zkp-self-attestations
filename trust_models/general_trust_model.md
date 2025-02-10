# ZKP Self-Attestations Trust Model

## Overview

This document presents a generalized trust model for solutions that enable self-attestation using zero-knowledge proofs (ZKPs). In such solutions, a client-side device (user/prover) uses a digitally signed credential (issued by a trusted authority) to generate a ZKP that attests to selected identity attributes -- or functions thereof (e.g., proving age is over 18, or that a computed sum exceeds a threshold) -- without revealing the complete underlying data. 

We call this a "generalized trust model" because it describes the common characteristics, requirements, and assumptions shared by a family of ZKP-based self-attestation protocols. In these protocols, the process by which the trusted authority (the "issuer") signs and issues credentials occurs outside the protocol boundary, yet the protocol is informed by the issuance process to implement a freshness guarantee within the framework of this model. (For example, Anon Aadhaar uses a dedicated endpoint to provide the most recent data.)

## 1. Participants and Boundaries

### 1.1 Credential Issuer (Identity Authority) – Outside Protocol Boundary
- **Role:** The issuer digitally signs and issues credentials that attest to an individual's identity.
- **Trust Basis:** Relies on secure key management (e.g., RSA or equivalent), reliable public key distribution (via trusted registries or PKI), proper key rotation, and robust digital signing methods.
- **Assumption:** The underlying infrastructure ensures the issuer's signatures are genuine and that the public key distribution remains uncompromised.

*Note:* Although the signing and issuance process occurs outside the protocol boundary, the design of the protocol is informed by these processes.

### 1.2 User/Prover (Client-Side Device) – Within Protocol Boundary
- **Role:** The user/prover "owns" the received credential and generates zero-knowledge proofs to attest to selected identity attributes or functions of those attributes (e.g., predicates or computed properties) without revealing the complete underlying data. In W3C Verifiable Credentials terminology, this entity acts as both the subject and the holder.
- **Trust Basis:** Relies on the security of the client device to protect secret inputs (such as private keys, biometric data, etc.) and on a correct, uncompromised implementation of the proof-generation process.
- **Assumption:** The client device environment is assumed to be secure enough to prevent data leakage, tampering, or side-channel attacks.

### 1.3 Verifier – Within Protocol Boundary
- **Role:** The verifier validates the zero-knowledge proofs to confirm both the cryptographic integrity and the freshness of the signed data.
- **Trust Basis:** Depends on the proper implementation of verification algorithms that check the mathematical soundness of the ZKP and ensure signed data freshness—typically by verifying an embedded timestamp (or equivalent freshness indicator) is within an acceptable time window.
- **Assumption:** It is assumed that the verifier's system faithfully adheres to the protocol rules and processes the proofs securely, without requiring access to the raw credential data.

## 2. Trust Boundaries and Mechanisms

### 2.1 Issuer–User Boundary
- **Mechanism:** Trust is established when the user accepts a credential digitally signed by a trusted issuer. The user’s device relies on secure public key distribution and the integrity of the digital signature to ensure the credential is valid.

### 2.2 User–Verifier Boundary
- **Mechanism:** The zero-knowledge proof mechanism enables the user to generate proofs that reveal only selected identity attributes or functions (e.g., computed predicates) without exposing the full underlying data. The verifier then checks the cryptographic validity of these proofs and verifies signed data freshness (using embedded timestamps or equivalent indicators) to confirm that the proof is both current and valid.

### 2.3 Mathematical and Implementation Trust
- **Mechanism:** The overall security of the system is founded on established cryptographic assumptions (such as the hardness of certain mathematical problems) and the secure implementation of key management, proof-generation circuits, and verification algorithms.


## 3. Trust Assumptions

- **Issuer Assumptions:**  
  * The issuer securely manages cryptographic keys, performs regular key rotations, and distributes its public keys through trusted channels.  
  * The digital signing process is performed correctly, ensuring the signature authentically represents the credential.

- **User/Prover Assumptions:**  
  * The client device is secure against malware and side-channel attacks that might compromise secret inputs.  
  * The implementation of zero-knowledge proof generation is correct and resistant to forgery.

- **Verifier Assumptions:**  
  * The verifier correctly implements all protocol verification steps, including cryptographic proof validation and checks for signed data freshness.  
  * The verifier’s system and underlying libraries are secure and have not been compromised.

## 4. Trust Lifecycle

### 4.1 Establishment
- **Credential Issuance:** A trusted issuer digitally signs identity data to produce a credential. Although this process is external to the protocol boundary, its security informs the protocol design (e.g., what assurances the ZKP must carry).  
- **User Device Installation:** The user installs a protocol-conformant application capable of generating zero-knowledge proofs from issued credentials.

### 4.2 Maintenance
- **Signed Data Freshness:** The protocol integrates a mechanism -- such as an embedded timestamp, nonce, or a freshness value obtained from a trusted endpoint -- to ensure that each zero-knowledge proof reflects a recent authentication event. For instance, in an Anon Aadhaar implementation, a dedicated freshness endpoint provides up-to-date data that must be included in the ZKP. This mechanism prevents replay attacks.
- **Key and Credential Management:** Ongoing procedures (like regular key rotations and credential revocations) ensure that any compromised keys or outdated credentials are promptly invalidated.

### 4.3 Termination
- **Credential Revocation:** If an issuer's key is compromised or a user’s device is deemed insecure, the corresponding credential is revoked. The data freshness window provides an upper bound for usage of a ZKP self-attestation post-revocation.
- **Proof Invalidation:** Any proofs that are stale (i.e., those with freshness data outside the allowed window) or replayed (as detected by the nonce) are rejected by the verifier.
