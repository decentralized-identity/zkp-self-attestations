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

## References

1. [Anon Aadhaar Specification](https://github.com/zkspecs/zkspecs/blob/main/specs/2/README.md)
2. [Aadhaar Secure QR Code](https://uidai.gov.in/en/ecosystem/authentication-devices-documents/qr-code-reader.html)
