# ZKP Self-Attestations Trust Model

## Overview

This document presents a generalized trust model for solutions that enable self-attestation using zero-knowledge proofs (ZKPs). In such solutions, a client-side device (user/prover) uses a digitally signed credential (issued by a trusted authority) to generate a ZKP that attests to selected identity attributes -- or functions thereof (e.g., proving age is over 18, or that a computed sum exceeds a threshold) -- without revealing the complete underlying data. 

We call this a "generalized trust model" because it describes the common characteristics, requirements, and assumptions shared by a family of ZKP-based self-attestation protocols. In these protocols, the process by which the trusted authority (the "issuer") signs and issues credentials occurs outside the protocol boundary, yet the protocol is informed by the issuance process to implement a freshness guarantee within the framework of this model. (For example, Anon Aadhaar uses a dedicated endpoint to provide the most recent data.)

## 1. Participants and Boundaries

![self-attestation-roles](https://github.com/user-attachments/assets/16932727-1965-4719-8a25-426d1711a869)

### 1.1 Credential Issuer (Identity Authority) – Outside Protocol Boundary
- **Role:** The issuer digitally signs and issues credentials that attest to an individual's identity.
- **Trust Basis:** Relies on secure key management (e.g., RSA or equivalent), reliable public key distribution (via trusted registries or PKI), proper key rotation, and robust digital signing methods. Also includes verifier-specific considerations, such as isser reputation or regulatory compliance.
- **Assumption:** The underlying issuing infrastructure ensures the issuer's signatures are genuine, that the public key distribution remains uncompromised, and that accurate records of issued and revoked credentials are kept.

*Note:* Although the signing and issuance process occurs outside the protocol boundary, the design of the protocol is informed by these processes.

### 1.2 User/Prover (Client-Side Device) – Within Protocol Boundary
- **Role:** The user/prover "owns" the received credential and generates zero-knowledge proofs to attest to selected identity attributes or functions of those attributes (e.g., predicates or computed properties) without revealing the complete underlying data. In W3C Verifiable Credentials terminology, this entity acts as both the subject and the holder.
- **Trust Basis:** Relies on the security of the client device to protect secret inputs (such as private keys, biometric data, etc.) and on a correct, uncompromised implementation of the proof-generation process.
- **Assumption:** The client device environment is assumed to be secure enough to prevent data leakage, tampering, or side-channel attacks. Correct implementation of proving.

### 1.3 Verifier – Within Protocol Boundary
- **Role:** The verifier validates the zero-knowledge proofs to confirm both the cryptographic integrity and the freshness of the signed data.
- **Trust Basis:** Depends on the proper implementation of verification algorithms that check the mathematical soundness of the ZKP and ensure signed data freshness—typically by verifying an embedded timestamp (or equivalent freshness indicator) is within an acceptable time window.
- **Assumption:** It is assumed that the verifier faithfully adheres to the protocol rules and processes the proofs securely, without requiring access to the raw credential data. The verifier has access to reliable timestamp data.

## 2. Trust Boundaries and Mechanisms

### 2.1 Issuer–User Boundary
- **Mechanism:** Trust is established when the user accepts a credential digitally signed by a trusted issuer. The user’s device relies on secure public key distribution and the integrity of the digital signature to ensure the credential is valid.

### 2.2 User–Verifier Boundary
- **Mechanism:** The zero-knowledge proof mechanism enables the user to generate proofs that reveal only selected identity attributes or functions (e.g., computed predicates) without exposing the full underlying data. The verifier then checks the cryptographic validity of these proofs and verifies signed data freshness (using embedded timestamps or equivalent indicators) to confirm that the proof is both current and valid.

### 2.3 Mathematical and Implementation Trust
- **Mechanism:** The overall security of the system is founded on established cryptographic assumptions (such as the hardness of certain mathematical problems) and the secure implementation of key management, proof-generation circuits, and verification algorithms.


## 3. Trust Assumptions and Mitigations

- **Issuer Assumptions (out of scope of protocol):**  
  * The issuer securely manages cryptographic keys, performs regular key rotations, and distributes its public keys through trusted channels.  
  * The digital signing process is performed correctly, ensuring the signature authentically represents the credential.
  * The issuer follows proper identity verification procedures before credential issuance
  * The issuer maintains accurate records of issued and revoked credentials.
  * Additional considerations, depending on Verifier requirements:
    - Established reputation and compliance with relevant regulatory frameworks
    - Adherence to periodic audits and certification requirements

- **User/Prover Assumptions:**  
  * The client device environment is secure against malware and side-channel attacks that might compromise, tamper with, or leak secret inputs.
      * Mitigation: TEE usage where available, secure enclaves, integrity checking
  * Time source used for freshness validation is reliable and secure
  * The proof generation process is correct, uncompromised, and resistant to forgery.
      * Mitigation: Open-source implementations, formal verification 

- **Verifier Assumptions:**  
  * The verifier correctly implements all protocol verification steps, including cryptographic proof validation and checks for signed data freshness.
  * Time source used for freshness validation is reliable and secure
  * The verifier’s system and underlying libraries are secure and have not been compromised.
      * Mitigation: Open-source implementations, formal verification  
 
The verifier may require addition verification methods (e.g., checking attribute disclosures against different sources), but this is outside the scope of the protocol. 

## 4. Trust Lifecycle

### 4.1 Establishment
- **Credential Issuance:** A trusted issuer authenticates the individual, digitally signs an identity credential, and securely delivers the signed credential to the user. Although this process is external to the protocol boundary, specifics of the implementation informs the protocol design.  
- **User Device Installation:** The user installs a protocol-conformant application capable of generating zero-knowledge proofs from issued credentials.

### 4.2 Maintenance
- **Signed Data Freshness:** The protocol integrates a mechanism -- such as an embedded timestamp, nonce, or a freshness value obtained from a trusted endpoint -- to ensure that each zero-knowledge proof reflects a recent authentication event. For instance, in an Anon Aadhaar implementation, a dedicated freshness endpoint provides up-to-date data that must be included in the ZKP.
- **Key and Credential Management:** Ongoing procedures (like regular key rotations and credential revocations) ensure that any compromised keys or outdated credentials are promptly invalidated.

### 4.3 Termination
- **Invalid Issuer Key:** If an issuer's key is compromised, expires, or is not consistent with the user's credential, the protocol will not generate a proof. The data freshness window provides an upper bound for usage of previously-generated ZKP self-attestations.
- **Credential Expiration and Revocation:** The issuer is responsible for revoking issued credentials, ensuring discovery of revocation lists, and ensuring availability of credential expiration metadata. If the user's credential is revoked or expired, the protocol will not generate a proof. The data freshness window provides an upper bound for usage of previously-issed ZKP self-attestations.
- **Proof Invalidation:** Any proofs that are stale (i.e., those with freshness data outside the allowed window) or replayed (as detected by the nonce) are rejected by the verifier.
