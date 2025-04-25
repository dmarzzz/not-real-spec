# Connectors

## Overview

Connectors are stateless adapters that facilitate interaction between the CDCD system and external services, such as social media platforms, cloud storage providers, and third-party APIs. They provide a standardized interface for secure data exchange, while maintaining the privacy and security guarantees of the core system.

## Containers

### ConnectorDefinition

```
class ConnectorDefinition(Container):
    connector_id: String
    service_type: String  # e.g., "social_media", "storage", "messaging"
    capabilities: List[String]
    authentication_methods: List[String]
    data_schema: JSONSchema
    rate_limits: RateLimitSpec
    versioning: VersionInfo
```

### ConnectorCredential

```
class ConnectorCredential(Container):
    credential_id: UUID
    connector_id: String
    user_id: WebID
    service_identifier: URI
    credential_blob: Bytes  # Encrypted credential data
    scope: List[String]
    issuance_time: Timestamp
    expiry: Timestamp
    refresh_metadata: Optional[JSON]
```

### ConnectorRequest

```
class ConnectorRequest(Container):
    request_id: UUID
    connector_id: String
    credential_reference: UUID
    operation: String
    parameters: JSON
    request_time: Timestamp
    sat_reference: Optional[UUID]
    request_signature: Bytes
```

### ConnectorResponse

```
class ConnectorResponse(Container):
    request_id: UUID
    status: String  # "success", "error", "rate_limited"
    result_data: Bytes  # Encrypted response data
    service_metadata: JSON
    timestamp: Timestamp
    response_signature: Bytes
```

## APIs

### Connector Registry API

```
GET /api/v1/connectors
```

Lists available connectors and their capabilities.

**Response:**
```json
{
  "connectors": [
    {
      "connector_id": "twitter",
      "service_type": "social_media",
      "capabilities": ["post", "read", "search"],
      "authentication_methods": ["oauth2"],
      "version": "1.2.0"
    },
    {
      "connector_id": "gdrive",
      "service_type": "storage",
      "capabilities": ["upload", "download", "share"],
      "authentication_methods": ["oauth2"],
      "version": "2.0.1"
    }
  ]
}
```

### Connector Authentication API

```
POST /api/v1/connectors/{connector_id}/auth
```

Initiates or completes authentication flow for a connector.

**Request Body (Initiation):**
```json
{
  "auth_method": "oauth2",
  "redirect_uri": "https://app.example.com/callback",
  "scope": ["read", "write"]
}
```

**Response (Initiation):**
```json
{
  "auth_url": "https://service.example.com/oauth2/authorize?...",
  "state": "secure-state-token",
  "expiry": "iso-datetime"
}
```

**Request Body (Completion):**
```json
{
  "auth_method": "oauth2",
  "state": "secure-state-token",
  "code": "authorization-code",
  "encryption_key_reference": "key-reference-for-credential-encryption"
}
```

**Response (Completion):**
```json
{
  "credential_id": "uuid",
  "status": "active",
  "expiry": "iso-datetime",
  "service_user_id": "service-specific-user-identifier",
  "capabilities": ["read", "write"]
}
```

### Connector Operation API

```
POST /api/v1/connectors/{connector_id}/operate
```

Performs an operation using a connector.

**Request Body:**
```json
{
  "connector_request": "serialized-connector-request",
  "sat_token": "optional-serialized-scheduled-access-token",
  "signature": "signature-over-request"
}
```

**Response:**
```json
{
  "connector_response": "serialized-connector-response",
  "signature": "signature-over-response"
}
```

### Connector Credential Management API

```
GET /api/v1/connector-credentials
```

Lists all connector credentials for the authenticated user.

**Response:**
```json
{
  "credentials": [
    {
      "credential_id": "uuid1",
      "connector_id": "twitter",
      "service_identifier": "https://twitter.com/username",
      "scopes": ["read", "write"],
      "issuance_time": "iso-datetime",
      "expiry": "iso-datetime",
      "status": "active"
    },
    {
      "credential_id": "uuid2",
      "connector_id": "gdrive",
      "service_identifier": "https://drive.google.com/user/123",
      "scopes": ["read"],
      "issuance_time": "iso-datetime",
      "expiry": "iso-datetime",
      "status": "expired"
    }
  ]
}
```

```
DELETE /api/v1/connector-credentials/{credential_id}
```

Revokes and deletes a connector credential.

**Response:**
```json
{
  "status": "revoked",
  "revocation_time": "iso-datetime",
  "service_revocation_status": "confirmed|pending"
}
```

## Supported Connector Types

### Social Media Connectors
- **Twitter/X**: Post, read timeline, search
- **Instagram**: Post images, retrieve feed
- **LinkedIn**: Post updates, access professional network
- **TikTok**: Video upload, feed access

### Storage Connectors
- **Google Drive**: File upload/download
- **Dropbox**: File synchronization
- **OneDrive**: Microsoft ecosystem integration
- **IPFS**: Decentralized storage

### Communication Connectors
- **Email**: SMTP/IMAP integration
- **Matrix**: Secure messaging
- **Discord**: Community messaging
- **Slack**: Workspace communication

### Productivity Connectors
- **Google Calendar**: Event scheduling
- **Notion**: Knowledge management
- **GitHub**: Development workflow
- **Jira**: Project management

## Security Requirements

1. **Zero Knowledge Design**: Connectors must not have access to unencrypted credentials.
2. **Minimal Scope**: Request only the minimal permissions needed for operation.
3. **Token Security**: OAuth tokens and other credentials must be encrypted at rest.
4. **Revocation Handling**: Immediate propagation of credential revocation.
5. **Rate Limiting**: Enforcement of service-specific and global rate limits.

## Implementation Guidelines

### Authentication Flows

1. **OAuth 2.0**: Standard flow with PKCE for web and mobile
2. **API Keys**: Encrypted storage with key rotation schedules
3. **App Passwords**: Generation and management for legacy services
4. **Zero-knowledge proofs**: For applicable privacy-preserving services

### Credential Management

1. **Key Rotation**: Automatic rotation of long-lived credentials
2. **Refresh Handling**: Transparent handling of token refreshes
3. **Revocation**: Immediate deletion upon user request
4. **Monitoring**: Detection of credential compromise

### Error Handling

1. **Retries**: Exponential backoff for temporary failures
2. **Circuit Breaking**: Protection against cascading failures
3. **Fallbacks**: Alternative operation modes when services degrade
4. **Status Reporting**: Clear error reporting with remediation steps

## Operational Considerations

- **Service Dependencies**: Monitoring of external service health
- **Version Compatibility**: Handling API changes in external services
- **Compliance**: Region-specific data handling requirements
- **Service Migration**: Support for changing between equivalent services

## Future Considerations

1. **Federated Authentication**: Integration with decentralized identity systems
2. **First-Party Connectors**: Service provider-built connectors with enhanced capabilities
3. **Multi-Service Operations**: Atomic operations spanning multiple connectors
4. **E2EE Service Integration**: Direct integration with end-to-end encrypted platforms 