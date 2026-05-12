# Part 3: Prompt Preparation
## Selected Pull Request
### Repository
aiokafka

### PR Title
Addes `seek_to_beginning` and  `seek_to_end` API

### PR Reference
PR: #193

---

# 3.1.2 Repository Context
`aiokafka` is an asynchronous Python client library for Apache Kafka built on top of Python’s `asyncio` framework. The repository provides high-level producer and consumer implementations that allow Python applications to communicate with Kafka brokers using non-blocking asynchronous operations. The library is mainly designed for distributed systems, real-time event streaming, and event-driven microservice architectures where high-throughput asynchronous messaging is required.

The repository supports Kafka features such as consumer groups, offset management, partition assignment, transactional messaging, metadata coordination, and asynchronous fetch handling. Developers use `aiokafka` to build scalable systems that process streams of events without blocking application execution. The asynchronous design allows applications to efficiently handle concurrent message production and consumption workloads.The primary users of the repository are backend developers, distributed systems engineers, and organizations building real-time streaming platforms. Common use cases include log processing systems, analytics pipelines, event-driven APIs, message queue systems, monitoring platforms, and microservice communication layers.One of the important responsibilities of Kafka consumers is offset management, since offsets determine which messages are consumed and from where message replay should begin. The repository therefore provides APIs for manually controlling consumer positions and fetch behavior. This pull request focuses on improving that consumer offset management functionality by introducing dedicated APIs for resetting offsets to the beginning or end of assigned partitions.

---

# 3.1.2 Pull Request Description
This pull request introduces two new consumer APIs: `seek_to_beginning()` and `seek_to_end()`. These methods allow Kafka consumers to directly move their fetch position to either the earliest available offset or the latest available offset for assigned partitions. Before this update, developers had to manually use the generic `seek()` method with explicit offset values, making offset reset operations more difficult and less intuitive.The new APIs simplify common Kafka consumption workflows such as replaying historical messages, restarting processing from the beginning of a topic, or skipping directly to the latest incoming records. The implementation also improves validation and partition safety checks to ensure that seek operations only execute on partitions currently assigned to the consumer.Previously, invalid partition operations could result in inconsistent behavior or unclear failures. The new implementation introduces proper validation logic and raises `IllegalStateError` when unassigned partitions are used. It also validates that provided arguments are valid `TopicPartition` objects and raises `TypeError` for invalid inputs.The pull request additionally improves asynchronous partition coordination by ensuring that partition assignments are fully established before executing seek operations. New regression tests were added to verify offset movement behavior, assignment validation, and error handling scenarios for both newly added APIs.

---

# 3.1.3 Acceptance Criteria
1. When `seek_to_beginning()` is called, the consumer should move to the earliest available offset for the assigned partition.

2. When `seek_to_end()` is called, the consumer should move to the latest available offset for the assigned partition.

3. The implementation should raise `IllegalStateError` when seek operations are executed on unassigned partitions.

4. The implementation should raise `TypeError` when invalid partition arguments are provided instead of `TopicPartition` objects.

5. The consumer should verify partition assignments before executing any seek operation.

6. Fetch positions should update correctly after offset reset operations.

7. Existing consumer workflows and asynchronous fetch behavior should continue functioning without regression.

8. Unit tests should validate beginning offset resets, end offset resets, invalid partitions, and assignment validation scenarios.

---

# 3.1.4 Edge Cases

### 1. Invalid Partition Type
The implementation should correctly handle situations where non-`TopicPartition` objects are passed into seek APIs and raise a `TypeError` instead of failing silently.

### 2. Unassigned Partition Access
If a consumer attempts to seek on a partition that is not currently assigned, the implementation should raise `IllegalStateError` and prevent inconsistent consumer state updates.

### 3. Empty Partition Assignment
The implementation should safely handle cases where seek operations are triggered before any partitions are assigned to the consumer.

### 4. Concurrent Asynchronus Operations
Seek operations occurring during active fetch cycles or rebalance events should not corrupt fetch positions or consumer assignment state.

### 5. Offset Reset During Consumer Rebalancing
The implementation should ensure that offset resets do not execute while partition ownership is unstable during rebalance operations.

---

# 3.1.5 Initial Prompt
Implement two new asynchronous consumer APIs in the `aiokafka` repository: `seek_to_beginning()` and `seek_to_end()` for Kafka consumers. The repository is an asyncio-based Kafka client library that is used for asynchronous event streaming and distributed messaging systems. The new APIs should improve offset management by letting consumers move directly to the earliest or latest available offsets for assigned partitions. The implementation should extend the current consumer offset management functionality inside `consumer.py`. Both APIs must support optional `TopicPartition` arguments. If no partitions are provided, the operation should apply to all currently assigned partitions. Before any seek operation runs, the implementation must ensure that partitions are assigned to the consumer. Add asynchronous assignment validation through the group coordinator layer to confirm partition ownership is fully established before updating fetch positions. The implementation must check all input arguments. If invalid partition types are passed, the APIs must raise `TypeError`. If operations are attempted on unassigned partitions, the implementation should raise `IllegalStateError`. The solution should prevent silent failures or inconsistent consumer states. Offset repositioning should use Kafka offset reset strategies like the following: `OffsetResetStrategy.EARLIEST` for `seek_to_beginning()`, `OffsetResetStrategy.LATEST` for `seek_to_end()` After marking partitions for offset reset, fetch positions should be updated asynchronously to ensure correct consumer behavior during the next fetch operations. Update `group_coordinator.py` to add the partition assignment verification needed for asynchronous seek operations. Add any required error definitions inside `errors.py`. The implementation must include unit tests that validate: - seeking to earliest offsets - seeking to latest offsets - handling of invalid partitions - behavior with unassigned partitions - asynchronous assignment validation - consumer behavior after offset resets