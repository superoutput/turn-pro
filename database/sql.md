How can we categorize SQL command `SELECT`, `INSERT`, `DELETE`, `UPDATE`, `CREATE`, `ALTER`, `DROP`? Why can only `SELECT` operation work when storage is full?

Great question — understanding **how SQL commands are categorized** and how they behave under **full disk storage** conditions helps you design and troubleshoot PostgreSQL (or any RDBMS) more effectively.

---

## 🧱 Categorizing SQL Commands

SQL commands are typically categorized into the following groups:

| Category                      | Examples                              | Purpose                                                               |
| ----------------------------- | ------------------------------------- | --------------------------------------------------------------------- |
| **DQL** (Query)               | `SELECT`                              | Retrieve data without modifying it.                                   |
| **DML** (Data Manipulation)   | `INSERT`, `UPDATE`, `DELETE`, `MERGE` | Modify data in tables (but not structure).                            |
| **DDL** (Data Definition)     | `CREATE`, `ALTER`, `DROP`             | Define or change database structure (tables, indexes, schemas, etc.). |
| **DCL** (Data Control)        | `GRANT`, `REVOKE`                     | Control access and permissions.                                       |
| **TCL** (Transaction Control) | `BEGIN`, `COMMIT`, `ROLLBACK`         | Manage transactions.                                                  |

---

## 💾 Why Can Only `SELECT` Work When Storage Is Full?

When your disk is full, most SQL commands will **fail** because they try to write to disk.

| Command  | Writes to disk?         | Will it work on full disk? |
| -------- | ----------------------- | -------------------------- |
| `SELECT` | ❌ No                    | ✅ Yes (usually)            |
| `DELETE` | ✅ Yes                   | ❌ No                       |
| `UPDATE` | ✅ Yes                   | ❌ No                       |
| `INSERT` | ✅ Yes                   | ❌ No                       |
| `DROP`   | ✅ Yes (updates catalog) | ❌ No                       |

---

### 🔍 Why `SELECT` Works:

* `SELECT` **only reads** from disk — it doesn’t modify data, metadata, or WAL (Write-Ahead Log).
* PostgreSQL doesn’t need to allocate space for `SELECT` operations (aside from temporary memory for sorting or joins, but that’s managed in RAM or `temp` space).

> 🟢 So as long as your **temporary space isn't full** (like `/tmp` or `pg_temp`), a `SELECT` can still execute — even when the main disk is at 100%.

---

### ❌ Why Others Fail on Full Disk:

| Command     | Reason for Failing on Full Disk                             |
| ----------- | ----------------------------------------------------------- |
| `INSERT`    | Needs space to write the new row and WAL.                   |
| `UPDATE`    | Actually inserts a new version of the row + WAL write.      |
| `DELETE`    | Marks the row as deleted (new version) + WAL.               |
| `DROP`      | Updates system catalogs + WAL.                              |
| All DML/DDL | All modify the database and require WAL + data file writes. |

> PostgreSQL **requires WAL (Write-Ahead Logging)** for durability and crash recovery, and that log needs disk space.

---

## 🛠 What You Can Do on Full Disk

If you're out of disk space, try:

1. **Delete old logs** or `pg_wal` archive files (if safe and backed up).
2. **Move tables to a different tablespace** (on another disk).
3. **Add disk space** or mount more storage.
4. **Stop replication slots** that are holding WAL.
5. **Truncate large log/history tables** (after confirming safe to do so).

---

