# Architectural Decision Record: Utilization of Kafka Headers for Efficient Event Filtering

## Context and Problem Statement We operate a Payments Services Event Bus utilizing Kafka, where multiple producers and consumers interact with the system. Each consumer typically requires only a subset of the total events published. However, due to the absence of an efficient filtering mechanism, consumers are forced to deserialize all incoming data to identify relevant events. This process necessitates additional server capacity to handle the large volume of events, even though only a fraction is needed by each consumer. This inefficiency leads to several issues: 
- **Consumer Frustration:** Consumers must process a vast amount of irrelevant data, which is time-consuming and resource-intensive.
-  **Increased Platform Costs:** The need for additional server capacity and resources to handle the deserialization and filtering process significantly drives up costs.
- **Network Latencies:** The large volume of data processed increases network latency, affecting overall system performance. The goal of this Architectural Decision Record (ADR) is to address these inefficiencies by implementing Kafka headers to enable more efficient event filtering. This approach aims to reduce the resource burden on consumers, lower platform costs, and improve system performance.


## Decision
Implement Kafka headers to include metadata that will allow consumers to filter relevant events without needing to parse the entire event payload.

## Alternatives Considered
1. **Continue with current approach:** Consumers parse and deserialize each event to filter relevant ones.
2. **Implement message filtering at the producer level:** Producers send events only to specific consumer topics.
3. **Utilize Kafka headers for metadata filtering:** Attach headers to events to allow consumers to filter based on header information.

## Consequences
- **Performance Improvement:** Consumers can filter events based on header metadata, reducing the need to deserialize non-relevant events.
- **Scalability:** The system will handle increased event volumes more efficiently.
- **Complexity:** Slight increase in producer complexity to include appropriate headers.
- **Flexibility:** Easier to add new filtering criteria in the future by updating headers.

## Pros of Including Kafka Headers
- **Improved Efficiency:** Consumers process fewer irrelevant events.
- **Reduced Latency:** Faster event processing for consumers.
- **Lower Resource Utilization:** Reduced CPU and memory usage for consumers.
- **Enhanced Maintainability:** Simplifies the consumer logic by avoiding full deserialization for filtering.

## Cons of Including Kafka Headers
- **Increased Complexity for Producers:** Producers need to manage and attach relevant headers to each event.
- **Potential for Inconsistent Headers:** Risk of incorrect or inconsistent header information if not managed properly.
- **Overhead in Message Size:** Headers add extra bytes to each message, potentially increasing network and storage usage.
- **Continued Coupling:** Whether using headers or message content, consumers need to understand the structure of the filtering criteria, maintaining a level of coupling to the message schema.
- **Complex Consumer Logic:** Consumers must implement logic to read and filter based on headers, which could still be complex, although more efficient than deserialization.

## Event Processing Information

| Aspect                            | Value            |
|-----------------------------------|------------------|
| Event Size                        | 8 to 10 KB       |
| Total Events                      | 1,000,000        |
| Consumers                         | 4                |
| Relevant Events per Consumer      | 100,000 (10%)    |
| Deserialization Time per Event    | 0.5 milliseconds |
| Filtering Time per Event          | 0.05 milliseconds|

## Performance Comparison

| Scenario                | Total Deserialization Time (seconds) | Total Filtering Time (seconds) | Total Processing Time (seconds) | Performance Improvement |
|-------------------------|--------------------------------------|-------------------------------|---------------------------------|-------------------------|
| Without Kafka Headers   | 2000                                 | 200                           | 2200                            | N/A                     |
| With Kafka Headers      | 200                                  | 20                            | 220                             | 10x                     |

## Performance Visualization

![Performance Comparison](path/to/your/image.png)

This visualization compares the performance of the scenarios with and without Kafka headers, highlighting the significant performance improvement when using Kafka headers.

