# Kafka

## Overview

Sends messages to a Kafka topic. This package provides comprehensive activities for Apache Kafka messaging, including message production and consumption with support for schema registry, consumer groups, and advanced Kafka features.

## Components

### Activities

#### Message Production

- **Send Kafka Message** (`kafka.producer.sendMessage`): Sends messages to a Kafka topic

#### Message Consumption

- **Consume Kafka Message** (`kafka.consumer.consumeMessage`): Consumes messages from a Kafka topic

## Dependencies

### Required

None - this package has no required dependencies.

### Optional

None - this package currently has no optional dependencies.

## Usage

### Message Production

```yaml
# Send a simple message
activity: kafka.producer.sendMessage
inputs:
  topic: "user-events"
  message: '{"userId": "12345", "action": "login", "timestamp": "2024-01-15T10:30:00Z"}'

# Send message with key
activity: kafka.producer.sendMessage
inputs:
  topic: "user-events"
  message: '{"userId": "12345", "action": "purchase", "amount": 99.99}'
  useKey: "true"
  key: "user-12345"

# Send message with custom headers
activity: kafka.producer.sendMessage
inputs:
  topic: "audit-logs"
  message: '{"event": "user_action", "details": {...}}'
  headers: '{"source": "web-app", "version": "1.0", "timestamp": "2024-01-15T10:30:00Z"}'
  options: '{"acks": "all", "compression.type": "gzip"}'

# Send message with schema registry
activity: kafka.producer.sendMessage
inputs:
  topic: "schema-validated-events"
  message: '{"id": "123", "name": "John Doe", "email": "john@example.com"}'
  useSchemaRegistry: "true"
  schemaRegistryUrl: "http://schema-registry:8081"
  eventName: "user.profile.updated"
```

### Message Consumption

```yaml
# Basic consumer
activity: kafka.consumer.consumeMessage
inputs:
  topic: "user-events"
  groupId: "analytics-processor"

# Consumer with advanced settings
activity: kafka.consumer.consumeMessage
inputs:
  topic: "order-events"
  groupId: "order-processor"
  fromBeginning: "false"
  parallelProcessing: "true"
  jsonParseMessage: "true"
  autoCommitThreshold: "100"
  sessionTimeout: "30000"

# Consumer with schema registry
activity: kafka.consumer.consumeMessage
inputs:
  topic: "schema-validated-events"
  groupId: "validation-processor"
  useSchemaRegistry: "true"
  schemaRegistryUrl: "http://schema-registry:8081"
  returnHeaders: "true"

# High-throughput consumer
activity: kafka.consumer.consumeMessage
inputs:
  topic: "high-volume-events"
  groupId: "stream-processor"
  parallelProcessing: "true"
  maxInFlightRequests: "5"
  autoCommitInterval: "1000"
  heartbeatInterval: "3000"
```

### Event-Driven Architecture

```yaml
# Complete event processing pipeline
steps:
  - name: Produce User Event
    activity: kafka.producer.sendMessage
    inputs:
      topic: "user-lifecycle"
      message: '{"userId": "${USER_ID}", "event": "registration", "metadata": {...}}'
      useKey: "true"
      key: "user-${USER_ID}"
      headers: '{"source": "registration-service", "version": "2.0"}'

  - name: Process User Events
    activity: kafka.consumer.consumeMessage
    inputs:
      topic: "user-lifecycle"
      groupId: "user-processor"
      jsonParseMessage: "true"
      autoCommitThreshold: "50"

  - name: Produce Notification Event
    activity: kafka.producer.sendMessage
    inputs:
      topic: "notifications"
      message: '{"userId": "${USER_ID}", "type": "welcome_email", "priority": "high"}'
      options: '{"acks": "all", "retries": 3}'
```

## Configuration

### Required Configuration

All activities require Kafka connectivity:

- **topic**: Name of the Kafka topic
- **groupId**: Consumer group ID (for consumers)

### Authentication

These activities use console handlers for demonstration and validation purposes. In a production environment, you would need:

- Kafka cluster connectivity
- Proper authentication credentials (SASL, SSL, etc.)
- Network access to Kafka brokers
- Schema Registry access (if using schema validation)

### Activity-Specific Configuration

#### Message Production

- **topic**: Target topic name
- **message**: Message content (typically JSON)
- **key**: Optional message key for partitioning
- **headers**: Optional message headers
- **options**: Producer configuration options

#### Message Consumption

- **topic**: Source topic name
- **groupId**: Consumer group for load balancing
- **fromBeginning**: Whether to read from topic beginning
- **parallelProcessing**: Enable/disable parallel message processing
- **autoCommitThreshold**: Number of messages before committing offsets

#### Schema Registry Integration

- **useSchemaRegistry**: Enable schema validation
- **schemaRegistryUrl**: Schema Registry endpoint
- **eventName**: Schema namespace and name

## Kafka Integration Patterns

### Event Sourcing

```yaml
# Event sourcing pattern
event_store:
  - topic: "user-events"
    events: ["created", "updated", "deleted"]

  - topic: "order-events"
    events: ["placed", "paid", "shipped", "delivered"]

  - topic: "inventory-events"
    events: ["restocked", "depleted", "reserved"]
```

### CQRS (Command Query Responsibility Segregation)

```yaml
# Separate read and write models
command_side:
  - topic: "user-commands"
    operations: ["create", "update", "delete"]

query_side:
  - topic: "user-views"
    projections: ["user-profile", "user-activity", "user-preferences"]
```

### Microservices Communication

```yaml
# Inter-service communication
services:
  - name: user-service
    produces: ["user-events"]
    consumes: ["notification-events"]

  - name: order-service
    produces: ["order-events"]
    consumes: ["user-events", "inventory-events"]

  - name: notification-service
    produces: ["notification-events"]
    consumes: ["user-events", "order-events"]
```

## Best Practices

### Message Production

- Use meaningful topic names that reflect business domains
- Include message keys for proper partitioning
- Use consistent message schemas across services
- Implement proper error handling and retries
- Include relevant headers for tracing and debugging

### Message Consumption

- Use descriptive consumer group IDs
- Implement idempotent message processing
- Configure appropriate commit strategies
- Monitor consumer lag and performance
- Handle message deserialization errors gracefully

### Topic Management

- Design topic partitioning strategy based on throughput needs
- Set appropriate retention policies for different data types
- Use topic naming conventions that reflect data lifecycle
- Implement proper access controls and security policies

### Schema Management

- Version your message schemas appropriately
- Use Schema Registry for schema evolution
- Implement backward and forward compatibility
- Document schema changes and migration strategies

## Event-Driven Workflows

### Real-Time Analytics

```yaml
# Real-time data pipeline
analytics_pipeline:
  - name: Collect User Actions
    activity: kafka.producer.sendMessage
    inputs:
      topic: "user-actions"
      message: "${USER_ACTION_EVENT}"

  - name: Process Actions
    activity: kafka.consumer.consumeMessage
    inputs:
      topic: "user-actions"
      groupId: "analytics-processor"
      parallelProcessing: "true"

  - name: Store Aggregated Data
    activity: kafka.producer.sendMessage
    inputs:
      topic: "analytics-results"
      message: "${AGGREGATED_METRICS}"
```

### Saga Pattern Implementation

```yaml
# Distributed transaction management
saga_workflow:
  - name: Start Transaction
    activity: kafka.producer.sendMessage
    inputs:
      topic: "saga-events"
      message: '{"sagaId": "${SAGA_ID}", "step": "start", "operation": "order"}'

  - name: Process Transaction Steps
    activity: kafka.consumer.consumeMessage
    inputs:
      topic: "saga-events"
      groupId: "saga-coordinator"

  - name: Handle Compensation
    activity: kafka.producer.sendMessage
    inputs:
      topic: "compensation-events"
      message: '{"sagaId": "${SAGA_ID}", "action": "compensate"}'
```

### Change Data Capture (CDC)

```yaml
# Database change streaming
cdc_pipeline:
  - name: Capture Database Changes
    activity: kafka.consumer.consumeMessage
    inputs:
      topic: "db-changes"
      groupId: "cdc-processor"
      fromBeginning: "false"

  - name: Transform and Route
    activity: kafka.producer.sendMessage
    inputs:
      topic: "entity-updates"
      message: "${TRANSFORMED_CHANGE_EVENT}"
```

## Performance Optimization

### Producer Optimization

```yaml
# High-throughput producer settings
high_throughput_producer:
  options: |
    {
      "batch.size": 16384,
      "linger.ms": 5,
      "compression.type": "snappy",
      "acks": "1",
      "buffer.memory": 33554432
    }
```

### Consumer Optimization

```yaml
# High-throughput consumer settings
high_throughput_consumer:
  inputs:
    parallelProcessing: "true"
    maxInFlightRequests: "5"
    autoCommitInterval: "1000"
    sessionTimeout: "30000"
    heartbeatInterval: "3000"
```

### Monitoring and Metrics

- Track producer and consumer throughput
- Monitor topic partition distribution
- Watch consumer group lag metrics
- Alert on failed message deliveries
- Monitor schema registry usage and errors

## Troubleshooting

### Common Issues

- **Message delivery failures**: Check broker connectivity and authentication
- **Consumer lag**: Optimize consumer processing or increase parallelism
- **Schema validation errors**: Verify schema compatibility and registration
- **Partition assignment issues**: Review consumer group configuration

### Error Handling

- Implement dead letter topics for failed messages
- Use appropriate retry policies with exponential backoff
- Log detailed error information for debugging
- Establish alerting for critical failure scenarios

### Performance Issues

- Monitor and tune batch sizes and commit intervals
- Optimize serialization and deserialization
- Review partition assignment and rebalancing
- Scale consumers horizontally based on throughput needs

## Security Considerations

### Authentication and Authorization

- Configure SASL authentication for production environments
- Implement proper ACLs for topic access control
- Use SSL/TLS for encrypted communication
- Secure Schema Registry access with authentication

### Message Security

- Encrypt sensitive data within messages
- Use message headers for audit trails
- Implement message signing for integrity verification
- Consider field-level encryption for PII data

### Network Security

- Configure firewall rules for Kafka broker access
- Use VPN or private networks for cross-region communication
- Implement proper certificate management for SSL
- Monitor and log access patterns for security analysis

## Multi-Environment Setup

### Development Environment

```yaml
development:
  kafka_cluster: "localhost:9092"
  schema_registry: "http://localhost:8081"
  topics:
    - "dev-user-events"
    - "dev-order-events"
```

### Staging Environment

```yaml
staging:
  kafka_cluster: "kafka-staging:9092"
  schema_registry: "http://schema-registry-staging:8081"
  topics:
    - "staging-user-events"
    - "staging-order-events"
```

### Production Environment

```yaml
production:
  kafka_cluster: "kafka-prod-1:9092,kafka-prod-2:9092,kafka-prod-3:9092"
  schema_registry: "https://schema-registry-prod:8081"
  security:
    ssl: true
    sasl: true
  topics:
    - "user-events"
    - "order-events"
```

## Notes

- All activities in this package use console handlers for output and validation
- These activities provide structure and validation for Kafka operations
- For production use, consider implementing corresponding PipelineRef handlers with actual Kafka client integration
- Ensure proper Kafka cluster access and authentication before using these activities
- Consider implementing message transformation and routing capabilities
- Regular monitoring of Kafka cluster health and consumer group performance is recommended
