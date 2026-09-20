# mitm_collector_employee-de_ora

## Overview

`mitm_collector_employee-de_ora` is a specialized Oracle data collector for the Man-in-the-Middle (MitM) Data Aggregator system, specifically tailored for Employee data in Germany (DE) and A1 company codes.

It is responsible for fetching employee data from an Oracle source database while simultaneously joining the `temp_assignment` and `org` tables directly on the database level. This ensures that organizational data (such as cost centers, personnel areas, and supervisors) is correctly overridden dynamically for employees with active temporary assignments, feeding a single, denormalized payload per employee to the downstream layers.

## Features

- Connects to an Oracle database using `go-ora/v2`.
- Performs an inline `LEFT JOIN` between the primary configured `employee` table, `temp_assignment`, and `org` tables to securely calculate active assignments.
- Supports cursor-based incremental data fetching.
- Implements Envelope Encryption (AES-GCM) for data-at-rest.
- Communicates via Unix Domain Sockets for IPC logging and scheduler orchestration.

## Configuration

### Database Configuration & Credentials

The component expects database credentials and the encryption master key to be injected at runtime. The resolution order is:

1. **IPC Scheduler Connection (Preferred):** If invoked by the `mitm_scheduler`, the component dynamically fetches the PostgreSQL credentials and `MASTER_KEY` via a Unix Domain Socket (IPC).
2. **JSON Config (Fallback):** Setting the `MITM_DB_CONFIG_JSON` environment variable containing a JSON string with a nested `"db"` object.
3. **Direct Environment Variables (Fallback):** Setting `MITM_DB_HOST`, `MITM_DB_PORT`, `MITM_DB_USER`, `MITM_DB_PASSWORD`, `MITM_DB_NAME`, and `MASTER_KEY` directly.

Invoked by the `mitm_scheduler` with JSON arguments to configure topics and table targets.

### Scheduler Parameters (JSON Arguments)

When the `mitm_scheduler` executes this collector, it passes a JSON configuration string as the first command-line argument (`os.Args[1]`).

The following parameters are supported:

| Field                 | Type   | Required | Description                                                                                                                                       |
| :-------------------- | :----- | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------ |
| `source_name`         | string | No       | The logical name of the source system. Defaults to `ORA_EMPLOYEE`. Must match a valid entry in the `source_credentials` DB table.                 |
| `table`               | string | No       | The primary Employee table to query from (e.g., `employee`). Defaults to `employees`.                                                             |
| `cursor_column`       | string | No       | The numeric/incremental column used to track fetch progress (e.g., `id`). If set to `"none"`, the collector will fetch all rows without a cursor. |
| `topic`               | string | No       | The MitM system topic for downstream matching. Defaults to `ora.<table_name>.data`.                                                               |
| `business_key_column` | string | No       | The unique column used to hash and generate the `correlation_id` (e.g., `pernr`). Defaults to `id`.                                               |
| `db_where_in` | map | No | Optional key-value map (e.g., `{"companycode": ["A1", "DE"]}`) to dynamically add `WHERE IN (...)` conditions to the extraction query. |
