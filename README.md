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
```

## Investigation Finding: Promotion Spike & Refund Surge

### 1. Data Quantification
* **Promotion:** During June 18–21, Garden Wing executed a discounted campaign (`MIDYEAR-CIRCLE`) offering Circle memberships at 225 credits (standard price: 300 credits).
* **Sales Volume:** 76 desk sales were completed across 12 associates, generating 2,052.00 credits in gross commission (12% tier).
* **Refund Spike:** Between June 24–28, 57 of the 76 memberships were refunded—a **75.0% refund rate**.
* **Concentration:** Garden Wing alone represented **61.74% of all refunds** across the entire four-venue museum network (71 of 115 total refunds).

### 2. Footfall Analysis
Cross-referencing the Admissions sheet established that daily attendance during the period averaged **454 visitors/day** (versus the June monthly baseline of 461 visitors/day). This demonstrates that the surge and subsequent refund wave were not driven by macro visitor footfall or attendance swings.

### 3. Financial & Compensation Impact
The 57 cancellations clawed back **1,539.00 credits** from front-desk staff. By reducing net new sales, the cancellations pushed associates **A028** and **A031** below the 25-sale threshold, depriving each associate of their 75-credit monthly bonus.

### 4. Plausible Causes & Recommended Audit
* **Plausible Causes:**
  1. *Exploitation / Multiple Accounts:* Single individuals or coordinated buyers exploiting front-desk admissions privileges and subsequently canceling memberships en masse.
  2. *Misleading Promotion / Buyer Remorse:* Front-desk staff aggressively selling the tier without communicating restrictions, resulting in buyer remorse once terms were reviewed off-site.
* **Distinguishing Audit Test:** Inspect payment method tokens, customer billing details, and support dispute reason codes. Clustered payment tokens confirm repeat gaming, whereas unique external accounts coupled with complaint logs confirm promotional misrepresentation.

---

## Refresh & Repeatability Instructions

The calculation pipeline is designed to be fully repeatable for subsequent months without manual row-by-row intervention:

1. **Ingest New Data:** 
   * Paste the new month's transaction and admissions records into their respective raw sheets (`Raw_Transactions` and `Raw_Admissions`), or replace the source table
2. **Execute Automated Refresh:** 
   * Navigate to the **Data** tab on the Excel ribbon.
   * Click **Refresh All** (or press `Ctrl + Alt + F5`).
3. **Automated Pipeline Processing:**
   * **Duplicate Key Isolation:** Power Query flags and routes all duplicated `transaction_id` rows directly to the `Exceptions` sheet.
   * **Relational Validation:** Refund records are joined to original sales; mismatched products, inconsistent credit amounts, or non-completed sales are isolated into the `Exceptions` register.
   * **Clawback Reversal:** Valid refund reversals are dynamically credited against the original selling associate's account rather than the processing associate.
   * **Payout & Bonus Calculation:** Net eligible new desk sales are recalculated, bonus eligibility ($\ge 25$ net new sales) is evaluated, and the final payout schedule in `Payout_Summary` updates automatically.
