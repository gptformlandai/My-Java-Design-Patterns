# Map Reduce Pattern

Category: Functional and Pipeline Processing Patterns  
MAANG interview meter: Very High  
Software usage meter: High  
Repository module: [map-reduce](../../github-repo/map-reduce)

---

## How to Study This Page

Study Map Reduce as "split massive work, process pieces independently, then merge the answers."

Remember the core flow:

```text
Input chunks -> Map -> Shuffle/Group -> Reduce -> Final result
```

The important interview idea is not the syntax. It is why independent map work plus grouped reduce work allows huge datasets to be processed in parallel.

---

## 1. Technical Definition

Map Reduce is a distributed data processing pattern that transforms input records into intermediate key-value pairs, groups values by key, and reduces each group into final aggregated results.

### 30-Second Interview Answer

Map Reduce is used when a large dataset can be split into independent chunks. The map phase emits key-value pairs, the shuffle phase groups values with the same key, and the reduce phase aggregates each group. It is useful for word counts, logs, indexing, analytics, and batch jobs. The benefit is parallelism and fault isolation; the cost is shuffle overhead, batch latency, and awkwardness for iterative or low-latency workloads.

---

## 2. Layman and Easy to Understand Definition

Imagine counting votes from thousands of boxes.

- Many people count their own box independently.
- Everyone writes totals per candidate.
- Totals for the same candidate are grouped together.
- A final person adds grouped totals.

That is Map Reduce: local work first, grouping second, final aggregation last.

---

## 3. Bit by Bit Explanation of What It Is

### Problem

One machine cannot efficiently process a huge dataset when:
- input is too large for memory
- processing takes too long on one CPU
- failures should not restart the whole job
- data is naturally partitioned across machines

### Map Reduce Flow

1. Split input into partitions.
2. Run a mapper on each partition.
3. Mapper emits intermediate key-value pairs.
4. Shuffle groups all values for the same key.
5. Reducer processes one key group at a time.
6. Final output is written to storage.

### Core Participants

| Participant | Responsibility |
|---|---|
| Input split | A chunk of source data |
| Mapper | Converts records into intermediate pairs |
| Intermediate key | The grouping key |
| Shuffle | Moves and groups data by key |
| Reducer | Aggregates grouped values |
| Output store | Stores final results |

---

## 4. Java Coding Example

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MapReduceDemo {
    static Map<String, Integer> map(String line) {
        Map<String, Integer> counts = new HashMap<>();
        for (String word : line.toLowerCase().split("\\s+")) {
            counts.merge(word, 1, Integer::sum);
        }
        return counts;
    }

    static Map<String, List<Integer>> shuffle(List<Map<String, Integer>> mapped) {
        Map<String, List<Integer>> grouped = new HashMap<>();
        for (Map<String, Integer> partial : mapped) {
            for (var entry : partial.entrySet()) {
                grouped.computeIfAbsent(entry.getKey(), key -> new ArrayList<>())
                        .add(entry.getValue());
            }
        }
        return grouped;
    }

    static Map<String, Integer> reduce(Map<String, List<Integer>> grouped) {
        Map<String, Integer> totals = new HashMap<>();
        for (var entry : grouped.entrySet()) {
            totals.put(entry.getKey(),
                    entry.getValue().stream().mapToInt(Integer::intValue).sum());
        }
        return totals;
    }

    public static void main(String[] args) {
        List<Map<String, Integer>> mapped = List.of(
                map("payment failed"),
                map("payment succeeded"),
                map("payment failed"));

        System.out.println(reduce(shuffle(mapped)));
    }
}
```

### Java Block by Block

`map` counts words inside one input line.

`shuffle` groups partial counts by word.

`reduce` sums the grouped counts.

In a real distributed system, these phases run across many workers and store intermediate output durably.

---

## 5. Python Coding Example

```python
from collections import defaultdict


def mapper(line):
    counts = defaultdict(int)
    for word in line.lower().split():
        counts[word] += 1
    return counts


def shuffle(mapped_results):
    grouped = defaultdict(list)
    for partial in mapped_results:
        for word, count in partial.items():
            grouped[word].append(count)
    return grouped


def reducer(grouped):
    return {word: sum(counts) for word, counts in grouped.items()}


lines = ["payment failed", "payment succeeded", "payment failed"]
print(reducer(shuffle([mapper(line) for line in lines])))
```

---

## 6. Where It Comes Handy in Real Life

| Use case | Why it fits |
|---|---|
| Log analytics | Logs are independent records that can be grouped by service, status, or user |
| Word count/search indexing | Records emit terms, reducers aggregate by term |
| Batch reporting | Large historical datasets can be processed in partitions |
| Clickstream analytics | Events can be mapped and grouped by campaign/session |
| Data warehouse transforms | Batch ETL often needs grouping and aggregation |
| Security analytics | Events can be grouped by IP, account, or rule |

---

## 7. Advantages Over Normal Code Without Pattern

Without Map Reduce:
- one job may try to load too much data
- one failure may restart all work
- parallelization logic becomes custom and fragile
- grouping huge intermediate data is hard

With Map Reduce:
- map tasks run independently
- reducers own deterministic key groups
- failed tasks can be retried
- data movement is explicit in the shuffle stage

---

## 8. Where It Excels

It excels when:
- the dataset is very large
- the job is batch-oriented
- records can be processed independently
- results can be aggregated by key
- throughput matters more than sub-second latency
- retrying individual tasks is valuable

---

## 9. Where It Fails

It fails when:
- the workflow needs low latency
- every record depends on global mutable state
- the algorithm is highly iterative
- shuffle volume is larger than useful work
- reducers have extreme hot keys
- the result requires many chained jobs with heavy disk I/O

Use stream processing, databases, or in-memory distributed compute when the workload needs real-time updates or repeated interactive queries.

---

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Tools |
|---|---|
| Java/JVM | Hadoop MapReduce, Apache Spark, Apache Flink |
| Python | PySpark, Dask, mrjob |
| Cloud | AWS EMR, Google Dataproc, Azure HDInsight |
| Data warehouses | BigQuery, Snowflake, Redshift batch SQL |
| Storage | HDFS, S3, GCS, ADLS |

---

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Scales across many machines | Shuffle can be expensive |
| Natural fit for batch aggregation | Higher latency than streaming |
| Mapper tasks are easy to retry | Reducer skew can hurt performance |
| Clear split between local and grouped work | Not ideal for iterative algorithms |
| Works well with immutable input data | Requires serialization and intermediate storage |

---

## 12. Real-World Identification Example

Question:

> You need to process 20 TB of historical access logs each night and compute request counts by API endpoint and status code. What pattern helps?

Strong answer:

Use Map Reduce. Each mapper reads a log partition and emits keys such as `(endpoint, statusCode)` with count `1`. The shuffle groups all matching keys. Reducers sum counts per group and write daily aggregates. I would watch for hot endpoints, shuffle volume, and failed worker retries.

---

## 13. MAANG Interview Triggers

Say Map Reduce when you hear:
- huge batch dataset
- process logs at scale
- word count
- group by key across many machines
- distributed aggregation
- shuffle phase
- fault-tolerant batch processing
- Hadoop or Spark batch job

---

## 14. Common Mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Ignoring shuffle cost | Network movement can dominate the job | Combine locally before shuffle |
| Sending huge values per key | Reducers become memory bottlenecks | Emit compact intermediate records |
| Creating hot keys | One reducer gets overloaded | Salt keys or repartition |
| Using Map Reduce for real-time APIs | Batch jobs have high latency | Use streaming or online stores |
| Making reducers depend on global order | Distributed execution breaks assumptions | Design reducers around one key group |

---

## 15. Map Reduce vs Similar Patterns

| Pattern | Difference |
|---|---|
| Pipeline | Pipeline stages pass transformed data forward; Map Reduce adds grouping and reducing by key |
| Fan-Out/Fan-In | Fan-out/fan-in runs parallel tasks and merges results; Map Reduce formalizes map, shuffle, and reduce |
| Event Queue | Event queue handles asynchronous messages; Map Reduce handles batch datasets |
| Stream Processing | Stream processing handles continuous events; Map Reduce usually handles bounded batches |
| Collection Pipeline | Collection pipeline is in-process; Map Reduce is distributed and shuffle-aware |

---

## 16. Map Reduce Design Checklist

- What is the input record?
- What key-value pairs does the mapper emit?
- Can mapper work be done independently?
- What is the reducer key?
- Can local combine reduce shuffle volume?
- Are there hot keys?
- What happens if a mapper or reducer fails?
- Where are intermediate and final outputs stored?
- Is the workload batch or real-time?

---

## 17. Quick Revision Notes

- One-line summary: Map records to key-value pairs, group by key, reduce each group.
- Memory hook: "map, shuffle, reduce."
- Best for: large batch aggregation.
- Avoid when: low latency or iterative computation is required.
- Interview line: "I would emit compact key-value pairs, reduce shuffle with local combining, and handle skewed keys explicitly."

---

## 18. Mini Exercise

Design a Map Reduce job for daily payment failures:
- define the input log format
- choose the mapper output key
- decide whether to use a combiner
- define the reducer output
- identify one possible hot key

---

## 19. Source Reference in This Repo

Study these files:
- [README](../../github-repo/map-reduce/README.md)
- [Mapper.java](../../github-repo/map-reduce/src/main/java/com/iluwatar/Mapper.java)
- [Shuffler.java](../../github-repo/map-reduce/src/main/java/com/iluwatar/Shuffler.java)
- [Reducer.java](../../github-repo/map-reduce/src/main/java/com/iluwatar/Reducer.java)
- [MapReduce.java](../../github-repo/map-reduce/src/main/java/com/iluwatar/MapReduce.java)
- [Main.java](../../github-repo/map-reduce/src/main/java/com/iluwatar/Main.java)

The repo implementation maps input strings into word-count maps, shuffles counts by word, and reduces each grouped word into a total count.
