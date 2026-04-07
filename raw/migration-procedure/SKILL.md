---
name: migration-procedure
description: 事前件数検証とドライランによる安全なデータ移行手順を定義する。データベースまたはデータ移行の実行時に発動。
---

# Migration Procedure

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Steps

1. MUST create an API to retrieve the number of data records subject to migration
2. MUST create an API to perform the migration and execute a Dry Run
3. IF pre-fetched data count == Dry Run changed count -> proceed to step 4
   ELSE -> MUST fix and re-run
4. MUST execute the migration, then delete the API
