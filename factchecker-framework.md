---
layout: default
title: "Factchecker Framework"
permalink: /factchecker-framework
---

# Factchecker Framework
*Last Updated: 2026-05-11*

## 1. Introduction
The framework is a logical architecture designed to decouple the **observation of an event** from the **validity of its content**. By separating the "Reported" status from the "Verified" status, the system creates a stable, audit-ready ledger that remains functional even when processing misinformation or logical paradoxes.

---

## 2. Data Schema

### A. Column Definitions

* **who_said_it**: The source entity or origin (e.g., `base_url`) credited with making the claim or publishing the report.
* **report_content**: The specific string, snippet, or proposition being evaluated for existence and accuracy.
* **State**: The classification of the data based on the intersection of the Reported and Verified gates.

---

### B. Design Policy

* **Attribution Ledger:** Every entry must first be validated as a `Reported_Fact`. The system prioritizes *who* said *what* before determining truth.
* **Content Neutrality:** The reporting layer remains `True` regardless of the content's validity, ensuring the audit trail never breaks.
* **Recursive Safety:** Logical paradoxes are contained within the `Verified_Fact` gate, preventing them from affecting the stability of the primary ledger.

---

## 3. The Interaction Matrix

| Reported | Verified | State Name |
|---|---|---|
| **True** | **True** | **The Gold Standard**: Authenticated report of an objective truth. |
| **True** | **False** | **The Misinformation**: Confirmed event of a false claim being made. |
| **False** | **True** | **The Hidden Truth**: Objective truth that lacks an indexed source. |
| **False** | **False** | **The Non-Event**: Information existing neither in reportage nor reality. |

---

## 4. Maintenance Notes

- **Extensibility:** Future iterations may include a `Confidence_Score` to weight the `Verified_Fact` result.
- **Generation:** This schema is optimized for automated ingestion via Fact Check APIs (mapping `publisher` to `who_said_it`).
- **Interpretation:** Users should interpret a "True" Reported status as an objective historical event, independent of the "Verified" status.
