# Confidential Runtime (CR)

## Overview

The Confidential Runtime (CR) is a Trusted Execution Environment (TEE)-enabled execution environment responsible for securely executing delegated computations over private user data. It provides isolation, attestation, and secure execution guarantees within a confidential computing environment.

## Containers

### ComputeRequest

```
class ComputeRequest(Container):
    request_id: UUID
    sat_token: ScheduledAccessToken
    parameters: JSON
    timestamp: Timestamp
    request_signature: Bytes
```

### ComputeResult

```
class ComputeResult(Container):
    request_id: UUID
    output: Bytes  # Encrypted for the requestor
    execution_receipt: ReceiptProof
    timestamp: Timestamp
    result_signature: Bytes
```

### ReceiptProof

```
class ReceiptProof(Container):
    execution_id: UUID
    data_fingerprints: List[Bytes]  # Hashes of data accessed
    compute_fingerprint: Bytes  # Hash of computation performed
    tee_attestation: Bytes
    timestamp_range: (Timestamp, Timestamp)
```

## APIs

### Attestation API

```
GET /api/v1/attestation
```

Returns the current TEE attestation report for verification.

**Response:**
```json
{
  "attestation_report": "base64-encoded-report",
  "tee_type": "SGX|TDX|SEV-SNP",
  "runtime_id": "uuid",
  "timestamp": "iso-datetime"
}
```

### Compute Delegation API

```
POST /api/v1/compute/delegate
```

Delegates a computation to be executed within the Confidential Runtime.

**Request Body:**
```json
{
  "compute_request": "serialized-compute-request",
  "sat_token": "serialized-scheduled-access-token",
  "signature": "signature-over-request" 
}
```

**Response:**
```json
{
  "request_id": "uuid",
  "status": "accepted|rejected",
  "estimated_execution_time": "iso-duration",
  "error": "error-message-if-rejected"
}
```

### Compute Status API

```
GET /api/v1/compute/status/{request_id}
```

Checks the status of a previously delegated computation.

**Response:**
```json
{
  "request_id": "uuid",
  "status": "queued|executing|completed|failed",
  "start_time": "iso-datetime-or-null",
  "completion_time": "iso-datetime-or-null",
  "result_available": true|false
}
```

### Compute Result API

```
GET /api/v1/compute/result/{request_id}
```

Retrieves the result of a completed computation.

**Response:**
```json
{
  "compute_result": "serialized-compute-result",
  "signature": "signature-over-result"
}
```

## Security Guarantees

The Confidential Runtime provides the following security guarantees:

1. **Execution Isolation**: Computations are isolated from the host OS and other workloads running on the same physical hardware.
2. **Data Confidentiality**: User data is decrypted only within the TEE boundary and never exposed in plaintext outside it.
3. **Code Integrity**: The code executed inside the TEE is attested and verified before execution.
4. **Verifiable Attestation**: Remote parties can verify the TEE's identity and state via attestation.
5. **Auditable Execution**: Every computation generates a cryptographic receipt that can be verified by authorized parties.

## Operational Requirements

- **Memory Requirements**: Minimum 8GB EPC (Enclave Page Cache) for SGX-based runtimes.
- **TEE Support**: Must run on hardware supporting TEE (SGX, TDX, SEV-SNP).
- **Network Security**: All API endpoints must be accessible only via TLS 1.3 or higher.
- **Attestation Freshness**: Attestation reports must be refreshed at least every 24 hours.

## Failure Modes

1. **TEE Compromise**: In the unlikely event of a TEE compromise, CR instances must implement immediate shutdown and key revocation.
2. **Memory Exhaustion**: When memory limits are approached, computations should be gracefully rejected rather than risking data exposure.
3. **Attestation Failure**: If attestation verification fails, the CR must enter a quarantine mode until administrator intervention.

## Implementation Considerations

- **Hardware Selection**: Intel SGX and AMD SEV-SNP are recommended for production deployments.
- **TEE SDK**: Use the SGX SDK, Open Enclave SDK, or TDX SDK depending on the hardware platform.
- **Remote Attestation**: Implement EPID (for SGX) or DCAP-based remote attestation for verification.
- **Scaling**: Support for horizontal scaling via a cluster of CR instances, each with its own attestation.

## Future Considerations

- **GPU TEE Support**: As GPU TEE technologies mature, support for accelerated ML workloads within TEEs will be added.
- **Multi-Party Computation**: Extensions to support secure multi-party computation (MPC) across multiple CRs.
- **Zero-Knowledge Proofs**: Integration with ZKP systems to provide additional verification of computation correctness. 