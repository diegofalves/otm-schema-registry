# OTM Schema Registry

Public, versioned registry of Oracle Transportation Management (OTM) database schema metadata.

This repository provides machine-readable JSON schema files organized by OTM release for use by SQL editors, autocomplete engines, validation tools, documentation systems, development utilities, and other tooling that needs structured knowledge of the OTM data model.

> This is an independent community project and is not affiliated with or endorsed by Oracle.

---

## Purpose

Oracle Transportation Management evolves between releases.

Tables, columns, data types, constraints, and relationships may change over time. Tools that rely on OTM database metadata therefore need to know which schema belongs to which OTM release.

This repository provides a versioned schema snapshot for each supported OTM release.

Conceptually:

```text
OTM Data Dictionary
        ↓
Schema generation
        ↓
Versioned JSON
        ↓
OTM Schema Registry
        ↓
SQL tools / autocomplete / validation
```

The registry is intentionally independent from any specific application.

Consumers discover available schema versions through `manifest.json` and then load the corresponding schema artifact.

---

## Repository Structure

```text
otm-schema-registry/
├── README.md
├── manifest.json
└── schemas/
    └── 26a/
        └── otm-schema.json
```

Each OTM release has its own directory under `schemas/`.

Example:

```text
schemas/
├── 26a/
│   └── otm-schema.json
├── 26c/
│   └── otm-schema.json
└── ...
```

Only releases that are actually available should be listed in `manifest.json`.

---

## Manifest

The root `manifest.json` is the discovery contract for consumers of the registry.

Current structure:

```json
{
  "manifestVersion": "1.0",
  "registry": "otm-schema-registry",
  "versions": [
    {
      "version": "26A",
      "path": "schemas/26a/otm-schema.json",
      "schemaVersion": "1.0",
      "status": "available"
    }
  ]
}
```

Consumers should use the manifest instead of assuming that a specific release exists.

Typical flow:

```text
GET manifest.json
        ↓
Find required OTM release
        ↓
Read schema path
        ↓
GET schemas/<release>/otm-schema.json
        ↓
Load metadata
```

This allows new OTM releases to be added without changing the integration contract.

---

## Available Releases

| OTM Release | Schema Version | Status | Path |
|---|---:|---|---|
| 26A | 1.0 | Available | `schemas/26a/otm-schema.json` |

Additional releases will be added as their schema snapshots are generated and validated.

---

## Schema Artifact

Each release contains a canonical `otm-schema.json` file.

The artifact is intended to describe OTM database metadata in a format suitable for programmatic consumption.

Depending on the available source metadata, the schema may contain information such as:

- tables;
- columns;
- Oracle data types;
- nullable attributes;
- primary keys;
- foreign keys;
- constraints;
- relationships;
- indexes;
- comments or descriptions;
- commonly used table aliases;
- schema generation metadata.

The exact JSON contract is versioned independently through the `schemaVersion` field in `manifest.json`.

---

## Example

A simplified schema entry may look conceptually like this:

```json
{
  "name": "ORDER_RELEASE",
  "columns": [
    {
      "name": "ORDER_RELEASE_GID",
      "dataType": "VARCHAR2",
      "nullable": false
    },
    {
      "name": "DOMAIN_NAME",
      "dataType": "VARCHAR2",
      "nullable": false
    }
  ]
}
```

The real schema artifact contains substantially more metadata.

Consumers should rely on the actual JSON structure rather than this simplified example.

---

## Source Metadata

Schema snapshots are generated from OTM metadata sources maintained by the project.

Where applicable, the artifact identifies its source metadata, for example:

```json
{
  "source": {
    "dictionary": "Oracle Transportation Management Data Dictionary 26A",
    "aliases": "Project-maintained OTM table alias conventions"
  }
}
```

The registry must not contain private filesystem paths, credentials, environment-specific secrets, customer-specific information, or other sensitive local metadata.

---

## Versioning Strategy

Two different concepts are versioned independently.

### OTM Release

Represents the Oracle Transportation Management release.

Examples:

```text
26A
26C
27A
```

Each release receives its own directory:

```text
schemas/26a/
schemas/26c/
schemas/27a/
```

### Schema Format Version

Represents the structure of the JSON contract itself.

Example:

```json
"schemaVersion": "1.0"
```

This allows the OTM release to change without necessarily changing the JSON format.

For example:

```text
OTM 26A → schemaVersion 1.0
OTM 26C → schemaVersion 1.0
OTM 27A → schemaVersion 1.0
```

If the JSON contract changes incompatibly in the future:

```text
schemaVersion 1.0
        ↓
schemaVersion 2.0
```

Consumers can then explicitly decide which schema format versions they support.

---

## Release Immutability

Published schema snapshots should be treated as immutable artifacts.

After a release is published and consumed by external tools, changes to that artifact should be limited to corrections that do not invalidate the published contract.

When the OTM product changes, a new release directory should normally be created instead of modifying a previous release to represent the new state.

Example:

```text
schemas/26a/otm-schema.json
schemas/26c/otm-schema.json
```

This makes integrations reproducible and allows consumers to target the OTM version they actually use.

---

## Adding a New OTM Release

The expected publication process is:

```text
1. Obtain the OTM metadata source
        ↓
2. Generate the schema JSON
        ↓
3. Validate the generated artifact
        ↓
4. Create schemas/<release>/
        ↓
5. Publish otm-schema.json
        ↓
6. Update manifest.json
        ↓
7. Validate public access
```

Example for OTM 26C:

```text
schemas/
├── 26a/
│   └── otm-schema.json
└── 26c/
    └── otm-schema.json
```

Then update the manifest:

```json
{
  "manifestVersion": "1.0",
  "registry": "otm-schema-registry",
  "versions": [
    {
      "version": "26A",
      "path": "schemas/26a/otm-schema.json",
      "schemaVersion": "1.0",
      "status": "available"
    },
    {
      "version": "26C",
      "path": "schemas/26c/otm-schema.json",
      "schemaVersion": "1.0",
      "status": "available"
    }
  ]
}
```

A release must not be added to the manifest before its corresponding schema artifact is available.

---

## Consumer Integration

Applications should not hard-code a single schema release unless that behavior is intentional.

Recommended approach:

```text
Application
    ↓
Load manifest.json
    ↓
Select configured OTM release
    ↓
Resolve schema path
    ↓
Load schema
    ↓
Build local indexes/cache
    ↓
Use autocomplete / validation / metadata features
```

A consumer may cache the schema locally after download to avoid repeatedly transferring the complete JSON artifact.

---

## SQL Editor Use Case

One of the primary use cases for this registry is providing structured metadata to SQL development tools.

Example:

```text
User types:

SELECT *
FROM ORDER_R
```

The editor can use the schema registry to suggest:

```text
ORDER_RELEASE
ORDER_RELEASE_LINE
ORDER_RELEASE_REFNUM
ORDER_RELEASE_STATUS
```

After a table is selected:

```sql
SELECT ORH.
FROM ORDER_RELEASE ORH
```

the schema can be used to suggest columns belonging to `ORDER_RELEASE`.

The same metadata can support:

- table autocomplete;
- column autocomplete;
- JOIN suggestions;
- relationship navigation;
- schema browsing;
- SQL validation;
- contextual documentation.

---

## Table Aliases

The registry may include project-maintained aliases commonly used when writing OTM SQL.

Example conceptually:

```json
{
  "table": "ORDER_RELEASE",
  "aliases": [
    "ORH"
  ]
}
```

Aliases are convenience metadata and do not represent an Oracle requirement.

Consumers should always treat the actual database object name as canonical.

---

## Performance Considerations

Schema files may be several megabytes in size.

Consumers should therefore avoid loading the complete registry repeatedly.

Recommended client behavior:

```text
manifest.json
    ↓
Resolve version
    ↓
Download schema once
    ↓
Cache locally
    ↓
Build optimized lookup indexes
```

Typical useful indexes include:

```text
TABLE_NAME → table metadata

TABLE_NAME.COLUMN_NAME → column metadata

ALIAS → TABLE_NAME

TABLE_NAME → related tables
```

Autocomplete engines should normally query these local indexes rather than repeatedly scanning the full JSON document.

---

## Security

This repository contains metadata only.

It must never contain:

- Oracle credentials;
- database passwords;
- authentication tokens;
- customer data;
- customer-specific SQL results;
- private URLs;
- private filesystem paths;
- environment secrets;
- connection strings containing credentials.

Applications consuming this repository remain responsible for their own authentication, authorization, and database access controls.

The schema registry does not provide database connectivity.

---

## Compatibility

The registry describes database metadata associated with specific OTM releases.

It does not guarantee that every OTM environment exposes every database object to every user.

Actual SQL access continues to depend on:

- Oracle Transportation Management configuration;
- environment permissions;
- Oracle Cloud access controls;
- SQL execution restrictions;
- user privileges.

Consumers should therefore treat the registry as metadata for development assistance, not as proof that a specific database operation is permitted.

---

## Repository Principles

This repository follows a small set of design principles:

### Versioned

Every schema belongs to an explicit OTM release.

### Discoverable

Consumers can discover supported versions through `manifest.json`.

### Portable

Artifacts use standard JSON and do not depend on a specific application or cloud provider.

### Immutable

Published release snapshots should remain stable.

### Machine-readable

The registry is designed primarily for programmatic consumption.

### Application-independent

The registry contains OTM metadata, not UI or application-specific business logic.

---

## Roadmap

Potential future enhancements include:

- additional OTM releases;
- automated schema generation;
- automated schema validation;
- JSON Schema validation for registry artifacts;
- release-to-release schema comparison;
- table relationship metadata;
- improved alias metadata;
- column and table descriptions;
- search indexes optimized for autocomplete consumers;
- automated publication workflows.

These additions should preserve backward compatibility whenever possible.

---

## Disclaimer

Oracle and Oracle Transportation Management are trademarks or registered trademarks of Oracle Corporation and/or its affiliates.

This project is independent, community-maintained, and is not affiliated with, sponsored by, or endorsed by Oracle.

The metadata provided by this repository should be validated against the target Oracle Transportation Management environment before being used for production-critical operations.

---

## License

No license has been declared yet.

Until a license file is explicitly added to this repository, repository contents should not be assumed to grant rights beyond those provided by applicable law.