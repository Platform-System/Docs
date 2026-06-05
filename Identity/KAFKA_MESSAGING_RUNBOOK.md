# Kafka Messaging Runbook

## Scope

This runbook covers the current `Identity -> Kafka -> Wallet` flow:

- producer-side outbox in `Platform.Identity.API`
- consumer-side inbox in `Platform.Wallet.API`
- topics:
  - `identity.user-provisioned`
  - `identity.user-provisioned.retry`
  - `identity.user-provisioned.dlt`

## Data Stores

### Identity DB

- `OutboxMessages`
  - pending publish rows
  - retry metadata
  - DLT handoff source

### Wallet DB

- `InboxMessages`
  - deduplication ledger per `MessageId + ConsumerGroup`

## Pre-flight Checks

Before replaying or cleaning anything:

1. confirm Kafka is healthy
2. confirm `Identity` and `Wallet` containers are healthy
3. confirm DB connectivity is normal
4. confirm whether the issue is:
   - producer/outbox side
   - consumer/retry side
   - DLT accumulation only

## Normal Flow

1. `SyncUserSession` writes user changes.
2. A pre-commit notification writes `IdentityUserSynced` into `OutboxMessages`.
3. `OutboxDispatcher` publishes to `identity.user-provisioned`.
4. `IdentityUserSyncedConsumerService` processes the event.
5. `ProcessIdentityUserSyncedMessageHandler` creates the wallet if missing and writes an inbox row.

## Retry / DLT Rules

### Identity outbox

- publish failure before max retry:
  - `RetryCount` increases
  - `NextRetryAt` is scheduled
- publish failure after max retry:
  - payload is published to `identity.user-provisioned.dlt`
- DLT publish failure:
  - row stays pending
  - dispatcher retries later

### Wallet consumer

- business failure before max retry:
  - message is published to `identity.user-provisioned.retry`
  - original offset is committed only after retry publish succeeds
- business failure after max retry:
  - message is published to `identity.user-provisioned.dlt`
  - original offset is committed only after DLT publish succeeds
- retry/DLT publish failure:
  - original offset is not committed
  - Kafka redelivers later

## Operational Checks

### Check app containers

```bash
docker compose ps
docker logs platform-identity-api --tail 200
docker logs platform-wallet-api --tail 200
docker logs platform-kafka --tail 200
```

### Check outbox backlog

```sql
select id, message_type, retry_count, next_retry_at, processed_at, error, created_at
from "OutboxMessages"
order by "CreatedAt" desc
limit 50;
```

### Check stuck outbox rows that are ready to retry now

```sql
select id, message_type, retry_count, next_retry_at, processed_at, error, created_at
from "OutboxMessages"
where "ProcessedAt" is null
  and ("NextRetryAt" is null or "NextRetryAt" <= now() at time zone 'utc')
order by "CreatedAt" asc
limit 50;
```

### Check inbox dedupe entries

```sql
select id, message_id, consumer_group, message_type, processed_at, created_at
from "InboxMessages"
order by "CreatedAt" desc
limit 50;
```

### Check whether a specific message id was already consumed

```sql
select id, message_id, consumer_group, processed_at, created_at
from "InboxMessages"
where "MessageId" = '<message-id>';
```

## DLT Triage

When `identity.user-provisioned.dlt` receives messages:

1. Inspect the envelope metadata:
   - `Payload`
   - `RetryCount`
   - `LastError`
   - `OccurredAtUtc`
   - `FailedAtUtc`
2. Identify the failure side:
   - producer/outbox path
   - wallet consumer path
3. Fix the root cause first.
4. Replay only after the cause is understood.

## Replay Guidance

### Replay from producer-side DLT

Use the `Payload` field from the dead-letter envelope and republish it to:

- `identity.user-provisioned`

Prefer preserving the original `MessageId` unless you explicitly need to bypass inbox dedupe.

### Replay from consumer-side retry/DLT

If the original `MessageId` already exists in `InboxMessages`, replaying the same payload will be skipped by design.

Use one of these approaches:

1. Preserve `MessageId`
   - when the message was never successfully processed
2. Use a new `MessageId`
   - only when you intentionally want to force reprocessing
3. Delete the matching inbox row
   - only after manual verification
   - use carefully because this removes dedupe protection for that message/group pair

## Retry and Cleanup Knobs

### Identity outbox

- `Messaging:IdentityUserSynced:MaxRetryCount`
- `Messaging:Outbox:DispatchIntervalSeconds`
- `Messaging:Outbox:BaseRetryDelaySeconds`
- `Messaging:Outbox:MaxRetryDelaySeconds`
- `Messaging:Outbox:CleanupIntervalMinutes`
- `Messaging:Outbox:ProcessedRetentionDays`
- `Messaging:Outbox:CleanupBatchSize`

### Wallet inbox

- `Messaging:IdentityUserSynced:MaxRetryCount`
- `Messaging:Inbox:CleanupIntervalMinutes`
- `Messaging:Inbox:ProcessedRetentionDays`
- `Messaging:Inbox:CleanupBatchSize`

## Smoke Test After Recovery

1. trigger one fresh `/sync` flow
2. confirm one new outbox row appears then becomes `ProcessedAt != null`
3. confirm `Wallet` logs show the message was handled
4. confirm one inbox row exists for the new `MessageId`
5. confirm only one wallet exists for that user

## Test Coverage Map

Current component-style E2E coverage in the repo:

- `Platform.Identity.API.Tests/Infrastructure/Outbox/OutboxFlowTests.cs`
  - outbox row creation
  - successful dispatch
  - retry scheduling
  - DLT publish after max retry
- `Platform.Wallet.API.Tests/Application/Features/Wallets/Commands/ProcessIdentityUserSyncedMessage/ProcessIdentityUserSyncedMessageFlowTests.cs`
  - duplicate message dedupe
  - one wallet for repeated delivery
  - inbox entries tracked per consumer group

## Cleanup Expectations

### Outbox

- processed rows are cleaned by `OutboxCleanupService`
- retryable rows are retained until processed or dead-lettered

### Inbox

- old dedupe rows are cleaned by `InboxCleanupService`
- retention window must stay long enough to cover realistic replay/redelivery windows

## Safe Recovery Sequence

1. Confirm Kafka is healthy.
2. Confirm `Identity` and `Wallet` apps are healthy.
3. Check whether failures are accumulating in:
   - `OutboxMessages`
   - retry topic
   - DLT topic
4. Fix the root cause.
5. Replay messages carefully.
6. Verify:
   - outbox rows are processed
   - inbox rows exist exactly once
   - wallet count stays correct

## Do Not Do This Blindly

- do not delete all outbox rows
- do not delete all inbox rows
- do not republish DLT payloads in bulk without checking `MessageId` semantics
- do not bypass retry/DLT by hot-patching offsets unless you accept message loss
