# LLM Context for Project_X

This document provides context for Large Language Models to understand the purpose and structure of the Project_X repository.

## Repository Purpose

Project_X is a specification repository for the Confidential Data and Compute Delegation Specification (CDCD Spec). It defines an architecture and standard enabling confidential storage and computation delegation over personal private data using object capabilities (OCaps) and Trusted Execution Environments (TEEs).

## Repository Structure

The repository is organized as follows:

```
project_x/
├── README.md                        # Main spec overview and introduction
├── LLM_Context.md                   # This file - context for LLMs
├── .gitignore                       # Standard git ignore patterns
└── specs/                           # Directory containing component specifications
    ├── confidential_runtime.md      # TEE-enabled execution environment spec
    ├── personal_data_storage.md     # Secure data storage infrastructure spec
    ├── scheduled_access_token.md    # Capability tokens with time constraints spec
    ├── agent_execution_engine.md    # AI model and agent execution framework spec
    ├── connectors.md                # External service integration adapters spec
    ├── confidential_compute_layer.md # Secure execution infrastructure layer spec
    ├── data_storage_layer.md        # Modular storage infrastructure layer spec
    └── capability_delegation_layer.md # Object capability foundation layer spec
```

## File Descriptions

### Root Files

1. **README.md** - The main entry point that provides:
   - Version information and abstract
   - Motivation and prerequisites 
   - Terminology and system parameters
   - Links to component specs
   - Security, privacy, and system invariants
   - Implementation guidance and backward compatibility

2. **LLM_Context.md** - This file, providing organization and context for LLMs to understand the repository.

3. **.gitignore** - Standard git ignore file with patterns for common temporary files, build artifacts, and environment configurations.

### Component Specifications

1. **confidential_runtime.md** - Defines the Confidential Runtime (CR) component:
   - TEE-enabled environment for secure computation
   - Container definitions for computation requests and results
   - APIs for attestation, delegation, status checking, and result retrieval
   - Security guarantees and operational requirements
   - Failure modes and implementation considerations

2. **personal_data_storage.md** - Covers the Personal Data Storage Layer (PDSL):
   - Secure, user-controlled storage infrastructure
   - Container definitions for data payloads and access grants
   - APIs for data storage, retrieval, and access control
   - Multiple backend implementation options (Solid, Vector, Object stores)
   - Security requirements and interoperability considerations

3. **scheduled_access_token.md** - Details the Scheduled Access Token (SAT) system:
   - Object-capability tokens with time-based constraints
   - Containers for tokens, invocations, and revocations
   - Schedule format based on RFC 5545 (iCalendar) RRULE
   - APIs for token issuance, verification, and revocation
   - Security constraints and implementation guidelines

4. **agent_execution_engine.md** - Specifies the Agent Execution Engine (AEE):
   - Compute layer for executing models within confidential environments
   - Containers for execution requests, results, and agent definitions
   - APIs for model and agent execution
   - Supported frameworks for LLMs and agents
   - Security guarantees and tool integration patterns

5. **connectors.md** - Describes the Connectors component:
   - Stateless adapters for external service interaction
   - Containers for connector definitions, credentials, and requests
   - APIs for registry, authentication, operation, and credential management
   - Supported connector types for various services
   - Security requirements and implementation guidelines

### Layer Specifications

1. **confidential_compute_layer.md** - Defines the Confidential Compute Layer:
   - Architecture combining CR, AEE, attestation, and key management
   - Interaction and data flow patterns
   - Security boundaries and attestation protocol
   - TEE technology support
   - APIs for layer management and attestation

2. **data_storage_layer.md** - Specifies the Data Storage and Access Layer:
   - Architecture combining PDSL, access control, encryption, and indexing
   - Backend options and data flow
   - Data lifecycle and access control model
   - Encryption architecture and query patterns
   - APIs for storage, access, and querying

3. **capability_delegation_layer.md** - Details the Capability Delegation Layer:
   - Architecture for secure, controlled delegation of rights
   - Capability model and delegation patterns
   - Multiple capability token formats
   - APIs for capability issuance, verification, and revocation
   - Revocation mechanisms and security considerations

## Key Concepts to Understand

When processing this repository, keep in mind these core concepts:

1. **Object Capabilities (OCaps)** - Security model where access is granted through unforgeable references

2. **Trusted Execution Environments (TEEs)** - Hardware-based security environments like Intel SGX, AMD SEV-SNP

3. **Solid Pods** - User-controlled personal data stores based on W3C Solid Protocol

4. **Scheduled Access Tokens** - Time-based capability tokens for controlled delegation

5. **Confidential Computing** - Protecting data in use through hardware isolation

6. **Data Sovereignty** - Users maintaining control over their personal data

## Intended Use

This specification repository is intended to:

1. Define standards for confidential data storage and computation delegation
2. Provide guidance for implementing compatible systems
3. Establish security and privacy guarantees for personal data
4. Enable modularity through well-defined components and interfaces

When answering questions about this repository, focus on the architectural components, their interactions, security guarantees, and implementation considerations as detailed in the specifications. 