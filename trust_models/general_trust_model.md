# ZKP Self-Attestations Trust Model

## Overview

This document presents a generalized trust model for solutions that enable self-attestation using zero-knowledge proofs (ZKPs). In such solutions, a client-side device (user/prover) uses a digitally signed credential (issued by a trusted authority) to generate a ZKP that attests to selected identity attributes -- or functions thereof (e.g., proving age is over 18, or that a computed sum exceeds a threshold) -- without revealing the complete underlying data. 

We call this a "generalized trust model" because it describes the common characteristics, requirements, and assumptions shared by a family of ZKP-based self-attestation protocols. In these protocols, the process by which the trusted authority (the "issuer") signs and issues credentials occurs outside the protocol boundary, yet the protocol is informed by the issuance process to implement a freshness guarantee within the framework of this model. (For example, Anon Aadhaar uses a dedicated endpoint to provide the most recent data.)

This document does not aim to provide a generalized threat model, but highlights key considerations of ZKP self-attestations (for future analysis), while anchoring in standards for generalized threat modeling.  

**External Standards Alignment**:  
- Compatibility with W3C Verifiable Credentials (VC) data model.  
- Referencing the [W3C Decentralized Identities Threat Model](https://github.com/w3c-cg/threat-modeling/blob/main/models/decentralized-identities.md) for foundational adversarial scenarios.  

**Out of Scope**  
- Credential issuer (identity authority), as outlined in Section 1.
- Generalized Threat Modeling
   - Broad adversarial behaviors (e.g., Sybil attacks, social engineering) are addressed in the [W3C Threat Model](https://github.com/w3c-cg/threat-modeling/blob/main/models/decentralized-identities.md).  
   - Physical attack vectors (e.g., NFC skimming) are deferred to hardware-specific specifications.  
- Implementation-Specific Details  
   - ZKP circuit designs (e.g., Circom vs. Halo2).  
   - Cryptographic primitives (e.g., RSA vs. BBS+).  
- Regulatory Compliance  
   - GDPR/CCPA alignment is delegated to application-layer policies

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
- **Credential Expiration and Revocation:** The issuer is responsible for revoking issued credentials, ensuring discovery of revocation lists, and ensuring availability of credential expiration metadata. If the user's credential is revoked or expired, the protocol will not generate a proof. The data freshness window provides an upper bound for usage of previously-issued ZKP self-attestations.
- **Proof Invalidation:** Any proofs that are stale (i.e., those with freshness data outside the allowed window) or replayed (as detected by the nonce) are rejected by the verifier.

## 5. Use Cases
- Authentication: Privacy-preserving login from a trusted credential.
- Sybil Resistance: Proves a user is unique without revealing identity.
- Proof of Age Range: Verifies “over 18” (etc.) without sharing full DOB.
- Proof of Country: Shows region compliance with minimal data.
- Wallet Infrastructure: Validate transactions and/or recover your wallet with ID proof.

## 6. Detailed Examples

### Anon Aadhaar: Selective Disclosure Atop Government-Issued IDs  

#### Workflow  

**Inputs**:
- `nullifierSeed` from Verifier (for application scoping)
- UIDAI-signed QR code (e-Aadhaar PDF/mAadhaar app) containing user identity data
- UIDAI Public Key

**Proof Flow**:
1. User scans QR code
2. Prover verifies and generates proof
    - Extracts signed data bytes and RSA signature
    - Verifies signature on signed data and authenticates signing key against UIDAI's RSA public key from [governance endpoint](https://uidai.gov.in/publickey)
    - Generates `nullifier`, a function of `nullifierSeed`, photo bytes, and identity data fields (to prevent reuse).
    - Converts timestamp on signed data to UNIX UTC to include in output
   - Optionally reveals age >18, gender, state, or pincode.  
3. Proof Submission 
   - Submits proof, nullifier, timestamp, and public inputs to verifier.
  
![Untitled diagram-2025-02-25-000302](https://github.com/user-attachments/assets/c904fa60-c683-4e30-9af3-88aa71df86a3)

**Details on Role of Timestamp and Nullifiers**
Anon Aadhaar mitigates specific threats as follows:

1. Proof Freshness: Timestamp from Aadhaar QR Code
   - The UIDAI-signed timestamp in the QR code ensures the Aadhaar data is recent (e.g., ≤30 days old).  
   - Verifiers enforce validity windows (e.g., reject proofs with timestamps older than 30 days).  
2. Prevent Proof Relay and Cross-Verifier Correlation: Nullifier and Nullifier Seed
   - Verifier-specific `nullifierSeed` ensures unique `nullifier = hash(nullifierSeed || photoBytes)`.  
       - Reused proofs are detected/rejected by checking nullifier registry  
   -  Unique `nullifierSeed` per verifier ensures distinct nullifiers for the same user across apps; prevents tracking users across services (e.g., healthcare vs. finance apps).  
 
### OpenPassport: NFC-Based Identity Verification Without Live Issuer Endpoints
OpenPassport verifies electronic passports’ NFC chip data. Unlike Aadhaar’s UIDAI endpoint, passports lack live issuer APIs, necessitating alternative freshness mechanisms.

#### Workflow
- User scans passport’s NFC chip, extracting signed datagroups (name, DOB, nationality) and the issuing authority’s RSA public key.
- The ZKP circuit:
    - Checks passport signature validity using the country’s public key (from ICAO’s registry).
    - Selectively disclosures user-selected attributes (e.g., nationality, age)
    - Nullifier: Generated from immutable fields (DOB, passport number) to link proofs across sessions:
     - User Nullifier: Session-specific hash (e.g., random nonce) for one-time use11.

**Freshness Enforcement**:
- Passport Issuance Date: Circuits reject proofs from passports issued >10 years ago.
- On-Chain Revocation: Integrates with Ethereum’s ERC-948 registries to check revocation status

**Threat Mitigation**:
- ZKP verifies RSA signatures against ICAO’s public key registry; revoked keys are excluded via governance votes.
- Verifier-specific salts (e.g., per-verifier nonces) prevent correlation across verifiers
- On-chain revocation checks and issuance-date filters compensate for lack of live issuer endpoints.

## 7. Implementations

- Privado ID / Billions Network - https://billions.network/
- Rarimo - https://rarimo.com/
- Self Protocol - https://self.xyz/
- ZkPassport - https://zkpassport.id/

## Appendix: ZKP Self-Attestation Threat Model Considerations
The threat model for these protocols are similar to that of [W3C Decentralized Identities Threat Model](https://github.com/w3c-cg/threat-modeling/blob/main/models/decentralized-identities.md) -- a general threat model for decentralized identity architectures. ZKP Self-Attestation implementations should consider:

- Linkability (c.f. LINDDUN Threats)
    - Persistent Pseudonymity if implemented off-chain with verifier-provided `nullifierSeed`
    - On-chain implementations should consider impact. Mitigations may include anonymity sets via solutions like Semaphore
- Colluding user threats (c.f. "A malicious Holder who wants to get what he is not entitled to from the verifier.")
    - Because these proofs are generated outside the usual 3-party decentralized identity model, mechanisms for ensuring holder binding may differ with selectively disclosed attributes.
    - Mitigations include signed data freshness, attribute disclosure, biometrics, and scarcity. 

