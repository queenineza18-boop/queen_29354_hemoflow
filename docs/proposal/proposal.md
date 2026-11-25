# HemoFlow: An Oracle PL/SQL System for Blood Component Inventory, Expiry Control and Donor Recall

Queen Ineza · Student ID 29354
INSY 8311 — Database Development with PL/SQL · Instructor: Eric Maniraguha
Faculty of Information Technology, AUCA · Academic year 2026–2027, Semester I
Repository: https://github.com/queenineza18-boop/queen_29354_hemoflow

## Problem Statement

Blood is a perishable, group-specific and safety-critical product. A unit can only be transfused if it is within its shelf life, released by the laboratory, and compatible with the recipient's ABO/Rh group. Across sub-Saharan Africa, stock-outs, transfusion delays and weak transfusion management systems remain documented barriers to safe blood access (Oreh et al., 2026), and 55 countries still collect fewer than 10 donations per 1000 people per year (World Health Organization, 2026). Where supply is this tight, every unit discarded at expiry is a patient who goes without.

Rwanda has largely solved availability at the national level: in 2025 the National Blood Transfusion Division collected 92,615 whole-blood units and supplied 126,519 components against a demand of 126,946, a 99.66% satisfaction rate, with 100% of donations screened for HIV, hepatitis B, hepatitis C and syphilis (Rwanda Biomedical Centre, 2026). The unresolved problem is therefore operational, inside the five regional centres and the hospitals they supply:

- units are not issued first-expiry-first-out, so older units expire on the shelf while newer ones are consumed;
- collected but not yet released units are physically present in storage and are counted as available;
- ABO compatibility is checked by hand, and no system refuses an incompatible issue;
- a group-level stock-out becomes visible only when a patient needs that group;
- status changes on a unit are not audited, so incidents cannot be reconstructed;
- donor recall is a broadcast to the whole list instead of a targeted call to the donors most likely to return.

A proposed centralised blood bank system for Rwanda (Tunezerwe & Musoni, 2025) addresses donor appointment scheduling and real-time availability reporting at district level. It does not model individual units, component shelf lives, laboratory release, first-expiry-first-out allocation, compatibility rules or an audit trail, and therefore cannot refuse an unsafe issue.

## Proposed Solution

HemoFlow is an Oracle 19c database with a PL/SQL rule engine that tracks every blood unit from donation to transfusion or discard. It has three layers.

1. A normalised (3NF) schema of 15 tables: blood groups, component types with their shelf lives, the ABO compatibility matrix, regions, centres, staff, donors, donations, blood units, hospitals, requests, issues, discards, donor contacts and an audit log.
2. A PL/SQL layer that does the work: `record_donation` creates the units a donation yields and computes each expiry date from the component shelf life; `pkg_blood_stock.fulfil_request` allocates units first-expiry-first-out through a `FOR UPDATE` cursor; triggers refuse an expired, unreleased or incompatible issue; a nightly `DBMS_SCHEDULER` job raises alerts for units expiring within seven days; every status change is written to `AUDIT_LOG`.
3. An Oracle APEX console where a centre officer sees stock by blood group and component, records issues, discards expiring units in one click and reviews denied actions.

The rule lives in the database, not in the screen: an issue attempted from APEX, SQL Developer or any other client passes through the same triggers and the same package.

## Data Source

The real data is the Blood Transfusion Service Center donor dataset from the UCI Machine Learning Repository (Yeh, 2008): 748 donor records with `Recency`, `Frequency`, `Monetary`, `Time` and a donation indicator, no missing values and no identifying information, distributed for research and teaching. It is stored unmodified in `data/raw/transfusion.csv` and loaded as the `DONOR` table; its RFM fields drive the `fn_recall_score` function used for targeted donor recall.

Operational volume and age distribution are calibrated to Rwanda's published figures (92,615 units collected, 126,519 components supplied, five regional centres; RBC, 2026). The donation, unit, request and issue records used for demonstration are derived sample data generated to those distributions; they are labelled as derived in `data/README.md` and are never presented as real patient or donor records.

## Planned Tables

`BLOOD_GROUP`, `COMPONENT_TYPE` and `COMPATIBILITY` are the lookup layer. `REGION` and `CENTRE` locate the stock; `STAFF` records who acts. `DONOR` (from the real dataset) gives `DONATION`, and each donation produces one or more `BLOOD_UNIT` rows carrying collection date, expiry date, laboratory test status and unit status. `HOSPITAL` raises `REQUEST` rows for a group and component; `ISSUE` joins a request to the specific unit released against it, so a unit's availability is always derived and never stored twice. `DISCARD` records destruction with a reason, `DONOR_CONTACT` records recall attempts and their outcome, and `AUDIT_LOG` holds row-level history. The schema reaches 3NF by separating all lookups from transactional tables and by removing any attribute that depends on part of a key or on a non-key attribute.

## Planned PL/SQL Features

Procedures `record_donation`, `fulfil_request` and `discard_unit`. Functions `fn_days_to_expiry`, `fn_units_available`, `fn_donor_eligibility` and `fn_recall_score`. An explicit cursor `c_fefo_units` with `FOR UPDATE` ordered by expiry date, and a cursor `FOR` loop `c_expiring_soon` for the nightly report. A package `pkg_blood_stock` with public routines, private constants and an overloaded logging helper. Triggers: `trg_unit_expiry` stamps each unit's expiry date before insert; `trg_block_expired_issue`, `trg_block_unreleased_issue` and `trg_block_incompatible_issue` refuse an unsafe issue; `trg_donation_interval` enforces the 90-day gap between donations; `trg_audit_unit` writes every insert, update and delete on `BLOOD_UNIT` to the audit log using `:OLD` and `:NEW`, which requires `BEFORE`/`AFTER ... FOR EACH ROW` timing (Oracle, 2024). Named exceptions map to `RAISE_APPLICATION_ERROR` codes and are logged then re-raised, so a denied action is both refused and recorded. A fine-grained audit policy additionally tracks changes to unit status.

## Innovation

**Option: user interface, built with Oracle APEX.** The HemoFlow Console connects to the same PDB and gives staff four screens: a stock board by group and component with a days-to-expiry band, a record-issue screen that calls `pkg_blood_stock.fulfil_request`, an expiring-units list with one-click discard, and a denied-actions view built from the audit log. This is relevant because it turns the rules into something a clerk can use under time pressure while the database keeps control. It will be demonstrated live in Phase VII, and documented in `innovation/` with run instructions and screenshots.

## Expected Results

A pluggable database with 15 tables in 3NF, the real donor data loaded and validated; a documented set of procedures, functions, cursors and a package that allocate stock first-expiry-first-out; six triggers that demonstrably refuse an expired, unreleased, incompatible or too-soon repeat donation and record the refusal; an audit log showing who changed what; an APEX console used live to issue units and to show stock by group and centre. Limitations: donor records come from a Taiwanese centre and Rwandan operational volumes are derived; no integration with laboratory analysers; the drone and cold-chain transport layer is out of scope.

## References

Oreh, A., Ajayi, K. V., Okoroiwu, H. U., Obeta, U. M., Maduoma, C., Owusu-Ofori, S., Postma, M., Nwagha, T., & van Hulst, M. (2026). Facilitators and barriers to blood and blood product accessibility and use in sub-Saharan Africa: A systematic review. *Frontiers in Health Services, 6*, 1837188. https://doi.org/10.3389/frhs.2026.1837188

Oracle Corporation. (2024). *CREATE TRIGGER statement* (Oracle Database 19c PL/SQL Language Reference). https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/CREATE-TRIGGER-statement.html

Rwanda Biomedical Centre. (2026, June 14). *World Blood Donor Day 2026* [Press release]. https://rbc.gov.rw/fileadmin/user_upload/press/Press_Release__English_World_Blood_Donor_Day_2026_2.pdf

Tunezerwe, E., & Musoni, W. (2025). Optimizing blood bank operations: A centralized system for enhanced transfusion services in Rwanda's healthcare. *International Journal of Innovative Science and Research Technology, 10*(1). https://doi.org/10.5281/zenodo.14724965

World Health Organization. (2026, June 12). *Blood safety and availability* (Fact sheet). https://www.who.int/news-room/fact-sheets/detail/blood-safety-and-availability

Yeh, I.-C. (2008). *Blood Transfusion Service Center* [Data set]. UCI Machine Learning Repository. https://archive.ics.uci.edu/dataset/176/blood+transfusion+service+center

Zhang, C., Ayer, T., White, C. C., Bodeker, J. N., & Roback, J. D. (2023). Inventory sharing for perishable products: Application to platelet inventory management in hospital blood banks. *Operations Research, 71*(5). https://doi.org/10.1287/opre.2022.2410
