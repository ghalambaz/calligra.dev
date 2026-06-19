<!--
  HASHNODE PUBLISH INSTRUCTIONS
  ══════════════════════════════════════════════════════
  1. Go to hashnode.com → Write → "Import article"
     OR paste content below into the Hashnode editor directly.

  2. After import / paste, fill in the UI fields:
     • Title       : Listening to the Heartbeat of Your Database: Understanding Change Data Capture (CDC)
     • Subtitle    : 
     • Slug        : listening-to-the-heartbeat-of-your-database-understanding-change-data-capture-cdc
     • Tags        : cdc, database, mysql, mongodb, postgresql, event-driven, data engineering, architecture
     • Cover image : https://calligra.dev/images/blog/listening-to-the-heartbeat-of-your-database-understanding-change-data-capture-cdc/cover.jpg

  3. Under "SEO" settings:
     • Canonical URL: https://calligra.dev/blog/listening-to-the-heartbeat-of-your-database-understanding-change-data-capture-cdc

  4. Save as draft, review, then publish.
  ══════════════════════════════════════════════════════
-->
# 

> CDC stands for **Change Data Capture**.

## What You Will Learn

- What CDC is
- The story behind it
- How it works internally
- Common use cases
- A simplified implementation example

---

# What Is CDC?

Change Data Capture (CDC) is a technique for capturing changes made to data and exposing those changes to external systems.

Instead of repeatedly querying a database to check whether something has changed, CDC allows applications to consume a stream of inserts, updates, and deletes as they happen.

Think of CDC as a way to listen to the heartbeat of your database.

Every time data changes, the database records that change internally. CDC makes those changes available so that other systems can react to them in real time.

---

# The Story Behind CDC

To understand CDC, it helps to understand how databases have worked for decades.

Whenever data changes, databases do not simply overwrite the old value and move on. They maintain an internal record of operations that have occurred.

These records are commonly stored in transaction logs.

Historically, every database implemented these logs differently, but the purpose was always the same:

- Record every change made to the database
- Recover from failures
- Replicate data to other nodes
- Maintain consistency

Every insert, update, and delete operation generates a log entry.

These entries are ordered and identified by a unique position or token.

The ordering is critical.

Imagine the following sequence:

1. Create User
2. Update User
3. Delete User

If a replica applied these operations in a different order, it would end up with completely different data.

Because these logs are ordered, databases can reliably replay changes and reconstruct state over time.

This is how replication works in many database systems.

A replica typically starts with an initial snapshot and then continuously applies new changes by replaying the transaction log.

---

# Where CDC Comes In

Originally, these logs were intended for the database itself.

CDC extends that idea by allowing external applications to consume those same changes.

Instead of only replicas reading database changes, your applications can read them too.

This means that whenever data changes, you can react immediately without constantly querying the database.

The underlying idea is simple:

> Expose an ordered stream of database changes that external systems can consume reliably.

---

# How CDC Works

Different databases implement CDC differently, but the concept remains the same.

### MongoDB

MongoDB provides CDC through **Change Streams**.

Each event contains a **resume token**, which allows consumers to continue reading from the last processed change after a restart or failure.

### MySQL

MySQL exposes changes through its **Binary Log (Binlog)**.

CDC tools can read the binlog and transform database operations into events.

### PostgreSQL

PostgreSQL provides CDC through **Logical Decoding** and **Logical Replication**.

These mechanisms allow applications to consume database changes in order.

Although the implementations differ, they all provide the same capability:

> An ordered stream of database changes.

---

# CDC Is Usually Pull-Based

One common misconception is that databases push changes directly to your application.

In reality, most CDC implementations are fundamentally pull-based.

The consumer requests the next available change from the database.

However, applications typically maintain a long-lived connection or cursor, making the experience feel very similar to receiving pushed events.

For example, MongoDB's `watch()` API keeps a stream open and continuously delivers new events as they become available.

From the application's perspective, it feels real-time.

---

# A Simplified CDC Example

To understand the concept, imagine that every database change is converted into a standardized event.

```php
function captureDataChange($operation, $table, $id, $before, $after)
{
    $cdcEvent = [
        "metadata" => [
            "operation" => $operation,
            "table" => $table,
            "id" => $id
        ],
        "before" => $before,
        "after" => $after
    ];

    echo "Streaming event: " . json_encode($cdcEvent) . PHP_EOL;
}

captureDataChange(
    "UPDATE",
    "users",
    42,
    ["name" => "Alice", "role" => "Developer"],
    ["name" => "Alice", "role" => "Architect"]
);
```

Output:

```json
{
  "metadata": {
    "operation": "UPDATE",
    "table": "users",
    "id": 42
  },
  "before": {
    "name": "Alice",
    "role": "Developer"
  },
  "after": {
    "name": "Alice",
    "role": "Architect"
  }
}
```

Real CDC systems work differently under the hood, but the idea is very similar:

1. Detect a database change
2. Convert it into an event
3. Publish it to interested consumers

---

# CDC vs Traditional Polling

Without CDC, applications often use polling.

For example:

```sql
SELECT *
FROM orders
WHERE updated_at > :last_seen_timestamp;
```

This approach works, but it has drawbacks:

- Additional load on the database
- Expensive queries at scale
- Possibility of missing updates
- Duplicate processing logic

CDC avoids these problems by consuming changes directly from the database's change stream.

Instead of repeatedly asking:

> "Has anything changed?"

the database tells you:

> "This is what changed."

---

# Why CDC Matters

CDC is more than a database feature.

It is an architectural building block.

Once database changes become events, many new possibilities emerge.

Common use cases include:

- Search index synchronization (Elasticsearch/OpenSearch)
- Analytics pipelines
- Event-driven architectures
- Cache invalidation
- Audit trails
- Data warehouse synchronization
- Building materialized views
- Synchronizing data across services

For example, when a product is updated in your database, a CDC consumer can automatically update:

- Search indexes
- Caches
- Reporting systems
- Recommendation engines

without requiring changes to the original application.

---

# CDC in Modern Architectures

Many organizations use CDC platforms such as Debezium to provide a unified way of consuming changes across different databases.

These platforms often publish CDC events into systems such as Apache Kafka, where multiple consumers can process the same stream independently.

This makes CDC a key building block in modern event-driven systems.

---

# Final Thoughts

At its core, CDC is a simple idea:

> Every database change is already being recorded somewhere. CDC allows external systems to listen to those changes.

Whether it is MongoDB Change Streams, MySQL Binlogs, or PostgreSQL Logical Replication, the principle remains the same:

An ordered stream of database changes that can be consumed reliably.

Once you start viewing database updates as events rather than rows being modified, CDC becomes one of the most powerful tools for building scalable and loosely coupled systems.