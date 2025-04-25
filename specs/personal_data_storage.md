# Personal Data Storage Layer (PDSL)

## Overview

The Personal Data Storage Layer (PDSL) provides secure, user-controlled storage for private data. It supports modular backend implementations, with Solid Pods as the reference implementation, while enabling alternative storage backends such as encrypted vector databases or object stores.

## Containers

### PersonalDataPayload

```
class PersonalDataPayload(Container):
    data_id: UUID
    encrypted_blob: Bytes
    metadata: JSON
    signature: Bytes
```

### StorageAccessGrant

```
class StorageAccessGrant(Container):
    grant_id: UUID
    issuer: WebID
    grantee: IdentityURI
    resource_patterns: List[URIPattern]
    permissions: List[PermissionMode]  # Read, Write, Append
    conditions: JSON
    expiry: Timestamp
    signature: Bytes
```

### DataIndexEntry

```
class DataIndexEntry(Container):
    data_id: UUID
    content_type: String
    content_hash: Bytes
    encryption_metadata: EncryptionMetadata
    semantic_tags: List[String]
    creation_time: Timestamp
    last_modified: Timestamp
    access_control_uri: URI
```

### EncryptionMetadata

```
class EncryptionMetadata(Container):
    algorithm: String  # e.g., "AES-256-GCM"
    key_id: UUID  # Reference to key in user's KMS
    iv: Bytes
    tag: Bytes
```

## APIs

### Data Storage API

```
PUT /api/v1/data/{data_id}
```

Stores encrypted data in the PDSL.

**Request Body:**
```json
{
  "payload": "serialized-personal-data-payload",
  "metadata": {
    "content_type": "mime-type",
    "semantic_tags": ["tag1", "tag2"],
    "access_control_mode": "owner_only|capability|access_control_list"
  },
  "signature": "signature-over-payload"
}
```

**Response:**
```json
{
  "data_id": "uuid",
  "storage_time": "iso-datetime",
  "storage_receipt": "storage-attestation-receipt",
  "resource_uri": "resource-uri-for-future-reference"
}
```

### Data Retrieval API

```
GET /api/v1/data/{data_id}
```

Retrieves encrypted data from the PDSL.

**Headers:**
```
Authorization: Bearer <access-token-or-capability>
```

**Response:**
```json
{
  "payload": "serialized-personal-data-payload",
  "access_receipt": "access-attestation-receipt"
}
```

### Data Query API

```
POST /api/v1/data/query
```

Performs a query against the data index.

**Request Body:**
```json
{
  "semantic_query": "natural-language-or-structured-query",
  "content_types": ["mime-type1", "mime-type2"],
  "tags": ["tag1", "tag2"],
  "time_range": {
    "start": "iso-datetime",
    "end": "iso-datetime"
  }
}
```

**Response:**
```json
{
  "results": [
    {
      "data_id": "uuid1",
      "relevance_score": 0.95,
      "metadata": "data-index-entry"
    },
    {
      "data_id": "uuid2",
      "relevance_score": 0.82,
      "metadata": "data-index-entry"
    }
  ],
  "total_matches": 42,
  "query_receipt": "query-attestation-receipt"
}
```

### Access Control API

```
POST /api/v1/data/{data_id}/access
```

Creates or updates access controls for data.

**Request Body:**
```json
{
  "access_grant": "serialized-storage-access-grant",
  "signature": "signature-over-grant"
}
```

**Response:**
```json
{
  "grant_id": "uuid",
  "status": "created|updated|replaced",
  "effective_time": "iso-datetime"
}
```

## Storage Backends

The PDSL supports multiple backend implementations:

### Solid Pod Backend

Reference implementation using Solid Protocol standards:
- WebID-based authentication
- WAC/ACP access control
- REST API with LDP containers
- Linked Data notifications for changes

### Encrypted Vector Store Backend

For ML-optimized storage:
- Document/embedding vector storage
- Supports similarity search
- Client-side encryption with server-side vector operations
- Compatible with FAISS, Qdrant, Pinecone

### Encrypted Object Store Backend

For blob and large media storage:
- S3-compatible API with SSE-C encryption
- Chunked storage for large files
- Content-addressable for deduplication
- MinIO or S3 as reference implementations

## Security Requirements

1. **Data Encryption**: All data must be encrypted client-side before storage.
2. **Key Management**: Encryption keys must never leave the user's control.
3. **Access Control**: Fine-grained permission model with capability-based delegation.
4. **Audit Logging**: Immutable logs of all access requests with cryptographic receipts.
5. **Integrity Verification**: Cryptographic verification of data integrity upon retrieval.

## Interoperability

To ensure interoperability, PDSL implementations must:

1. Support standardized Solid Protocol interfaces where applicable
2. Implement REST APIs with consistent resource patterns
3. Support JSON-LD for metadata exchange
4. Enable OAuth 2.0/OIDC for authentication flows
5. Implement capability-based sharing with standardized formats

## Implementation Considerations

- **Metadata Indexing**: Balance between searchability and privacy requires careful indexing design.
- **Caching Strategy**: Define clear cache invalidation to prevent stale data access.
- **Backup Procedures**: Implement secure backup strategies that preserve encryption.
- **Migration Path**: Enable migration between different backend implementations.

## Future Considerations

- **Federated Storage**: Support for data spanning multiple storage providers.
- **Private Information Retrieval**: PIR techniques to hide access patterns.
- **End-to-End Encrypted Search**: Homomorphic encryption for search over encrypted data. 