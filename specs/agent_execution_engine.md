# Agent Execution Engine (AEE)

## Overview

The Agent Execution Engine (AEE) is the compute layer responsible for executing models and agents within a confidential environment. It provides a secure runtime for LLMs and other AI models to operate on personal data with privacy guarantees, enabled by Trusted Execution Environments (TEEs).

## Containers

### ModelExecutionRequest

```
class ModelExecutionRequest(Container):
    request_id: UUID
    model_id: String
    input_data_references: List[URI]
    execution_parameters: JSON
    output_specifications: OutputSpec
    attestation_requirements: AttestationReqs
    execution_signature: Bytes
```

### AgentExecutionRequest

```
class AgentExecutionRequest(Container):
    request_id: UUID
    agent_definition: AgentDefinition
    goals: List[String]
    constraints: List[String]
    input_data_references: List[URI]
    tool_permissions: List[ToolPermission]
    max_execution_time: Duration
    execution_signature: Bytes
```

### ModelExecutionResult

```
class ModelExecutionResult(Container):
    request_id: UUID
    model_id: String
    output_data: Bytes  # Encrypted for the requestor
    performance_metrics: ExecutionMetrics
    execution_receipt: ExecutionReceipt
    result_signature: Bytes
```

### AgentExecutionResult

```
class AgentExecutionResult(Container):
    request_id: UUID
    goal_completion_status: JSON
    output_data: Bytes  # Encrypted for the requestor
    tool_usage_summary: List[ToolUsageRecord]
    execution_trace: Bytes  # Encrypted execution history
    execution_receipt: ExecutionReceipt
    result_signature: Bytes
```

### AgentDefinition

```
class AgentDefinition(Container):
    agent_id: String
    model_dependencies: List[ModelDependency]
    tools_required: List[ToolDefinition]
    memory_configuration: MemoryConfig
    prompt_templates: Dict[String, String]
    reasoning_framework: String  # e.g., "ReAct", "ReflexionGPT"
```

### ExecutionReceipt

```
class ExecutionReceipt(Container):
    execution_id: UUID
    model_fingerprint: Bytes  # Hash of the model used
    input_fingerprints: List[Bytes]  # Hashes of input data
    operation_summary: String  # Description of operations performed
    timestamp_range: (Timestamp, Timestamp)
    tee_attestation: Bytes
    resource_usage: ResourceUsage
```

## APIs

### Model Execution API

```
POST /api/v1/models/execute
```

Executes a specific model on provided data.

**Request Body:**
```json
{
  "model_execution_request": "serialized-model-execution-request",
  "sat_token": "serialized-scheduled-access-token",
  "signature": "signature-over-request"
}
```

**Response:**
```json
{
  "request_id": "uuid",
  "status": "accepted|rejected",
  "estimated_completion_time": "iso-datetime",
  "execution_id": "execution-uuid-if-accepted"
}
```

### Agent Execution API

```
POST /api/v1/agents/execute
```

Executes an agent with specified goals and constraints.

**Request Body:**
```json
{
  "agent_execution_request": "serialized-agent-execution-request",
  "sat_token": "serialized-scheduled-access-token",
  "signature": "signature-over-request"
}
```

**Response:**
```json
{
  "request_id": "uuid",
  "status": "accepted|rejected",
  "estimated_completion_time": "iso-datetime",
  "execution_id": "execution-uuid-if-accepted"
}
```

### Execution Status API

```
GET /api/v1/execution/{execution_id}/status
```

Checks the status of a previously submitted execution.

**Response:**
```json
{
  "execution_id": "uuid",
  "status": "queued|preparing|executing|completed|failed",
  "progress": 0.75,
  "start_time": "iso-datetime-or-null",
  "estimated_completion": "iso-datetime-or-null",
  "status_message": "detailed-status-information"
}
```

### Execution Result API

```
GET /api/v1/execution/{execution_id}/result
```

Retrieves the result of a completed execution.

**Response:**
```json
{
  "execution_result": "serialized-model-or-agent-execution-result",
  "signature": "signature-over-result"
}
```

## Supported Model Frameworks

The AEE supports multiple model frameworks:

### LLM Frameworks
- **vLLM**: Optimized for performance and memory efficiency
- **Hugging Face TGI**: Standard-compliant inference API
- **ONNX Runtime**: Hardware-accelerated inference
- **LLaMA.cpp**: Lightweight inference for smaller deployments

### Agent Frameworks
- **LangChain**: Tool integration and agent orchestration
- **AutoGen**: Multi-agent collaboration and reasoning
- **CrewAI**: Specialized for multi-agent workflows
- **Custom Agent Specifications**: Using the AEE Agent Definition Format

## Security Guarantees

1. **Model Integrity**: Models are verified against known checksums before execution.
2. **Input/Output Confidentiality**: Data is encrypted in transit and at rest, only decrypted within the TEE.
3. **Execution Isolation**: Agents run in isolated environments preventing cross-request contamination.
4. **Tool Safety**: Tool permissions are strictly enforced via capability-based security.
5. **Verifiable Execution**: Execution receipts provide cryptographic proof of operations performed.

## Tool Integration

The AEE provides a capability-constrained tool API for agents:

### Native Tool Types
- **Data Retrieval**: Access to authorized data sources
- **Web Search**: Privacy-preserving web search capabilities
- **Connectors**: Integration with external services
- **Filesystem**: Scoped access to temporary workspace

### Tool Permission Model

```
class ToolPermission(Container):
    tool_id: String
    allowed_operations: List[String]
    resource_scope: URIPattern
    rate_limits: RateLimitSpec
    invocation_quota: Integer
```

## Implementation Requirements

1. **TEE Compatibility**: Must run within SGX, TDX, or SEV-SNP enclaves.
2. **Model Loading**: Secure process for loading and verifying model weights.
3. **Memory Management**: Efficient handling of models exceeding enclave memory limits.
4. **Scheduling**: Fair-share scheduling for multi-tenant deployments.
5. **Logging**: Detailed, encrypted audit logs of all operations.

## Operational Considerations

- **Resource Requirements**: Varies by model, but typically 16-64GB RAM, 8-32 CPU cores.
- **GPU Acceleration**: Optional support for confidential GPU computing where available.
- **Scaling**: Horizontal scaling through orchestrated AEE instances.
- **Failover**: Automatic recovery and request rescheduling on hardware/software failures.

## Future Considerations

1. **Hardware Acceleration**: Integration with emerging confidential GPU technologies.
2. **Multi-Model Execution**: Orchestration of complex pipelines spanning multiple models.
3. **Continuous Learning**: Privacy-preserving incremental learning within TEE boundaries.
4. **Federated Execution**: Distributed execution across multiple user-owned devices/enclaves. 