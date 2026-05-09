# Database Sharding Strategies

## What is Sharding?

As a system grows from a few thousand users to millions, a single database instance inevitably becomes a bottleneck. Sharding is the architectural process of breaking up a large dataset into smaller, faster, and more manageable parts called shards.

Unlike Vertical Scaling (adding more CPU/RAM), Sharding is a form of Horizontal Scaling, allowing us to distribute data across an array of low-cost server nodes.

<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/3daec342-a29f-46ef-a551-0fec97c46dab" />


---

# Why Sharding is Needed

As systems grow, a single database eventually becomes a bottleneck due to:

* CPU limits
* Memory limits
* Disk I/O limits
* Network throughput
* Write throughput

Sharding helps achieve:

✅ Horizontal scalability
✅ Better throughput
✅ Reduced hotspots
✅ Improved performance
✅ Better fault isolation

---

# Types of Sharding

---

# 1. Horizontal Sharding

## Definition

We keep the schema identical but split the Rows.

The Strategy: Store rows 1-1M on Shard A, 1M-2M on Shard B.

Best For: Handling massive datasets that exceed the storage or I/O capacity of a single machine.

Example:

| StoreId Range | Shard   |
| ------------- | ------- |
| 1–1000        | Shard A |
| 1001–2000     | Shard B |

---

## Example

```text
StoreId = 1500
→ Route to Shard B
```

---

## Best Use Cases

* E-commerce platforms
* Food delivery systems
* Banking systems
* SaaS applications
* Social media platforms

---

## Advantages

✅ Easy horizontal scaling
✅ Reduced database load
✅ Better write throughput

---

## Challenges

❌ Cross-shard joins become difficult
❌ Rebalancing shards is complex
❌ Hot partitions possible

---

# 2. Vertical Sharding

## Definition

We divide the database based on Features or Domains.

The Strategy: Tables related to "Users" go to Database A; tables for "Orders" go to Database B.

Best For: Isolating noisy neighbors. If the Loyalty service crashes, the Order service remains unaffected.

Example:

```text
User Database
Order Database
Inventory Database
Analytics Database
```

---

## Best Use Cases

* Microservices architecture
* Domain-driven systems
* Independent service scaling

---

## Real Example

```text
Menu Service DB
Store Service DB
Order Service DB
```

---

## Advantages

✅ Clear domain ownership
✅ Independent scaling
✅ Better service isolation

---

## Challenges

❌ Distributed transactions
❌ Complex orchestration between services

---

# 🛠️ Sharding Strategies & Types

# 1. Hash-Based Sharding

## Definition

A hash function determines which shard stores the data.

Pros: Prevents "Hotspots" by distributing data perfectly evenly.

Cons: Resharding (adding new nodes) is extremely difficult because the math changes for every record.

Example:

```text
hash(storeId) % 4
```

---

## Example Distribution

| StoreId | Shard   |
| ------- | ------- |
| 101     | Shard 1 |
| 102     | Shard 3 |
| 103     | Shard 2 |

---

## Best Use Cases

* Large distributed systems
* Even traffic distribution
* High-scale workloads

---

## Common Systems Using This

* CosmosDB
* Cassandra
* DynamoDB
* Kafka partitioning

---

## Advantages

✅ Even load distribution
✅ Prevents hotspots
✅ Good scalability

---

## Challenges

❌ Range queries become difficult
❌ Resharding complexity

---

# 2. Range-Based Sharding

## Definition

Data is partitioned based on ranges.

Pros: Excellent for range queries (e.g., "Show me all orders from the last 30 days").

Cons: Leads to Database Hotspots. If everyone is ordering in December, Shard 4 will be overloaded while Shards 1-3 sit idle.

Examples:

```text
A-M users → Shard A
N-Z users → Shard B
```

Or:

```text
2025 Logs → Shard A
2026 Logs → Shard B
```

---

## Best Use Cases

* Time-series data
* Logging systems
* Analytics platforms
* Financial systems

---

## Advantages

✅ Efficient range queries
✅ Easy time-based partitioning

---

## Challenges

❌ Hotspot risk
❌ Uneven shard growth

Example:
Newest data shard receives most traffic.

---

# 3. Geo Sharding

## Definition

Data is partitioned by geographical region.

Pros: Massive reduction in Network Latency and compliance with data residency laws (GDPR/CCPA).

Cons: Complex to manage for users who travel between regions.

Example:

```text
India Users → India DB
US Users → US DB
Europe Users → EU DB
```

---

## Best Use Cases

* Global SaaS systems
* Low-latency applications
* Compliance/data residency systems

---

## Advantages

✅ Lower latency
✅ Regulatory compliance
✅ Better user experience

---

## Challenges

❌ Cross-region querying complexity
❌ Data synchronization challenges

---

# 4. Directory-Based Sharding

## Definition

A lookup service determines which shard stores the data.

Pros: Maximum flexibility. You can move a specific high-value "VIP" customer to their own dedicated hardware without changing the application logic.

Cons: The Lookup Table becomes a single point of failure and a performance bottleneck.

Example:

```text
StoreId → Shard Mapping Table
```

---

## Example

```text
Store 101 → Shard A
Store 102 → Shard C
```

---

## Best Use Cases

* Dynamic shard allocation
* Flexible routing systems

---

## Advantages

✅ Flexible shard placement
✅ Easier rebalancing

---

## Challenges

❌ Requires central routing metadata
❌ Routing layer becomes critical dependency

---
# Choosing the Right Sharding Strategy

<img width="515" height="77" alt="image" src="https://github.com/user-attachments/assets/9278e872-466b-4492-9d09-a79116629491" />


| Strategy        | Best For                       |
| --------------- | ------------------------------ |
| Horizontal      | Large transactional systems    |
| Vertical        | Microservices/domain isolation |
| Hash-Based      | Even distribution/high scale   |
| Range-Based     | Time-series/range queries      |
| Geo Sharding    | Global applications            |
| Directory-Based | Dynamic shard management       |

---

# CosmosDB and Sharding

CosmosDB internally uses partitioning/sharding.

The partition key is extremely important.

---

# Good Partition Key Example

```text
storeId
```

Why?

✅ Queries localized to one partition
✅ Better RU efficiency
✅ Better scalability

---

# Bad Partition Key Example

```text
country = India
```

Problem:

All India traffic may hit the same partition, causing hotspots.

---

# Sharding vs Partitioning

| Concept      | Meaning                            |
| ------------ | ---------------------------------- |
| Partitioning | Logical data split                 |
| Sharding     | Physical distribution across nodes |

In practice, these terms are often used interchangeably.

---

# Real-World Architecture Patterns

Most large-scale systems combine multiple strategies:

Examples:

```text
Geo + Hash
Vertical + Hash
Range + Replication
```

---

# Key System Design Insight

Good sharding is fundamentally about:

```text
Balancing load while keeping queries localized.
```

Poor shard design can cause:

* Hot partitions
* Expensive cross-shard queries
* Scalability bottlenecks

Good shard design enables:

* Massive scalability
* Better throughput
* Lower latency
* Fault isolation

---

# Practical Recommendation for Restaurant/Menu Systems

Recommended partition key:

```text
storeId
```

Why?

* Most queries are store-centric
* Menus accessed per store
* Excellent partition locality
* Better cache efficiency
* Predictable scaling

Avoid:

* Country-based partitioning
* Highly skewed traffic keys
* Random unstable keys

---

# Final Takeaway

There is no universally "best" sharding strategy.

The correct strategy depends on:

* Query patterns
* Traffic distribution
* Growth expectations
* Consistency requirements
* Geographic requirements
* Operational complexity

The best architectures choose sharding based on actual access patterns, not theory alone.
