---
name: async-messaging
description: >-
  Event-driven architecture and asynchronous messaging patterns. Use when:
  designing pub/sub systems, implementing SQS/SNS/EventBridge workflows,
  building event-driven microservices, adding dead letter queues, designing
  message schemas, choosing between synchronous and asynchronous communication,
  auditing an existing system for messaging gaps, or implementing saga patterns.
  Covers AWS messaging services, message schema design, ordering guarantees,
  idempotency, error handling, and event sourcing.
tags:
  - developer
  - operations
---

# Async Messaging

## When to Use

- Decoupling services that don't need synchronous responses
- Designing event-driven workflows (order placed → payment → fulfillment)
- Implementing SQS queues, SNS topics, or EventBridge rules
- Building reliable background job processing
- Adding dead letter queues for failed message processing
- Designing message/event schemas for cross-service communication
- Choosing between synchronous API calls and async messaging
- Auditing an existing system for missing retries, DLQs, or ordering issues
- Implementing saga patterns for distributed transactions

---

## Core Principle: Design for Failure and Replay

**Every message will be delivered at least once. Some will be delivered more than once. Some will arrive out of order.** Design consumers to be idempotent and tolerant of duplicates and reordering.

---

## When to Use Async vs Sync

| Use async when | Use sync when |
|---------------|--------------|
| Consumer doesn't need immediate response | Caller needs the result to proceed |
| Work can be deferred (emails, reports, analytics) | User is waiting for the response (API request) |
| Producer and consumer scale independently | Both sides are in the same process |
| Retries and DLQ are needed for reliability | Simple request-response suffices |
| Multiple consumers need the same event | Only one consumer exists |
| Spiky workloads need buffering | Load is steady and predictable |

---

## AWS Messaging Services

### Service Selection

| Service | Pattern | Use for |
|---------|---------|---------|
| **SQS** | Point-to-point queue | Background jobs, task distribution, buffering |
| **SNS** | Pub/sub fan-out | Notifying multiple subscribers of an event |
| **EventBridge** | Event bus with rules | Cross-service events, scheduled triggers, third-party integrations |
| **Step Functions** | Orchestration | Multi-step workflows with branching, retries, and state |
| **Kinesis** | Streaming | High-throughput ordered event processing, analytics |

### SQS Configuration

```yaml
# serverless.yml
resources:
  Resources:
    OrderQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${sls:stage}-orders
        VisibilityTimeout: 300          # 6x Lambda timeout
        MessageRetentionPeriod: 1209600 # 14 days
        RedrivePolicy:
          deadLetterTargetArn: !GetAtt OrderDLQ.Arn
          maxReceiveCount: 3            # Move to DLQ after 3 failures

    OrderDLQ:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${sls:stage}-orders-dlq
        MessageRetentionPeriod: 1209600

functions:
  processOrder:
    handler: handlers/orders.process
    timeout: 50
    events:
      - sqs:
          arn: !GetAtt OrderQueue.Arn
          batchSize: 10
          functionResponseType: ReportBatchItemFailures
```

**Key settings**:
- `VisibilityTimeout`: At least 6x the Lambda timeout to prevent duplicate processing during retries.
- `maxReceiveCount`: 3 is a good default — retries transient failures without infinite loops.
- `ReportBatchItemFailures`: Return partial failures so only failed messages retry.

### SNS Fan-Out

```yaml
resources:
  Resources:
    OrderEventsTopic:
      Type: AWS::SNS::Topic
      Properties:
        TopicName: ${self:service}-${sls:stage}-order-events

    # Each subscriber gets its own queue
    InventoryQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${sls:stage}-inventory

    EmailQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${sls:stage}-email

    InventorySubscription:
      Type: AWS::SNS::Subscription
      Properties:
        TopicArn: !Ref OrderEventsTopic
        Protocol: sqs
        Endpoint: !GetAtt InventoryQueue.Arn
        FilterPolicy:
          event_type:
            - order.placed
            - order.cancelled
```

### EventBridge Rules

```yaml
resources:
  Resources:
    OrderPlacedRule:
      Type: AWS::Events::Rule
      Properties:
        EventBusName: ${self:service}-${sls:stage}
        EventPattern:
          source:
            - orders
          detail-type:
            - OrderPlaced
        Targets:
          - Arn: !GetAtt FulfillmentQueue.Arn
            Id: fulfillment
          - Arn: !GetAtt AnalyticsQueue.Arn
            Id: analytics
```

**Use EventBridge when**: Events cross service boundaries, you need content-based routing, or you want a central event bus.

---

## Message Schema Design

### Standard Envelope

Every message follows the same envelope:

```json
{
  "id": "msg_01HXYZ123ABC",
  "source": "orders",
  "type": "order.placed",
  "timestamp": "2025-01-15T12:00:00Z",
  "version": "1.0",
  "data": {
    "order_id": "ord_456",
    "customer_id": "cust_789",
    "total_cents": 4999
  },
  "metadata": {
    "correlation_id": "req_abc123",
    "trace_id": "trace_def456"
  }
}
```

### Schema Rules

- **`id`**: Globally unique message ID (ULID or UUID). Used for idempotency.
- **`source`**: Service that produced the event.
- **`type`**: Dot-separated event name (`domain.action`).
- **`version`**: Schema version. Increment on breaking changes.
- **`data`**: Event payload. Contains only the facts — not commands or instructions.
- **`metadata`**: Correlation and tracing IDs for observability.

### Schema Evolution

- **Additive changes** (new optional fields): Safe. Consumers ignore unknown fields.
- **Removing fields**: Breaking. Use schema versioning.
- **Changing field types**: Breaking. Use schema versioning.
- **Rule**: Consumers must tolerate unknown fields. Producers must not remove fields without a version bump.

---

## Idempotent Consumers

Every consumer must handle duplicate messages safely:

```python
from typing import Any


def process_order(message: dict[str, Any]) -> None:
    """Process an order event idempotently."""
    message_id = message["id"]

    # Check if already processed
    if is_processed(message_id):
        logger.info("Duplicate message, skipping", extra={"message_id": message_id})
        return

    # Process the event
    order = message["data"]
    create_fulfillment(order["order_id"])

    # Mark as processed (with TTL matching message retention)
    mark_processed(message_id, ttl_days=14)
```

### Idempotency Storage

| Option | Best for | Notes |
|--------|----------|-------|
| DynamoDB table with TTL | Lambda consumers | Fast, serverless, auto-cleanup |
| Database unique constraint | Services with existing DB | Use `INSERT ... ON CONFLICT DO NOTHING` |
| Redis SET with TTL | High-throughput consumers | Fast but volatile |

### DynamoDB Idempotency Table

```yaml
resources:
  Resources:
    IdempotencyTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:service}-${sls:stage}-idempotency
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: message_id
            AttributeType: S
        KeySchema:
          - AttributeName: message_id
            KeyType: HASH
        TimeToLiveSpecification:
          AttributeName: expires_at
          Enabled: true
```

---

## Partial Batch Failures

When processing SQS batches, report individual failures so only failed messages retry:

```python
from typing import Any


def handler(event: dict[str, Any], context: Any) -> dict[str, Any]:
    """SQS Lambda handler with partial batch failure reporting."""
    failed_ids: list[str] = []

    for record in event["Records"]:
        try:
            message = json.loads(record["body"])
            process_message(message)
        except Exception:
            logger.exception("Failed to process message", extra={
                "message_id": record["messageId"],
            })
            failed_ids.append(record["messageId"])

    return {
        "batchItemFailures": [
            {"itemIdentifier": msg_id} for msg_id in failed_ids
        ],
    }
```

---

## Ordering Guarantees

| Service | Ordering | Use when |
|---------|----------|----------|
| SQS Standard | Best-effort (no guarantee) | Ordering doesn't matter |
| SQS FIFO | Strict within message group | Order-sensitive within an entity (per-customer, per-order) |
| Kinesis | Strict within shard (partition key) | High-throughput ordered streams |
| EventBridge | No ordering guarantee | Event routing, not sequencing |

### SQS FIFO Pattern

```python
import json

sqs = boto3.client("sqs")


def publish_order_event(order_id: str, event: dict[str, Any]) -> None:
    """Publish to FIFO queue with order-level ordering."""
    sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(event),
        MessageGroupId=order_id,              # All events for this order are ordered
        MessageDeduplicationId=event["id"],   # Prevent duplicates within 5-minute window
    )
```

---

## Dead Letter Queue (DLQ) Processing

### DLQ Monitoring

Set alarms on DLQ depth:

```yaml
resources:
  Resources:
    DLQAlarm:
      Type: AWS::CloudWatch::Alarm
      Properties:
        AlarmName: ${self:service}-${sls:stage}-dlq-depth
        MetricName: ApproximateNumberOfMessagesVisible
        Namespace: AWS/SQS
        Dimensions:
          - Name: QueueName
            Value: !GetAtt OrderDLQ.QueueName
        Statistic: Sum
        Period: 300
        EvaluationPeriods: 1
        Threshold: 1
        ComparisonOperator: GreaterThanOrEqualToThreshold
        AlarmActions:
          - !Ref AlertTopic
```

### DLQ Replay

After fixing the root cause, replay DLQ messages back to the main queue:

```bash
# AWS CLI: move messages from DLQ back to main queue
aws sqs start-message-move-task \
  --source-arn arn:aws:sqs:us-east-1:123456789:orders-dlq \
  --destination-arn arn:aws:sqs:us-east-1:123456789:orders
```

---

## Saga Pattern (Distributed Transactions)

When a workflow spans multiple services, use a saga to coordinate:

### Choreography (Event-Driven)

Each service emits an event on completion. The next service reacts to it.

```
OrderPlaced → PaymentProcessed → InventoryReserved → ShipmentCreated
     ↓ (failure)        ↓ (failure)
OrderCancelled     PaymentRefunded
```

**Pros**: Decoupled, no central coordinator.
**Cons**: Hard to trace, compensating actions spread across services.

### Orchestration (Step Functions)

A central coordinator drives the workflow:

```json
{
  "StartAt": "ProcessPayment",
  "States": {
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123:function:process-payment",
      "Next": "ReserveInventory",
      "Catch": [{
        "ErrorEquals": ["PaymentFailed"],
        "Next": "CancelOrder"
      }]
    },
    "ReserveInventory": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123:function:reserve-inventory",
      "Next": "CreateShipment",
      "Catch": [{
        "ErrorEquals": ["States.ALL"],
        "Next": "RefundPayment"
      }]
    },
    "CreateShipment": { "Type": "Succeed" },
    "RefundPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123:function:refund-payment",
      "Next": "CancelOrder"
    },
    "CancelOrder": { "Type": "Fail" }
  }
}
```

**Pros**: Centralized visibility, clear failure handling.
**Cons**: Single coordinator is a dependency.

**Use orchestration for**: Multi-step workflows with branching, retries, and compensating actions.
**Use choreography for**: Simple fan-out where each consumer acts independently.

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| No DLQ on any queue | Failed messages silently lost after max retries | Always attach a DLQ; alarm on depth > 0 |
| Processing messages without idempotency | Duplicates cause double-charges, double-sends | Use message ID deduplication with idempotency table |
| Synchronous HTTP in event handler | Coupling, latency, cascading failures | Use message queues between services |
| Unbounded batch size | Lambda timeout on large batches | Set `batchSize` ≤ 10 for SQS; tune for workload |
| No visibility timeout tuning | Messages re-appear while still processing | Set visibility timeout to 6x Lambda timeout |
| Tight coupling via message content | Consumer breaks when producer changes payload | Use schema versioning and tolerate unknown fields |
| No monitoring on DLQ | Failed messages accumulate unnoticed | Alarm on `ApproximateNumberOfMessagesVisible > 0` |
| Processing order-dependent events on standard queue | Race conditions, inconsistent state | Use FIFO queue with message group ID per entity |
| No correlation ID in events | Cannot trace requests across services | Include `correlation_id` in message metadata |

---

## Audit Checklist

When auditing an existing system for messaging patterns:

- [ ] Every queue has a dead letter queue configured
- [ ] DLQ depth is monitored with alarms
- [ ] DLQ replay process is documented and tested
- [ ] Consumers are idempotent (handle duplicate messages safely)
- [ ] Messages follow a standard envelope schema (id, source, type, version, data)
- [ ] Visibility timeout is at least 6x the consumer timeout
- [ ] Partial batch failures are reported (`ReportBatchItemFailures` for SQS+Lambda)
- [ ] Ordering requirements are met (FIFO queues where needed)
- [ ] Correlation IDs propagate through the message chain
- [ ] Schema evolution strategy exists (additive-only or versioned)
- [ ] No synchronous HTTP calls between services that should be async
- [ ] Message retention configured appropriately (14 days for SQS)
- [ ] Step Functions used for multi-step workflows with compensating actions
