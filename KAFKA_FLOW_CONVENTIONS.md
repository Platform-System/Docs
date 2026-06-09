# Kafka Flow Conventions

Tai lieu nay chot pattern de mo rong Kafka trong he `Platform.*`.

Muc tieu:

- feature moi di cung 1 kieu producer/consumer
- service nao cung biet khi nao dung outbox, retry, DLT
- test duoc viet theo 1 checklist giong nhau

## Scope

Convention nay ap dung cho cac flow event-driven moi trong repo, dac biet la cac flow can:

- phat event tu DB state change
- tieu thu event va cap nhat DB ben consumer
- can retry an toan
- can idempotency

Flow tham chieu hien tai:

- `Platform.Identity.API` -> publish `IdentityUserSynced`
- `Platform.Wallet.API` -> consume event va tao vi neu user chua co vi

## Producer-side pattern

Neu event duoc phat sau khi DB state thay doi, producer phai di theo outbox.

Pattern chot:

1. `Application` tao `PreCommitEvent` hoac notification business.
2. notification handler ghi row vao `OutboxMessages`.
3. `Infrastructure/Outbox` dispatch row do ra Kafka.
4. row chi duoc mark processed sau khi publish thanh cong.

Khong duoc:

- publish Kafka truc tiep trong command handler sau khi vua sua DB
- de business flow phu thuoc runtime vao service downstream

File dat cho producer:

- `Application/Features/<Feature>/Notifications`
- `Application/Abstractions/Messaging`
- `Infrastructure/Outbox`
- event contract trong `Platform.Contracts`

## Consumer-side pattern

Consumer la mot async entrypoint, dat o `Consumers`.

Pattern chot:

1. consumer deserialize message
2. validate contract can ban
3. map sang command/query trong `Application`
4. `Application` handler xu ly business rule
5. consumer commit offset sau khi:
   - xu ly thanh cong
   - hoac retry/DLT handoff thanh cong

File dat cho consumer:

- `Consumers/<Transport>`
- `Application/Features/<Feature>/Commands/Process<EventName>`
- `Application/Abstractions/Messaging` neu can inbox/store contract

## Idempotency rule

Moi consumer co side effect tren DB phai co idempotency ledger.

Pattern chot:

- dung `InboxMessages` de track `MessageId + ConsumerGroup`
- handler phai check `ExistsAsync(...)` truoc khi ap side effect
- neu side effect da ap roi, handler return success

Khong duoc:

- dua vao Kafka offset de dedupe business
- bo qua `MessageId`

## Retry / DLT rule

### Producer side

- outbox row giu:
  - `RetryCount`
  - `NextRetryAt`
  - `Error`
- publish fail truoc max retry:
  - tang `RetryCount`
  - schedule `NextRetryAt`
- publish fail sau max retry:
  - publish DLT
- neu DLT fail:
  - row van pending de dispatch lai sau

### Consumer side

- retry metadata phai luu rieng khoi offset Kafka
- retry row can co identity ro rang theo:
  - `MessageId`
  - `ConsumerGroup`
- business fail truoc max retry:
  - update retry row
- business fail sau max retry:
  - publish DLT
- retry/DLT publish fail:
  - khong commit offset neu dang xu ly live Kafka message

## Shared base phai uu tien dung

Neu flow moi giong pattern hien tai, uu tien dung cac thanh phan da co trong `Platform.Messaging`:

- `KafkaConsumerWithRetryBase<TMessage>`
- `KafkaOutboxDispatcherBase<TClaimedMessage>`
- `KafkaRetryEnvelope<TPayload>`
- `KafkaDeadLetterEnvelope<TPayload>`
- `KafkaMessageContext<TMessage>`
- `KafkaMessageProcessResult`
- `RelationalLeaseClaimHelper`
- `EntityMutationHelper`

Chi nen viet implementation rieng khi:

- transport khac pattern hien tai
- khong can retry/outbox
- co business rule dac thu ma base khong cover duoc

## Topic naming

Tam chot naming nhu sau:

- main topic:
  - `<bounded-context>.<event-name>`
- dead-letter topic:
  - `<main-topic>.dlt`

Vi du:

- `identity.user-provisioned`
- `identity.user-provisioned.dlt`

Neu sau nay co retry topic thuc thu tren broker thi uu tien:

- `<main-topic>.retry`

## Consumer group naming

Consumer group nen noi ro service + muc dich.

Format khuyen nghi:

- `<service>-<use-case>`

Vi du:

- `wallet-api-user-provisioned`

## Contract rule

Event contract dat trong `Platform.Contracts`.

Contract toi thieu nen co:

- `MessageId`
- business identity chinh
- `OccurredAt`

Khong nen:

- nhung dependency type tu service implementation
- dua business logic vao contract object

## Test checklist bat buoc

Moi flow Kafka moi nen co du 4 tang test nay:

1. `Application` business test
   - handler dung business rule
   - duplicate message khong tao side effect thua

2. producer outbox flow test
   - business handler / notification ghi outbox row
   - dispatcher publish thanh cong
   - retry / DLT producer side

3. consumer flow test
   - consume event -> handler -> DB side effect
   - persisted retry xu ly thanh cong
   - duplicate replay van idempotent

4. shared base test
   - `Platform.Messaging.Tests`
   - cover retry loop, outbox loop, helper behavior

## Migration checklist cho flow moi

Khi chuyen mot flow sync sang Kafka, di theo thu tu nay:

1. chot event contract trong `Platform.Contracts`
2. tao producer-side notification/outbox writer
3. tao outbox dispatcher hoac map vao base dispatcher
4. tao consumer service
5. tao application handler `Process<Event>`
6. them inbox/idempotency
7. them retry/DLT
8. them test theo checklist
9. chi xoa call sync cu khi da confirm khong con caller can no

## Current reference files

Neu can copy dung pattern hien tai, doc truoc:

- `Platform.Identity.API/Application/Features/Users/Commands/SyncUserSession/SyncUserSessionHandler.cs`
- `Platform.Identity.API/Application/Features/Users/Notifications/IdentityUserSyncedOutboxNotificationHandler.cs`
- `Platform.Identity.API/Infrastructure/Outbox/OutboxDispatcher.cs`
- `Platform.Wallet.API/Consumers/Kafka/IdentityUserSyncedConsumerService.cs`
- `Platform.Wallet.API/Application/Features/Wallets/Commands/ProcessIdentityUserSyncedMessage/ProcessIdentityUserSyncedMessageHandler.cs`
- `Platform.Messaging/Hosting/KafkaConsumerWithRetryBase.cs`
- `Platform.Messaging/Hosting/KafkaOutboxDispatcherBase.cs`

## Related docs

- `Platform.Docs/Identity/KAFKA_MESSAGING_RUNBOOK.md`
- `Platform.Docs/WALLET_BACKEND_FLOW.md`
- `Platform.Docs/BACKEND_SERVICE_STRUCTURE.md`
