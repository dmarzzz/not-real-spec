# Scheduled Access Token (SAT)

## Overview

The Scheduled Access Token (SAT) is an object-capability token that embeds delegation rules defining when, how, and with what permissions a delegatee can access user data or execute computations. SATs extend traditional capability tokens with time-based constraints, enabling automated scheduled operations while maintaining security and control.

## Containers

### ScheduledAccessToken

```
class ScheduledAccessToken(Container):
    issuer: WebID
    delegatee: ExecutionAddress
    capability_uri: URI
    schedule_rrule: String
    expiry: Timestamp
    signature: Bytes
```

### CapabilityInvocation

```
class CapabilityInvocation(Container):
    token_id: UUID
    invocation_id: UUID
    invocation_time: Timestamp
    invocation_proof: Bytes
    invoke_params: JSON
```

### TokenRevocation

```
class TokenRevocation(Container):
    token_id: UUID
    revocation_reason: String
    revocation_time: Timestamp
    replacement_token_id: Optional[UUID]
    revocation_signature: Bytes
```

## SAT Schedule Format

SATs use RFC 5545 (iCalendar) RRULE format to express schedules, providing a standardized, flexible, and expressive way to define recurring patterns:

```
RRULE:FREQ=DAILY;INTERVAL=1;BYHOUR=3;BYMINUTE=0
```

### Common Schedule Patterns

| Description | RRULE Format |
|-------------|--------------|
| Daily at 3 AM | `RRULE:FREQ=DAILY;BYHOUR=3;BYMINUTE=0` |
| Weekly on Monday | `RRULE:FREQ=WEEKLY;BYDAY=MO` |
| Monthly on the 1st | `RRULE:FREQ=MONTHLY;BYMONTHDAY=1` |
| Weekdays only | `RRULE:FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR` |
| Every 3 hours | `RRULE:FREQ=HOURLY;INTERVAL=3` |

## APIs

### Token Issuance API

```
POST /api/v1/tokens/issue
```

Issues a new Scheduled Access Token.

**Request Body:**
```json
{
  "delegatee": "execution-address",
  "capability": {
    "uri": "capability-uri",
    "permissions": ["read", "compute"],
    "resource_patterns": ["resource-uri-pattern"]
  },
  "schedule": "RRULE:FREQ=DAILY;BYHOUR=3",
  "expiry": "iso-datetime",
  "restrictions": {
    "ip_ranges": ["cidr-range"],
    "max_invocations": 100
  }
}
```

**Response:**
```json
{
  "token_id": "uuid",
  "sat_token": "serialized-token",
  "token_fingerprint": "token-hash",
  "issued_at": "iso-datetime",
  "first_invocation": "estimated-first-invocation-time"
}
```

### Token Verification API

```
POST /api/v1/tokens/verify
```

Verifies a Scheduled Access Token's validity.

**Request Body:**
```json
{
  "sat_token": "serialized-token",
  "verification_time": "iso-datetime-or-now"
}
```

**Response:**
```json
{
  "valid": true|false,
  "token_id": "uuid",
  "issuer": "issuer-webid",
  "next_valid_window": {
    "start": "iso-datetime",
    "end": "iso-datetime"
  },
  "remaining_validity": "iso-duration",
  "verification_result": "detailed-verification-message"
}
```

### Token Revocation API

```
POST /api/v1/tokens/revoke
```

Revokes a previously issued Scheduled Access Token.

**Request Body:**
```json
{
  "token_id": "uuid",
  "revocation_reason": "user-initiated|security-concern|replacement",
  "replacement_token_id": "replacement-uuid-if-applicable"
}
```

**Response:**
```json
{
  "status": "revoked",
  "revocation_time": "iso-datetime",
  "propagation_estimate": "iso-duration-until-fully-effective"
}
```

### Token Invocation History API

```
GET /api/v1/tokens/{token_id}/history
```

Retrieves the invocation history for a token.

**Response:**
```json
{
  "token_id": "uuid",
  "invocations": [
    {
      "invocation_id": "uuid1",
      "invocation_time": "iso-datetime1",
      "status": "successful|failed|denied",
      "invoker": "invoker-address"
    },
    {
      "invocation_id": "uuid2",
      "invocation_time": "iso-datetime2",
      "status": "successful|failed|denied",
      "invoker": "invoker-address"
    }
  ],
  "total_invocations": 42,
  "next_scheduled": "next-scheduled-invocation-time"
}
```

## Security Constraints

1. **Maximum Validity**: SATs cannot be issued with validity exceeding `MAX_SAT_DURATION` (recommended: 365 days).
2. **Cryptographic Requirements**: SATs must be signed using EdDSA (Ed25519) or ECDSA (secp256k1).
3. **Replay Protection**: Each invocation must include a unique nonce to prevent replay attacks.
4. **Revocation Checking**: Systems must verify against a revocation list before accepting a token.
5. **Time Synchronization**: Systems must use secure time synchronization to enforce schedule constraints.

## Implementation Guidelines

### Issuing SATs

1. Generate a cryptographically secure random token ID
2. Validate the schedule RRULE for correctness and security
3. Apply appropriate constraints based on the delegatee's identity and permissions
4. Sign the token using the issuer's private key
5. Record the token issuance in an append-only log

### Validating SATs

1. Verify the token's signature against the issuer's public key
2. Check the token against revocation lists
3. Verify that the current time falls within the schedule's allowed windows
4. Validate that any additional constraints (IP range, etc.) are satisfied
5. Generate and store a proof of validation

### Revoking SATs

1. Create a signed revocation certificate
2. Distribute the revocation to all potential verifiers
3. Update revocation lists with minimal latency
4. If replacing a token, issue the replacement before revoking the original

## Edge Cases

1. **Time Zone Changes**: Schedule interpretation must handle time zone changes correctly.
2. **Leap Years/Days**: Schedule calculations must account for calendar irregularities.
3. **Clock Skew**: Implementations should allow a small tolerance window (±30 seconds recommended) to accommodate minor clock differences.
4. **Token Duplication**: Systems should detect and prevent the use of the same token from multiple sources simultaneously.

## Future Considerations

1. **Delegation Chains**: Support for chains of delegation where delegatees can further delegate subsets of their capabilities.
2. **Dynamic Schedules**: Adaptation of schedules based on system conditions or user behavior patterns.
3. **Federated Verification**: Distributed verification across multiple trust domains. 