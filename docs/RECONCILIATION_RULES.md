# BOQ Blindspot — Reconciliation Engine & Rule Definitions

> **Core Principle**: The reconciliation engine is **strictly deterministic**. Calculations, comparisons, quantity bounds, and rule evaluations are executed by tested, auditable Python code—**never by probabilistic language model completion**.

---

## 1. Fundamental Quantity Taxonomy

In construction commercial practice, "quantity" is ambiguous unless explicitly qualified. BOQ Blindspot enforces strict distinction between seven (7) distinct quantity states:

| Quantity State | Definition | Source of Truth | Lifecycle Stage |
|---|---|---|---|
| **1. Contract Quantity ($Q_{contract}$)** | The baseline volume or count specified in the signed Bill of Quantities / Price Schedule. | Contract Document / Baseline BOQ | Baseline Scope |
| **2. Reported Quantity ($Q_{reported}$)** | Field-estimated work described in daily logs, site photos, or verbal notes (often rough or unverified). | Daily site diary, supervisor logs, voice notes | Field Evidence |
| **3. Measured Quantity ($Q_{measured}$)** | Physical dimensions calculated from site measurements (e.g., $Length \times Width \times Depth$). | Signed measurement sheets, take-off sheets | Field Verification |
| **4. Verified Quantity ($Q_{verified}$)** | A measured or reported quantity that has been reviewed and confirmed by the subcontractor's QS. | Human QS review in BOQ Blindspot | Commercial Signoff |
| **5. Claimed Quantity ($Q_{claimed}$)** | The cumulative or period quantity submitted by the subcontractor in an interim payment application. | Interim Payment Application / Valuation Claim | Billing Submission |
| **6. Certified Quantity ($Q_{certified}$)** | The quantity officially assessed and approved for payment by the main contractor's QS or architect. | Payment Certificate / Valuation Certificate | Valuation Assessment |
| **7. Paid Quantity ($Q_{paid}$)** | The quantity reflected in actual funds disbursed / remittance advices. | Remittance advice / Bank record | Cash Realization |

> **Inviolable Rule**: These seven quantities must never be treated as interchangeable. Arithmetic that combines or compares them must state explicitly which states are involved. Never invent or synthesize values for unverified states.

---

## 2. Core Mathematical Baseline & Boundaries

### 2.1 The Baseline Entitlement Ceiling
For any given BOQ item $i$:

$$Q_{authorized}(i) = Q_{contract}(i) + \sum Q_{approved\_variations}(i)$$

- **Contract Quantity ($Q_{contract}$)**: Immutable baseline from contract import.
- **Approved Variations ($\sum Q_{approved\_variations}$)**: Variations with explicit approval status (`approved`). Variations marked `draft`, `submitted`, or `pending` do **not** increment $Q_{authorized}$.

### 2.2 Quantity Progression Invariants
In healthy construction administration, the seven quantity states form two linked progression chains across sequential billing cycles ($t = 1, \dots, N$):

**1. Field Execution Chain**:
$$Q_{reported}(i) \longrightarrow \text{physical measurement} \longrightarrow Q_{measured}(i) \longrightarrow \text{QS signoff} \longrightarrow Q_{verified}(i)$$

**2. Commercial Valuation Chain**:
$$Q_{claimed}(i, t) \le Q_{authorized}(i, t)$$

$$Q_{certified}(i, t) \le Q_{claimed}(i, t)$$

$$Q_{paid}(i, t) \le Q_{certified}(i, t)$$

**3. Execution vs. Valuation Interface**:
$$Q_{verified}(i) \longleftrightarrow \sum_{t=1}^{current} Q_{claimed}(i, t)$$

Any unexpected deviation between verified physical work ($Q_{verified}$) and billed claims ($Q_{claimed}$), or any violation of the valuation bounds, triggers a candidate finding for investigation.

---

## 3. Reconciliation Rules Catalog

### Rule REC-01: Cumulative Over-Claim Check
- **Intent**: Prevent claiming quantities in excess of contractual limits without an approved variation.
- **Formula**:
  $$\Delta Q = \sum_{t=1}^{current} Q_{claimed}(i, t) - Q_{authorized}(i)$$
- **Trigger**: If $\Delta Q > 0$, flag as `OVER_CLAIM_RISK`.
- **Severity**: High.
- **Traceability**: Links to contract line, approved variation records, and all historical claim lines.

### Rule REC-02: Unbilled Executed Work (Blindspot Discovery)
- **Intent**: Identify physical site work that has been measured or verified on site but is absent or under-claimed in the interim payment application.
- **Formula**:
  $$\Delta Q_{unbilled} = Q_{verified}(i) - \sum_{t=1}^{current} Q_{claimed}(i, t)$$
- **Trigger**: If $\Delta Q_{unbilled} > \text{Tolerance Threshold}$ (e.g., $> 0.01$), flag as `POTENTIAL_UNBILLED_WORK`.
- **Severity**: Medium / Opportunity.
- **Important Qualification**: A positive variance indicates potential unbilled work for investigation; it is **not guaranteed recoverable revenue** until reviewed against contract billing milestones, retention rules, and cut-off dates.

### Rule REC-03: Certification Under-Valuation / Deduction
- **Intent**: Highlight where the main contractor or client QS has certified less quantity than claimed without an explicit deduction reason.
- **Formula**:
  $$\Delta Q_{deducted}(t) = Q_{claimed}(i, t) - Q_{certified}(i, t)$$
- **Trigger**: If $\Delta Q_{deducted}(t) > 0$, flag as `CERTIFICATION_SHORTFALL`.
- **Severity**: Medium.

### Rule REC-04: Rate Consistency & Discrepancy
- **Intent**: Ensure that the unit rate applied in the payment claim or valuation certificate matches the agreed contract rate or approved variation rate.
- **Trigger**: If $|R_{claimed}(i) - R_{contract}(i)| > 0.001$, flag as `RATE_MISMATCH`.
- **Severity**: High.
- **Traceability**: Cites the contract rate table vs. the specific claim line.

### Rule REC-05: Unit Compatibility & Dimensional Integrity
- **Intent**: Prevent comparing or calculating values across mismatched physical dimensions.
- **Rule**: Before any quantity comparison or summation, the units of measure must match or have an explicit, verified conversion factor.
- **Trigger**: If $Unit(A) \neq Unit(B)$ (e.g., `m²` vs `m³`, `linear metres` vs `tonnes`), the engine **must not** guess a conversion. It must halt comparison for that item and flag an `INCOMPATIBLE_UNITS_WARNING`.
- **Severity**: Critical.

### Rule REC-06: Unsubstantiated Variation Claim
- **Intent**: Identify variation items claimed on billing sheets that lack an approved status or lack attached site evidence.
- **Trigger**: If $Status(Variation) \neq \text{'approved'}$ OR $EvidenceCount(Variation) == 0$, flag as `UNSUBSTANTIATED_VARIATION`.
- **Severity**: High.

### Rule REC-07: Overlapping & Duplicate Claim Detection
- **Intent**: Prevent billing the same physical measurement or site ticket twice across different line items or billing periods.
- **Trigger**: If two distinct `BillLine` entries cite the same `MeasurementRecord` or identical physical grid/chainage locations within overlapping date windows, flag as `POTENTIAL_DUPLICATE_CLAIM`.
- **Severity**: High.

---

## 4. Handling Ambiguity, Missing Data & Conflicts

The engine operates under strict completeness constraints:

1. **Never Invent Missing Values**:
   - If a measurement sheet omits a thickness or multiplier, the calculated quantity is **Unknown**, not zero or one.
   - If a rate is missing, the total financial impact is marked as **Unquantified**, not zero.
2. **Explicit Ambiguity Warnings**:
   - When OCR text is below confidence thresholds, or conflicting quantities appear across two different documents (e.g., site diary says 30m², delivery ticket says 25m²), the engine creates a `CONFLICTING_EVIDENCE_FINDING`.
3. **No Phantom Entitlement**:
   - An unbilled work finding must never state "You are owed £X". It must state: "Verified site measurements reflect $Q_{verified} = 150\,\text{m}²$, whereas cumulative billing reflects $Q_{claimed} = 100\,\text{m}²$. Estimated variance: $50\,\text{m}²$ at contract rate £10.00/m² (£500.00 potential unbilled scope subject to contract milestones)."

---

## 5. Finding Structure & Traceability Requirements

Every `Finding` generated by the reconciliation engine must persist the following auditable fields:

```json
{
  "finding_id": "FIND-2026-0042",
  "audit_run_id": "RUN-0012",
  "rule_id": "REC-02",
  "rule_version": "1.0.0",
  "item_id": "BOQ-ITEM-034",
  "severity": "HIGH",
  "finding_type": "POTENTIAL_UNBILLED_WORK",
  "summary": "Verified site measurement exceeds cumulative claimed quantity by 25.00 m²",
  "calculation_basis": {
    "verified_quantity": 125.00,
    "claimed_quantity_cumulative": 100.00,
    "variance_quantity": 25.00,
    "unit": "m2",
    "contract_unit_rate": 45.00,
    "estimated_variance_amount": 1125.00,
    "currency": "GBP"
  },
  "evidence_references": [
    {
      "evidence_id": "EVI-0089",
      "file_name": "Site_Measurement_Sheet_Grid_B_W42.xlsx",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "location": "Sheet1!C14:F14",
      "fact_id": "FACT-0210"
    }
  ],
  "assumptions_and_limitations": [
    "Assumes contract unit rate is applicable without tiered volume discounts.",
    "Assumes work meets contractual completion and testing milestones."
  ],
  "disposition": "PENDING_REVIEW"
}
```

---

## 6. Safe Audit Run Guarantee

- **Atomic Audit Runs**: An audit run either completes successfully and commits its findings, or aborts cleanly.
- **No False Clean Audits**: If files failed extraction, OCR was unreadable, or rates were missing, the resulting report header must prominently display:
  > `AUDIT STATUS: COMPLETED WITH UNRESOLVED WARNINGS (3 files unparsed, 2 rates missing). THIS IS NOT A CLEAN AUDIT.`
