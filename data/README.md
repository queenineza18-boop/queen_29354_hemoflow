# Data — provenance, licence and disclosure

## 1. Real data (primary)

**Blood Transfusion Service Center** — UCI Machine Learning Repository, donated 2008 by Prof. I-Cheng Yeh (Department of Information Management, Chung-Hua University, Taiwan).

- Page: https://archive.ics.uci.edu/dataset/176/blood+transfusion+service+center
- Downloaded as the official UCI archive `blood+transfusion+service+center.zip` (it contains `transfusion.data` and `transfusion.names`) and saved as CSV with the header row kept.
- File: `data/raw/transfusion.csv` — 748 data rows, 5 columns, comma separated, no missing values.
- The original `transfusion.names` documentation is readable on the UCI page above.

The source file ships CRLF line endings and keeps a space before some of the commas (for example `2 ,50,12500,98 ,1`). Those values are preserved exactly as UCI published them; only the line endings may read as LF when the file is opened in this repository.

| Column | Meaning | Type |
|---|---|---|
| `Recency` | months since the donor's last donation | integer |
| `Frequency` | total number of donations | integer |
| `Monetary` | total blood donated, in cc | integer |
| `Time` | months since the donor's first donation | integer |
| `DonorClass` | 1 = donated in March 2007, 0 = did not | 0/1 |

**Licence and ethics.** The dataset is distributed by UCI for research and teaching use, with the citation above. It contains no names, no identifiers and no dates of birth — only aggregated donation behaviour per anonymised record. No personal data that identifies a real person is used in this project. Cite Yeh (2008) as the source and UCI as the distributor.

## 2. National reference figures (not a dataset)

`raw/rbc_national_stats_2025.md` records the published Rwandan figures that calibrate the derived operational tables. These are quotations from a public press release, not records.

## 3. Derived sample data — disclosed, not real

The UCI data describes **donors only**. It contains no units, no expiry dates, no hospitals and no requests. To exercise the schema, Phases II–IV will generate operational rows:

- `DONATION`, `BLOOD_UNIT`, `HOSPITAL`, `REQUEST`, `ISSUE`, `DISCARD`, `DONOR_CONTACT`;
- generated to the distribution implied by the RBC 2025 figures (92,615 units collected, 126,519 components supplied, 126,946 demanded, five regional centres) and to standard component shelf lives;
- every generated row will carry `record_source = 'DERIVED'` so it can never be mistaken for an observed record;
- the generator script will be committed with a fixed seed so results are reproducible.

**This disclosure is deliberate.** Presenting generated records as real data would be fabrication; the assignment requires real data, and the real data here is the donor dimension.

## 4. Reproducing the download

```bash
curl -fsSL -o blood.zip \
  "https://archive.ics.uci.edu/static/public/176/blood+transfusion+service+center.zip"
unzip blood.zip          # yields transfusion.data and transfusion.names
cp transfusion.data transfusion.csv
```

The archive's `transfusion.data` already contains the header row:

```
Recency (months),Frequency (times),Monetary (c.c. blood),Time (months),"whether he/she donated blood in March 2007"
```
