# Museum Membership Incentive & Audit Pipeline

An end-to-end financial reconciliation pipeline and anomaly audit built using Microsoft Excel and Power Query. This project processes monthly membership transactions and visitor admissions data across multiple venues to calculate net sales incentives, enforce data integrity controls, claw back refund commissions, and uncover operational irregularities.

---

## Business Problem & Scenario

A cultural institution operating four physical venues (*Clock Hall*, *Garden Wing*, *North Gallery*, and *River Annex*) incentivizes front-desk associates on membership sales. The challenge requires an automated, fully repeatable monthly workflow to:
1. **Enforce Transaction Data Controls:** Flag duplicated keys, orphan refunds, and mismatched values into an auditable exception log prior to calculation.
2. **Reconcile Variable Incentive Pay:** Calculate multi-tiered commissions across product categories and transaction kinds (New vs. Renewal), trace refund reversals back to original sellers, and determine volume threshold bonuses.
3. **Investigate Operational Concentrations:** Cross-reference sales spikes and refund surges with venue footfall to diagnose root causes and quantify downstream compensation impacts.

---

## Key Features

- **Automated ETL Pipeline (Power Query):** Ingests raw multi-sheet records, handles self-joins to trace refunds to their original point of sale, isolates invalid rows, and refreshes entirely via `Data > Refresh All`.
- **Audit & Exception Register:** Holds duplicate transaction IDs and invalid refund references (e.g., refunds pointing to VOID sales or non-existent IDs) without dropping rows blindly.
- **Relational Commission Clawbacks:** Correctly attributes refund deductions to the original seller rather than the associate processing the refund.
- **Two-Way Reconciliation:** Mathematically reconciles the grand payout total ($33,455.16) across both vertical associate summaries and horizontal financial component totals.
- **Root-Cause Investigation:** Leverages pivot modeling and daily admissions data to isolate an anomalous 75% refund surge tied to a localized marketing campaign.

---

## Incentive & Business Logic

| Event Type | Sale Kind | Product Tier | Incentive Rate | Rules / Caveats |
| :--- | :--- | :--- | :---: | :--- |
| **Desk Sale** | NEW | Single | **8%** | Completed events only; PENDING/VOID earn 0 |
| **Desk Sale** | NEW | Duo | **10%** | Completed events only |
| **Desk Sale** | NEW | Circle | **12%** | Completed events only |
| **Desk Sale** | RENEWAL | All Products | **3%** | Completed events only |
| **Refund** | Any | Any | **Reversal** | Reverses original incentive credited to original seller |
| **Online Sale / Refund** | Any | Any | **0% / 0 reversal** | Online transactions earn zero commission |
| **Monthly Bonus** | NEW | All Products | **+75 Credits** | Granted if associate achieves $\ge 25$ Net Eligible NEW Sales |

$$\text{Final Payout} = \max(0, \text{Sale Incentives} - \text{Refund Reversals} + \text{Bonus}) \text{}$$

---

## Executive Summary & Reconciliation

Out of 2,121 raw transaction records, 6 rows were held for administrative audit (4 duplicate IDs, 2 invalid refund linkages), leaving 2,115 valid records.

```text
============================================================
RECONCILIATION SUMMARY (JUNE 2026)
============================================================
Gross Eligible Sales Incentives:           32,990.82 credits
Less: Valid Refund Reversals (113):       - 2,310.66 credits
Plus: Monthly Bonuses (37 staff @ 75):     + 2,775.00 credits
------------------------------------------------------------
RECONCILED GRAND TOTAL PAYOUT:             33,455.16 credits
============================================================
Variance between vertical and horizontal math:          0.00
============================================================
