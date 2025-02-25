# Anon Aadhaar Trust Model, Draft

## 1. Introduction

This trust model defines trust relationships and boundaries for Anon Aadhaar, supplementing the [protocol specification](https://github.com/zkspecs/zkspecs/blob/main/specs/2/README.md). The model focuses on what must be trusted for the protocol's security properties to hold, rather than implementation details.

## 2. System Overview

### Participants and Boundaries

1. **UIDAI** (outside protocol boundary):
   - Root of trust via RSA key infrastructure
   - Issues up-to-date, signed Aadhaar identity data, encoded as [Aadhaar Secure QR codes](https://uidai.gov.in/en/ecosystem/authentication-devices-documents/qr-code-reader.html)
   - Controls key rotation and distribution
2. **User/Prover** (within protocol boundary):
   - Legitimate Aadhaar identity owner
   - Controls Aadhaar Secure QR code generation
   - Manages identity attribute disclosure
   - Must protect private inputs on local device
3. **Verifier** (within protocol boundary):
   - Validates proofs against protocol rules

### Trust Boundaries and Transitions

1. **UIDAI / User Boundary**
   - Trust established through authenticated endpoints
   - UIDAI delegates trust via RSA signatures
2. **User / Verifier Boundary**
   - Trust preserved through zero-knowledge proofs and protocol compliance
   - Protocol enforces selective disclosure controls; no raw Aadhaar data crosses boundary

## 4. Trust Assumptions

### Required

1. **UIDAI Trust**
   - Secure RSA key management
   - Accurate identity data signing
   - Reliable timestamp generation
2. **User Domain Trust**
   - TODO: assumptions about device/local environment preventing leakage of sensitive data
3. **Verifier Trust**
   - Correct implementation of protocol, including nullifiers and timestamps (TODO: what about source of timestamps relative to issuer?)

### Non-Assumptions

It is not assumed that Verifiers will:

- Preserve privacy beyond protocol requirements
- Keep nullifiers private from other applications/verifiers
- Protect against user tracking if nullifiers are reused

## 5. Trust Lifecycle

### Establishment

- UIDAI key distribution
- User authentication to endpoints
- Verifier protocol compliance

### Maintenance

- UIDAI Key Management
  - Regular key rotation procedures
  - Distribution of new public keys
  - Maintaining key validity periods
- Proof Freshness
  - Enforcement of timestamp validity windows, thereby helping prevent proof replay
  - Tracking of UIDAI key rotation events
- Nullifier Management
  - Tracking used nullifiers within applications
  - Enforcing uniqueness constraints
  - Managing action-specific nullifier

### Termination

1. Key Invalidation
   - Procedures for compromised UIDAI keys
   - Rejection of proofs using invalid keys
   - Migration to new / rotated keys
2. Proof Invalidation
   - Criteria for rejecting outdated proofs
   - Handling of deprecated protocol versions
   - Response to discovered vulnerabilities
3. Nullifier Updates
   - Handling of Aadhaar photo changes
   - Procedures for refreshing nullifiers
   - Management of invalid nullifier lists
  
## 6. Example Implementations

### Anon Aadhaar: Selective Disclosure Atop Government-Issued IDs  

#### Workflow  

**Inputs**:
- `nullifierSeed` from Verifier (for application scoping)
- UIDAI-signed QR code (e-Aadhaar PDF/mAadhaar app) containing user identity data

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
   - Verifier-specific `nullifierSeed` ensures unique `nullifier = SHA-256(nullifierSeed || photoBytes)`.  
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

## References

1. [Anon Aadhaar Specification](https://github.com/zkspecs/zkspecs/blob/main/specs/2/README.md)
2. [Anon Aadhaar - How does it work](https://documentation.anon-aadhaar.pse.dev/docs/how-does-it-work)
3. [Aadhaar Secure QR Code](https://uidai.gov.in/en/ecosystem/authentication-devices-documents/qr-code-reader.html)
