# Diamond Manufacturing: Payroll Validation & ERP Data Restructuring

## Overview

Led comprehensive payroll validation and operational data standardization at a diamond manufacturing company, identifying a recurring five-figure monthly financial loss from outdated ERP logic.

## Key Impact

- **Financial:** Identified **50000 rupee** monthly loss from systematic payroll calculation error
- **Operational:** Standardized fragmented production data across multiple departments
- **System:** Coordinated with ERP vendor to add missing processes and enable critical reports

## The Problem

The company had expanded from natural to lab-grown diamonds, but ERP rate calculation logic hadn't been updated. Combined with severely messy operational data (inconsistent worker names, duplicate process entries, no unique IDs), payroll accuracy was impossible to verify.

## Approach

- Cleaned and standardized large operational datasets using fuzzy matching + manual validation
- Mapped complete diamond production flow to validate data capture logic
- Independently recalculated payroll using Python/SQL rather than trusting ERP outputs
- Performed variance analysis to identify systematic vs. random errors

## Key Finding

ERP was using "Making CTS" (post-processing weight) for all diamonds, including lab-grown. Lab-grown rates should use "Issue CTS" (pre-processing weight). This systematic logic error caused recurring monthly financial loss that wasn't visible in standard reports.

## Tools & Techniques

**Languages:** Python, SQL  
**Libraries:** pandas, RapidFuzz, FuzzyWuzzy, numpy,openlyxl  
**Techniques:** Fuzzy string matching, data standardization, independent logic validation, variance analysis

## Read More

[📄 Full detailed case study →](./diamond-manufacturing-payroll-analysis/case-study.md)

---

**Skills demonstrated:** Data cleaning • Business logic validation • ERP analysis • Stakeholder coordination • Financial analysis
