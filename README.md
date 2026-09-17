# OTM Schema Registry

Public, versioned registry of Oracle Transportation Management (OTM) database schema metadata.

This repository provides machine-readable JSON schema files organized by OTM release for use by SQL editors, autocomplete engines, validation tools, documentation systems, development utilities, and other tooling that needs structured knowledge of the OTM data model.

> This is an independent community project and is not affiliated with or endorsed by Oracle.

---

## Purpose

Oracle Transportation Management evolves between releases.

Tables, columns, data types, constraints, and relationships may change over time. Tools that rely on OTM database metadata therefore need to know which schema belongs to which OTM release.

This repository provides an immutable schema snapshot for each supported release.

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
