---
title: Automated ASPICE Compliance Pipeline
description: Reduced ASPICE compliance cycle time from 3 weeks of manual auditing to 30 minutes per release for a major automotive OEM.
---

# Automated ASPICE Compliance Pipeline

!!! abstract "Summary"
    **Type**: Delivery project - major automotive OEM
    **Stack**: Jenkins, Python, CI/CD

    **Impact:**

    - Compliance cycle time: 3 weeks manual → 30 minutes automated per release
    - ASPICE Level 2 assessment achieved
    - Full artifact traceability enforced automatically on every build

## Challenge

A major automotive OEM diagnostic software project was undergoing ASPICE (Automotive SPICE) assessment at Level 2. Compliance required generating, linking, and validating a large set of work product artifacts across the software development lifecycle - requirements, design documents, test cases, test results, review records - and demonstrating traceability between them.

This process was being done manually before each assessment cycle. Engineering teams spent approximately 3 weeks per release collecting artifacts, checking consistency, filling gaps, and preparing evidence packages. The manual cycle was error-prone, bottlenecked on a small number of people who understood the ASPICE requirements, and incompatible with the project's release cadence.

## Approach

Architected an automated ASPICE validation pipeline integrated into the existing Jenkins CI/CD infrastructure:

- **Custom integration tooling**: Wrote Python tools to extract artifact metadata from Jira (requirements, test cases, review records), Confluence (design documentation), and the version control system (source code, test results)
- **Traceability matrix generation**: Automated construction of the requirements-to-test-case traceability matrix, flagging gaps (requirements with no linked test, tests with no linked requirement)
- **Evidence packaging**: Automated collection and formatting of work product evidence into the structure expected by ASPICE assessors
- **Pipeline gates**: Jenkins stages that block the build on traceability violations, with a configurable exception list for known deferred items
- **Reporting**: Auto-generated compliance summary reports in a format ready for assessor review

## Results

- Compliance artifact generation reduced from ~3 weeks manual effort to a 30-minute automated pipeline run triggered on each release build
- ASPICE Level 2 assessment passed
- Traceability gaps identified and resolved systematically rather than discovered during audit
- Compliance verification moved from a pre-assessment scramble to a continuous, gated CI/CD check

## Tech Stack

| Layer | Technology |
|---|---|
| CI/CD | Jenkins |
| Scripting | Python |
| Work item source | Jira, Confluence |
| Standard | ASPICE L2, ISO 26262 (ASIL-B) |
| Protocol context | OEM Diagnostic Protocol (UDS) |
