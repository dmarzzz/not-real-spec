# Capability Delegation Layer

## Overview

The Capability Delegation Layer enables secure, controlled delegation of access and computation rights over personal data using object capability (OCap) principles. It provides the mechanisms for precise, time-bound, and revocable permissions through Scheduled Access Tokens (SATs) and other capability-based mechanisms.

## Components

The Capability Delegation Layer consists of the following primary components:

1. **[Scheduled Access Token (SAT)](scheduled_access_token.md)** - Time-based capability tokens
2. **Capability Registry** - Maintains the registry of issued capabilities
3. **Delegation Manager** - Handles creation and validation of delegation chains
4. **Revocation Service** - Manages revocation of capabilities

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                 Capability Delegation Layer                  │
├──────────────────┬─────────────────────┬────────────────────┤
│                  │                     │                    │
│  Scheduled       │    Capability       │   Delegation       │
│  Access Tokens   │     Registry        │    Manager         │
│                  │                     │                    │
├──────────────────┼─────────────────────┼────────────────────┤
│                                                             │
│                     Revocation Service                      │
│                                                             │
└──────────────────────────────────────────────────────────────┘
```

## Capability Model

The Capability Delegation Layer implements a pure object-capability (OCap) model with the following characteristics:

1. **Designation**: Capabilities designate specific resources
2. **Authority**: Capabilities convey specific rights over resources
3. **Attenuability**: Capabilities can be restricted when delegated
4. **Dynamism**: Rights can be modified over time through delegation
5. **Composability**: Capabilities can be combined for complex permissions

## Delegation Patterns

The system supports several delegation patterns:

### Direct Delegation

A user directly delegates capabilities to a service or agent:

```
User ---(delegates)---> Service/Agent
```

### Chain Delegation

Capabilities can be delegated through a chain of entities:

```
User ---(delegates)---> Service A ---(delegates subset)---> Service B
```

### Scheduled Delegation

Delegation with time-based constraints using Scheduled Access Tokens:

```
User ---(delegates with schedule)---> Service
```

### Threshold Delegation

Requires multiple approvals for sensitive operations:

```
User ---(delegates with 2/3 threshold)---> [Service A, Service B, Service C]
```

## Capability Token Formats

The Capability Delegation Layer supports multiple capability token formats:

### 1. Scheduled Access Token (SAT)

Primary token format for time-based delegations:

```
class ScheduledAccessToken(Container):
    issuer: WebID
    delegatee: ExecutionAddress
    capability_uri: URI
    schedule_rrule: String
    expiry: Timestamp
    signature: Bytes
```

### 2. zcap-LD

W3C-aligned Linked Data Capabilities:

```json
{
  "@context": "https://w3id.org/security/v2",
  "id": "urn:uuid:e8fc7810-9524-11ea-bb37-0242ac130002",
  "invoker": "did:example:alice",
  "parentCapability": "https://pod.example/resource",
  "allowedAction": ["read", "write"],
  "caveat": [{
    "type": "TimeBoundCaveat",
    "validUntil": "2025-01-01T00:00:00Z"
  }],
  "proof": {
    "type": "Ed25519Signature2020",
    "verificationMethod": "did:example:alice#key1",
    "created": "2023-02-15T21:29:01Z",
    "proofPurpose": "capabilityDelegation",
    "proofValue": "z4oey5q..."
  }
}
```

### 3. Biscuit Tokens

Attenuation-friendly capability tokens:

```
authority {
  resource("https://pod.example.com/user123/data/");
  operation("READ");
}

attenuation by user {
  operation("READ");
  schedule("RRULE:FREQ=DAILY;BYHOUR=3");
  delegate_to("external-service");
}
```

## APIs

### Capability Issuance API

```
POST /api/v1/capabilities/issue
```

Issues a new capability token.

**Request Body:**
```json
{
  "capability_type": "SAT|zcap-LD|biscuit",
  "resource": "resource-uri",
  "permissions": ["read", "compute"],
  "delegatee": "delegatee-identifier",
  "constraints": {
    "schedule": "RRULE:FREQ=DAILY;BYHOUR=3",
    "expiry": "iso-datetime",
    "ip_ranges": ["cidr-range"]
  }
}
```

**Response:**
```json
{
  "capability_id": "uuid",
  "capability_token": "serialized-token",
  "token_fingerprint": "token-hash",
  "issued_at": "iso-datetime",
  "valid_until": "iso-datetime"
}
```

### Capability Verification API

```
POST /api/v1/capabilities/verify
```

Verifies a capability token's validity.

**Request Body:**
```json
{
  "capability_token": "serialized-token",
  "resource": "resource-uri",
  "action": "requested-action",
  "context": {
    "current_time": "iso-datetime",
    "requestor_ip": "ip-address"
  }
}
```

**Response:**
```json
{
  "valid": true|false,
  "capability_id": "uuid",
  "issuer": "issuer-identifier",
  "delegatee": "delegatee-identifier",
  "permissions_granted": ["read"],
  "validation_result": "detailed-validation-message",
  "expiry": "iso-datetime"
}
```

### Capability Revocation API

```
POST /api/v1/capabilities/revoke
```

Revokes a previously issued capability.

**Request Body:**
```json
{
  "capability_id": "uuid",
  "revocation_reason": "user-initiated|security-concern|replacement",
  "replacement_capability_id": "replacement-uuid-if-applicable"
}
```

**Response:**
```json
{
  "status": "revoked",
  "revocation_time": "iso-datetime",
  "propagation_status": "complete|in-progress",
  "propagation_estimate": "iso-duration-until-fully-effective"
}
```

### Capability Registry API

```
GET /api/v1/capabilities
```

Lists all capabilities issued by the authenticated user.

**Response:**
```json
{
  "capabilities": [
    {
      "capability_id": "uuid1",
      "capability_type": "SAT",
      "resource": "resource-uri-1",
      "delegatee": "delegatee-1",
      "permissions": ["read"],
      "issuance_time": "iso-datetime",
      "expiry": "iso-datetime",
      "status": "active"
    },
    {
      "capability_id": "uuid2",
      "capability_type": "zcap-LD",
      "resource": "resource-uri-2",
      "delegatee": "delegatee-2",
      "permissions": ["compute"],
      "issuance_time": "iso-datetime",
      "expiry": "iso-datetime",
      "status": "revoked"
    }
  ]
}
```

## Revocation Mechanisms

The Capability Delegation Layer supports multiple revocation mechanisms:

### 1. Direct Revocation

Immediate revocation through the Revocation Service:
- Capability ID is added to a revocation list
- All verifiers check against the revocation list
- Propagation delay is minimized through push notifications

### 2. Expiry-Based Revocation

Automatic revocation when capabilities expire:
- No explicit revocation action required
- Relies on synchronized time across the system
- Grace period can be configured for clock skew

### 3. Delegation Chain Revocation

Revocation of a capability in a delegation chain:
- Revokes the specified capability and all derived capabilities
- Propagates through the delegation graph
- Notification to affected delegatees

## Security Considerations

1. **Capability Confinement**: Capabilities should be bound to specific contexts
2. **Least Privilege**: Capabilities should convey minimal necessary permissions
3. **Secure Distribution**: Capability tokens must be transmitted securely
4. **Verification Performance**: Efficient verification with minimal latency
5. **Revocation Propagation**: Minimize the window between revocation and effectiveness

## Implementation Guidelines

### Issuing Capabilities

1. Generate a cryptographically secure capability identifier
2. Validate requested permissions against resource access policies
3. Apply constraints based on the delegatee and resource sensitivity
4. Sign the capability using the issuer's private key
5. Record issuance in the Capability Registry

### Verifying Capabilities

1. Validate the capability's cryptographic signature
2. Check against the revocation list
3. Verify that constraints are satisfied (time, network, etc.)
4. Validate the delegation chain if applicable
5. Authorize the requested action if verification passes

### Revoking Capabilities

1. Update the central revocation registry
2. Push revocation notifications to relevant verifiers
3. Update delegation chains to reflect revocation
4. Provide feedback on revocation propagation status

## Future Roadmap

1. **Federated Capability Verification**: Distributed verification across trust domains
2. **Privacy-Preserving Verification**: Zero-knowledge proofs for capability verification
3. **Programmable Capabilities**: Dynamic constraint evaluation
4. **Cross-Domain Delegation**: Seamless delegation across different administrative domains 