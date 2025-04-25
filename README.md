# Confidential Data and Compute Delegation Specification (CDCD Spec)

Version 0.1.0-pre — Last updated 2025-04-24  
(Draft – feedback welcome, breaking changes expected)

## Abstract

Introduces an architecture and standard enabling confidential storage and computation delegation over personal private data using object capabilities (OCaps) and Trusted Execution Environments (TEEs). This specification describes a modular and extensible system capable of securely hosting private user data, delegating controlled computation to personally hosted or cloud-hosted confidential large language models (LLMs), and safely interacting with external services, such as social media platforms. Inspired by Web3 data sovereignty efforts, notably Solid Pods, and object-capability security principles, this system allows for modularity in backend storage, capability delegation, and compute frameworks, enabling scalability and customizability.

## Prerequisites

Familiarity with the following concepts, specifications, and technologies is assumed:
- 🔗 Object Capabilities (OCaps)
- 🔗 Web3 Solid Protocol
- 🔗 Confidential Computing TEEs (SGX, TDX, SEV-SNP)
- 🔗 Ethereum JSON-RPC APIs
- 🔗 Flashblocks Specification (style reference)

## Motivation

Today's internet users increasingly demand control and privacy over their personal data. Current centralized architectures lack meaningful data sovereignty, putting sensitive information at risk and limiting user autonomy. At the same time, modern AI services often require extensive access to personal data, complicating privacy concerns and making secure delegation of compute challenging.

Existing decentralized data initiatives, such as Solid Pods, offer promising data sovereignty but currently lack seamless and confidential delegation of computation to cloud environments. Similarly, trusted execution environments (TEEs) and object-capability (OCap) architectures provide promising security foundations but remain fragmented and underutilized in production-grade user-facing applications.

This specification bridges these gaps by defining a modular architecture that integrates Solid-inspired data pods, object capabilities for precise and revocable access control, and confidential computing via TEEs. The resulting architecture securely delegates computation over private data to either personally hosted or cloud-hosted confidential LLM agents, offering strong guarantees of privacy, modular scalability, and user-driven delegation.

## Terminology

- **Confidential Runtime (CR)**: TEE-enabled execution environment that securely executes delegated computations.
- **Personal Data Storage Layer (PDSL)**: Modular, secure backend storage for user data, supporting Solid Pods or alternative storage backends.
- **Scheduled Access Token (SAT)**: Object-capability token embedding delegation rules (frequency, duration, permissions).
- **Agent Execution Engine (AEE)**: Compute layer executing models/agents within a confidential environment.
- **Solid Pod**: Standards-based decentralized user data store adhering to Solid Protocol specifications.
- **Connector**: Stateless adapter facilitating interaction with external services (e.g., social media platforms).

## Parameters

| Constant | Recommended Value | Description |
|----------|-------------------|-------------|
| MAX_SAT_DURATION | 365 days | Maximum validity period for SATs |
| DEFAULT_EXEC_INTERVAL | 24 hours | Default frequency of scheduled delegated compute |

## System Architecture

The system is divided into three primary modular components:
- [Confidential Compute Layer](specs/confidential_compute_layer.md)
- [Data Storage and Access Layer](specs/data_storage_layer.md)
- [Capability Delegation Layer](specs/capability_delegation_layer.md)

## Component Specifications

- [Confidential Runtime (CR)](specs/confidential_runtime.md)
- [Personal Data Storage Layer (PDSL)](specs/personal_data_storage.md)
- [Scheduled Access Token (SAT)](specs/scheduled_access_token.md)
- [Agent Execution Engine (AEE)](specs/agent_execution_engine.md)
- [Connectors](specs/connectors.md)

## Security & Privacy Considerations

- **Key Management**: User-owned KMS ensures that cloud providers never access keys.
- **Capability Revocation**: Instant revocation via deletion of SAT or underlying Solid Access Grant.
- **Side-channel Mitigation**: TEEs mitigate side-channels; additional noise masking recommended.
- **Auditability**: Immutable receipts for every action to ensure audit trail completeness and tamper resistance.

## System Invariants

- **No unauthorized data access**: All data interactions must be capability-proven and auditable.
- **Scheduled delegation adherence**: Invocations strictly adhere to schedules and capability constraints.
- **Data integrity and confidentiality**: All user data remains encrypted at rest and in transit; decrypted only within attested TEE environment.

## Reliability and Operational Considerations

- **Failover**: CR instances support failover with no loss of confidentiality guarantees due to attestation.
- **Scalability**: Storage backends can scale independently; vector caches recommended for high-throughput scenarios.
- **High Availability (HA)**: Future specs may define detailed HA setups for confidential runtime clusters.

## Implementation Guidance

Initial reference implementations are recommended using:
- **TEE**: Azure Confidential VMs or AWS Nitro Enclaves
- **Storage Backend**: Solid Community Server (development) → Inrupt ESS (production)
- **Capabilities**: Biscuit tokens wrapped over Solid Access Grants
- **Compute Framework**: vLLM, Hugging Face TGI, or AutoGen runtimes

## Backwards Compatibility

- Modular design enables progressive adoption without legacy disruption.
- Existing Solid Pods compatible without modifications.

## Open Questions

- Optimal grammar and standardization of SAT scheduling
- Side-channel resistance of GPU TEEs under high load conditions
- Interoperability testing with various Solid implementations
