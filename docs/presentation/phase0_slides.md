# Phase 0 presentation — 12 slides

Copy each section onto one slide. Times New Roman throughout; title 32–36 pt, body 20–24 pt, no more than six lines of body text per slide. Put the diagram on slide 8 and the screenshot placeholder on slide 7.

---

## Slide 1 — Title

**HemoFlow: Blood Component Inventory, Expiry Control and Donor Recall**

An Oracle PL/SQL system for blood units, not just blood quantities

Queen Ineza — Student ID 29354
INSY 8311 Database Development with PL/SQL
Instructor: Eric Maniraguha · Semester I, 2026–2027
GitHub: github.com/queenineza18-boop/queen_29354_hemoflow

---

## Slide 2 — Background and problem

- Blood is perishable, group-specific and usable only after laboratory release.
- 120.4 million donations a year worldwide, but 55 countries still collect under 10 per 1000 people (WHO, 2026).
- In sub-Saharan Africa, stock-outs and transfusion delays are documented barriers to care (Oreh et al., 2026).
- **Who is affected:** the five regional transfusion centres, the hospitals they supply, and every patient in an obstetric emergency or a road accident.

---

## Slide 3 — Motivation

- Rwanda already reports 99.66% of hospital blood demand satisfied (RBC, 2026) — so *availability* is solved.
- The loss that remains is operational: units expiring on the shelf, unreleased units counted as stock, compatibility checked by hand, no audit trail.
- A national percentage cannot tell a doctor whether O-negative is in the fridge at 2 a.m.
- Every avoidable discard is a patient who goes without.

---

## Slide 4 — Objectives

**Main:** build an Oracle database with PL/SQL that manages blood components from donation to issue and refuses to release an unsafe unit.

Specific:
1. Model donation-to-transfusion in a 3NF schema of 15 tables.
2. Implement procedures, functions, cursors and a package for donation, FEFO allocation and discard.
3. Enforce six business rules with triggers and write an audit trail.
4. Build an Oracle APEX console for centre staff.
5. Validate on 748 real donor records and Rwanda's published volumes.

---

## Slide 5 — Research: existing solutions and their gaps

| Source | What it gives | Gap it leaves |
|---|---|---|
| Tunezerwe & Musoni (2025) — centralised Rwandan blood bank system | Donor self-scheduling, district-level real-time availability | No unit-level inventory, no expiry, no release status, no compatibility rule, no audit |
| Zhang et al. (2023) — perishable inventory in hospital blood banks | Proof that age-aware allocation cuts wastage | A model, not an operational database; no enforcement layer |
| WHO (2026) / WHO AFRO (2024) | National and regional donation rates | Aggregate reporting cannot show a group-level stock-out |

**The gap:** nobody enforces the rules inside the data layer.

---

## Slide 6 — Data source

**UCI Blood Transfusion Service Center** — Yeh (2008), archive.ics.uci.edu/dataset/176

- 748 donor records, 5 attributes, no missing values, no identifying fields.
- Fields: Recency (months since last donation), Frequency, Monetary (cc donated), Time (months since first donation), donated-in-March-2007 flag.
- Sample records: `2, 50, 12500, 98, 1` · `0, 13, 3250, 28, 1` · `1, 16, 4000, 28, 1`
- Calibrated against RBC 2025 national figures: 92,615 units collected, 126,519 supplied.
- Operational tables are derived sample data and labelled as such.

---

## Slide 7 — Proposed solution and innovation

**Solution:** unit-level traceability with the rules in PL/SQL — donations produce units, units carry expiry and release status, requests are filled first-expiry-first-out.

**Innovation (option: user interface — Oracle APEX):** *HemoFlow Console*
- Stock board by group × component × centre, with a days-to-expiry band
- Record issue → calls `pkg_blood_stock.fulfil_request`, so the screen cannot bypass a rule
- Expiring units → one-click discard with audit entry
- Denied actions → who tried, which unit, why it was refused

*[screenshot placeholder: APEX stock board]*

---

## Slide 8 — Planned database

*[insert the ERD diagram from README.md]*

15 tables: `BLOOD_GROUP`, `COMPONENT_TYPE`, `COMPATIBILITY`, `REGION`, `CENTRE`, `STAFF`, `DONOR`, `DONATION`, `BLOOD_UNIT`, `HOSPITAL`, `REQUEST`, `ISSUE`, `DISCARD`, `DONOR_CONTACT`, `AUDIT_LOG`.

Key relations: one `DONATION` produces many `BLOOD_UNIT`s; one `REQUEST` is met by many `ISSUE`s; one `ISSUE` points to exactly one `BLOOD_UNIT`. Availability is derived from `BLOOD_UNIT` minus `ISSUE` — never stored twice.

---

## Slide 9 — PL/SQL approach: where each feature is used

- **Trigger** `trg_block_expired_issue` — refuses to issue any unit whose `expiry_date` has passed.
- **Trigger** `trg_block_unreleased_issue` — refuses a unit still `PENDING` laboratory release.
- **Trigger** `trg_block_incompatible_issue` — reads `COMPATIBILITY` and refuses an A→O or B→A issue.
- **Trigger** `trg_donation_interval` — refuses a donor who donated fewer than 90 days ago.
- **Trigger** `trg_audit_unit` — writes `:OLD`/`:NEW` of every unit change to `AUDIT_LOG`.
- **Procedure** `record_donation` — creates units and stamps expiry from the component shelf life.
- **Cursor** `c_fefo_units` (`FOR UPDATE`, ordered by expiry) — the allocation engine.
- **Package** `pkg_blood_stock` — public API used by APEX; exceptions `-20001` to `-20004`.
- **Job** `DBMS_SCHEDULER` — nightly list of units expiring within 7 days.

---

## Slide 10 — Expected results

- A working PDB with 15 tables in 3NF and the 748 real donor records loaded and validated.
- Live demonstration that each trigger refuses an illegal action, with the refusal visible in the audit log.
- A FEFO allocation shown side by side with a naive "oldest request first" allocation, proving fewer expired units.
- The APEX console issuing a unit end to end and showing stock by group and centre.
- A donor recall list scored from the real RFM data.

---

## Slide 11 — Future improvements and limitations

- Donor records come from a Taiwanese centre; Rwandan operational volumes are derived, not observed.
- No integration with laboratory analysers, so release status is entered manually.
- Cold-chain temperature monitoring and drone transport are out of scope.
- Future: hospital-level demand forecasting, cross-centre stock sharing (following Zhang et al., 2023), and an SMS recall channel via ORDS.

---

## Slide 12 — References

Oreh, A., et al. (2026). Facilitators and barriers to blood and blood product accessibility and use in sub-Saharan Africa. *Frontiers in Health Services, 6*, 1837188.

Oracle Corporation. (2024). *CREATE TRIGGER statement*. Oracle Database 19c PL/SQL Language Reference.

Rwanda Biomedical Centre. (2026, June 14). *World Blood Donor Day 2026* [Press release].

Tunezerwe, E., & Musoni, W. (2025). Optimizing blood bank operations. *IJISRT, 10*(1).

World Health Organization. (2026, June 12). *Blood safety and availability* (Fact sheet).

Yeh, I.-C. (2008). *Blood Transfusion Service Center* [Data set]. UCI Machine Learning Repository.

Zhang, C., et al. (2023). Inventory sharing for perishable products. *Operations Research, 71*(5).
