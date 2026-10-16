# SQLite Batch Transfer Benchmark

## 1. Project Overview

This project benchmarks SQLite database performance using a simulated bank-transfer system. It explores how Write-Ahead Logging (WAL) and batch processing improve transaction throughput.

The project tests different batch sizes and measures how many successful transfers SQLite can process per second.

## 2. Objectives

- Understand SQLite WAL mode.
- Analyze the impact of transaction batching.
- Measure transaction throughput.
- Verify account balance integrity.
- Identify a suitable batch size for better performance.

## 3. Technologies Used

- Python: Benchmark implementation.
- SQLite: Lightweight database.
- WAL: Write-Ahead Logging for database journaling.
- Pandas/Matplotlib (optional): Data analysis and visualization.

## 4. How the Project Works

1. Create a SQLite database.
2. Create an `accounts` table containing 10,000 accounts.
3. Initialize each account with 100,000 cents.
4. Generate 50,000 random transfer attempts.
5. Execute transfers using different batch sizes.
6. Measure execution time and successful transfers per second.
7. Verify account count, total balance, and non-negative balances.
8. Compare the performance results.

## 5. Database Configuration

The benchmark uses the following settings:

```
conn.execute("PRAGMA journal_mode=WAL;")conn.execute("PRAGMA synchronous=NORMAL;")
```

- WAL mode: Records database changes in a write-ahead log before checkpointing them into the main database.
- Synchronous NORMAL: Reduces synchronization overhead while retaining some durability protections. Recent committed transactions may be lost after certain operating-system crashes or power failures.

## 6. Batching Explained

Batching means executing multiple transfers inside one database transaction before committing.

For example:

- Batch size 1: One transfer per transaction.
- Batch size 100: One hundred transfers per transaction.
- Batch size 1,000: One thousand transfers per transaction.

Larger batches reduce commit overhead per transfer, potentially improving performance.

## 7. Experimental Configuration

| Parameter                   | Value                        |
| --------------------------- | ---------------------------- |
| Total accounts              | 10,000                       |
| Initial balance per account | 100,000 cents                |
| Transfer attempts           | 50,000                       |
| Journal mode                | WAL                          |
| Synchronous mode            | NORMAL                       |
| Batch sizes                 | 1, 100, 1,000, 5,000, 10,000 |

## 8. Benchmark Results

The following results were measured during one benchmark run.

| Batch size | Successful transfers | Duration (seconds) | Throughput (transfers/sec) | Integrity |
| ---------- | -------------------- | ------------------ | -------------------------- | --------- |
| 1          | 50,000               | 4.428              | 11,293                     | PASS      |
| 100        | 50,000               | 0.696              | 71,808                     | PASS      |
| 1,000      | 50,000               | 0.409              | 122,309                    | PASS      |
| 5,000      | 50,000               | 0.396              | 126,225                    | PASS      |
| 10,000     | 50,000               | 0.387              | 129,279                    | PASS      |

Note: Results depend on the hardware, operating system, SQLite version, and workload.

## 9. Results Analysis

### Batch size 1

The benchmark achieved 11,293 transfers per second. Each transfer uses its own transaction, resulting in frequent commits and additional overhead.

### Batch size 100

Throughput increased to 71,808 transfers per second because multiple transfers share a transaction.

### Batch size 1,000

Throughput reached 122,309 transfers per second, approximately 10.8 times faster than batch size 1.

### Batch size 5,000

Throughput increased slightly to 126,225 transfers per second, showing diminishing performance gains.

### Batch size 10,000

The highest measured throughput was 129,279 transfers per second. However, this was only a small improvement over batch size 1,000.

## 10. Integrity Verification

The benchmark checks three conditions:

- Account count: The database must contain 10,000 accounts.
- Total balance: The combined account balance must remain unchanged.
- Minimum balance: No account should have a negative balance.

All tests passed these checks.

These checks provide basic evidence of correctness but do not prove complete production reliability or crash recovery.

## 11. Recommended Configuration

For this benchmark, the recommended starting configuration is:

- Journal mode: `WAL`
- Synchronous mode: `NORMAL`
- Batch size: `1,000`

This configuration achieved 122,309 successful transfers per second, approximately 94.6% of the highest measured throughput.

It offers a useful balance between performance and transaction size for this particular workload. Real applications should determine their batch size through repeated testing.

## 12. Limitations

- Results are based on one run per batch size.
- The benchmark uses one SQLite connection and does not test concurrent writers.
- The workload is simulated rather than a real banking workload.
- Performance varies across machines and storage environments.
- The benchmark does not test crash recovery or power-loss scenarios.
- `synchronous=NORMAL` has durability trade-offs that may not suit financial applications.

## 13. Future Improvements

Potential improvements include:

- Run each test multiple times and report median throughput.
- Compare `WAL/FULL`, `WAL/NORMAL`, and `WAL/OFF`.
- Visualize throughput and latency using Matplotlib.
- Measure transaction latency percentiles.
- Test different numbers of accounts and transfer attempts.
- Investigate concurrent readers and writers.
- Test database recovery and backup procedures.

## 14. Conclusion

This project demonstrates that combining WAL with transaction batching can significantly improve SQLite throughput.

In the recorded benchmark, throughput increased from 11,293 transfers per second with batch size 1 to 129,279 transfers per second with batch size 10,000.

Batch size 1,000 is a reasonable starting point because it achieved most of the maximum measured throughput without using extremely large transactions.

The experiment highlights the importance of reducing transaction overhead while considering durability, data integrity, and application requirements.
