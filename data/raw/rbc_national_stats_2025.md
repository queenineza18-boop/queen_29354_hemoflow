# Rwanda national blood statistics — reference figures

Source: Rwanda Biomedical Centre, *World Blood Donor Day 2026* press release, 14 June 2026.
https://rbc.gov.rw/fileadmin/user_upload/press/Press_Release__English_World_Blood_Donor_Day_2026_2.pdf

These figures are quoted, not measured by this project. They calibrate the derived operational
tables and appear as evidence in the problem statement.

| Figure | Value (2025 unless stated) |
|---|---|
| Whole-blood units collected | 92,615 |
| Blood and blood components supplied | 126,519 |
| Hospital demand (units) | 126,946 |
| Hospital demand satisfaction rate | 99.66% |
| Donations screened for HIV, HBV, HCV, syphilis | 100% |
| Regional centres for blood transfusion | 5 |
| Service established | 1976 |
| Voluntary non-remunerated donors only since | 1985 |
| Jan–May 2026: units collected | more than 40,000 |
| Jan–May 2026: requests met | 99.7% |

Supporting figure — WHO African Region (2024): many countries collect under 5 units per 1000
population against an estimated requirement of 10.
https://www.afro.who.int/publications/status-blood-availability-safety-and-quality-who-african-region-2022-survey-report

Supporting figure — WHO (2026): 120.4 million donations worldwide; 55 countries collect fewer than
10 donations per 1000 people.
https://www.who.int/news-room/fact-sheets/detail/blood-safety-and-availability

## Derived implications used in the design

- ~126,519 components from ~92,615 donations implies roughly 1.37 components per donation
  (whole blood plus separated components), so `record_donation` creates 1–3 units per donation.
- 126,946 demand against 126,519 supplied leaves ~427 units of unmet demand — small nationally,
  but concentrated in specific groups and days, which is exactly what the group × centre stock
  board must expose.
- Five regional centres over 30 districts gives the `REGION`/`CENTRE` cardinality.
