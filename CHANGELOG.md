# Changelog

All notable changes to the `mitm_collector_employee-de_ora` project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [v0.1.1] - 2026-09-16

### Changed

- **Assignment Logic:** Updated SQL extraction to use `COALESCE(ta.supervisor, e.supervisor)` for mapping the supervisor from `temp_assignment` to overwrite the origin.

## [v0.1.0] - 2026-09-15

### Added

- **Initial Release:** Created the `mitm_collector_employee-de_ora` collector based on the generic Oracle employee collector.
- **Assignment Logic:** Integrated complex inline `LEFT JOIN` logic directly into the SQL extraction to overwrite standard `employee` fields with `temp_assignment` and `org` data for employees in company codes `A1` and `DE` with active assignment dates (`enddate >= TRUNC(SYSDATE)`).
- **Oracle Adaptations:** Adjusted SQL syntax for Oracle (unquoted uppercase resolution compatibility, `TO_DATE`, and mapping empty supervisors to `NULL`).
