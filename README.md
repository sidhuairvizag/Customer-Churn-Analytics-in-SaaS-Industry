# SaaS Customer Churn Analytics

A junior-analyst case study on a dirty SaaS book of business: clean the files, define the metric, find who is at risk, read the exit comments, and turn that into a four-page Power BI brief.

This is descriptive analytics. It is not a churn-prediction model and not a production system.

**Public figures (match the dashboard — use these on a resume):**

94,687 accounts · 19,676 churned · **20.78%** churn · **₹8.48M** churned ARR · 3,319 interviews · 3,266 classified replies

Repo: https://github.com/sidhuairvizag/Customer-Churn-Analytics-in-SaaS-Industry

---

## Official numbers vs notebook numbers

The clean customer file has **95,000** unique IDs. The two published counts remove different leftover rows.

| Source | Accounts | Churned | Rate | What was removed |
| --- | --- | --- | --- | --- |
| Power BI (public number) | **94,687** | **19,676** | **20.78%** | 313 rows where plan and region stayed `Unknown` |
| Notebook | 94,419 | 19,740 | 20.9% | 581 rows where `Churn_Flag` was still blank |

Neither number is a bug.

- Notebook rate = churn among accounts we can label. Blank flags were **not** filled.
- Dashboard rate = churn among accounts with a usable plan name.

Slides and this README use **94,687 and 20.78%** so they match the `.pbix`.

---

## Business question

Customers are leaving. Exit interviews exist. Management needs three things that can be acted on next week:

1. Is churn a sudden 2025 spike or a steady leak?
2. Which slice is high-risk (rate) and which slice is expensive (lost ARR)?
3. What do leavers actually say — and does that change the fix?

---

## What I delivered

| Step | Work |
| --- | --- |
| Cleaning | Standardized IDs, labels, dates, spend, age, tenure, tickets. Removed 320 duplicate rows and 60 extra ID collisions. Left 581 blank churn flags untouched. |
| Structured EDA | Churn rate and lost ARR by plan, tenure, region, tickets, signup year. |
| Text | Cleaned HTML / emojis / stubs. Tagged 3,266 usable interviews with a primary reason, mood, and co-mention flags. |
| Dashboard | Four Power BI pages with plan / region / tenure / signup-year filters. |
| Decision | Three retention actions, not a model score. |

---

## Headline findings

| Finding | Number |
| --- | --- |
| Company churn | 20.78% on 94,687 accounts |
| Lost annual spend | ₹8.48M |
| Highest-rate plans | Basic 22.11% (7,294 / 32,993) · Standard 22.10% (7,324 / 33,133) |
| Lowest-rate plan | Enterprise 15.08% (736 / 4,882) |
| Largest lost ARR | Premium ~₹3.53M at 18.25% |
| Riskiest tenure | 0–6 months **26.12%**; later buckets sit near 21% |
| Tickets | 0–3 tickets ~21%; **4+ tickets ~30%** |
| Region | 20.5–21.1% — not a driver |
| Signup year | Rate is flat — not a 2025 spike |
| Stated reasons | Value **40.88%** · Price **40.39%** · Support 10.69% · Product 4.41% · Performance 2.51% · Onboarding 0.77% · Competitor 0.37% |
| Mood | Frustrated **65.74%** · Disappointed 21.19% · Hopeful 10.07% · Neutral 3.00% |

Two results that change the action:

- **Rate and rupees disagree.** Basic and Standard leave more often. Premium is where most lost ARR sits.
- **Price and Value are almost the same size.** A blanket discount treats two different complaints as one.

Notebook cut used to size the action lists (known churn flags only):

| Slice | n | Churn | Churned ARR |
| --- | --- | --- | --- |
| First 6 months AND 4+ tickets | 144 | 37.5% | ₹22,014 |
| 4+ tickets, any tenure | 3,233 | 29.7% | ₹415,361 |
| Basic + Standard | 65,710 | 22.2% | ₹3,501,038 |

The 144-row cell is a short priority queue, not the whole ₹8.48M problem. The weekly list is 4+ tickets.

---

## Dashboard

Filters on every page: Plan, Region, Tenure, Signup Year.  
Screenshots: `bi dashboard images/`. Interactive file: `power-bi file/Customer Churn Analytics in SaaS Industry.pbix`.

### Page 1 — SaaS Churn Overview
Scorecard, churn by signup year, churned ARR by plan, plan table.

Churn is ~20.8% and stable by signup year. Basic/Standard hold the highest rate. Premium holds the largest lost ARR (~₹3.53M).

### Page 2 — Who Is High Risk
Churn by plan, tenure, region, ticket count, plus a tenure × tickets grid.

Region does not move the rate. Tickets stay near 21% through 3, then jump. First operational cut: 0–6 month accounts with 4+ tickets.

### Page 3 — Why Customers Leave
Primary reason on usable interviews only (3,266 of 3,319).

Value 40.88% and Price 40.39%. Do not only discount. Support is third as the *primary* label; co-mentions sit in the flag columns.

### Page 4 — Mood of Departing Customers
Mood mix and reason × mood matrix.

Frustrated is the default voice (65.74%). Support and Performance are almost only Frustrated. Price and Value hold every Disappointed comment. Hopeful (~10%) usually means “we would have stayed if X improved.”

---

## Data

### Raw — `data/raw data/`

| File | Shape (raw) | Problems |
| --- | --- | --- |
| `churn_customers_95k_dirty.csv` | 95,380 × 10 | Duplicate rows, colliding IDs, mixed churn labels (`0` / `1` / `Yes` / `No` / blank), messy gender/region/plan strings, `₹` in spend, invalid ages, negative tenure, mixed date formats |
| `churn_exit_interviews_3500_dirty.csv` | 3,605 × 3 | Duplicate rows, HTML, emojis, truncated text, `N/A` / `NULL`, `XC`-prefixed IDs, mixed churn labels |

Customer columns: `Customer_ID`, `Age`, `Gender`, `Region`, `Tenure_Months`, `Subscription_Type`, `Monthly_Spend`, `Support_Tickets`, `Churn_Flag`, `Signup_Date`.

Interview columns: `Customer_ID`, `Churn_Flag`, `Exit_Reason_Text`.

### Processed — `data/processed data/`

| File | Grain | Added fields |
| --- | --- | --- |
| `customer_clean.csv` | 95,000 IDs | `Annual_Spend`, `Signup_Year`, `Tenure_Group`, `Spend_Group`; standardized plan / region / gender |
| `interviews_classified.csv` | one cleaned interview per row | `usable`, `primary_reason`, `mood`, `flag_Price` … `flag_Competitor` |

`Churn_Flag` is missing when the raw label could not be mapped. Those 581 rows stay in the customer file and are excluded from the notebook rate.

---

## Cleaning rules (customers)

Start: 95,380 rows.

| Step | Count |
| --- | --- |
| Duplicate full rows removed | 320 |
| Extra ID collisions after ID normalize (keep first) | 60 |
| Invalid ages (<18 or >80) set to median | 474 |
| Negative tenure flipped; tenure > 120 capped | 190 / 190 |
| Negative tickets set to 0 | 99 |
| Churn labels left missing | 581 |
| Clean customer table | 95,000 × 14 |

Other rules:

- IDs: trim, upper, map `XC…` → `C…`, zero-fill to `C000001`.
- Gender / region / plan: regex + lookup (`mid`/`std`/`pro` → Standard, `gold`/`prm` → Premium, `corp`/`ent` → Enterprise). Unmapped values stay `Unknown`.
- Churn: `1/yes/y/true` → 1; `0/no/n/false` → 0; else missing.
- Dates: `%d-%m-%Y`, then `%m-%d-%Y`; drop a trailing `00:00`.
- Spend: strip currency symbols, then numeric.
- `Annual_Spend` = `Monthly_Spend` × 12. That is the ARR proxy in every “churned ARR” figure.

---

## Text rules (interviews)

- Strip HTML, emojis, extra spaces, and null-like tokens.
- Short or empty replies: `usable = False`, `primary_reason = Unclassified`. Dropped from reason shares.
- Each comment is scored against a fixed keyword list for Price, Value, Support, Product, Performance, Onboarding, Competitor. A comment can light more than one flag. Charts use `primary_reason`.
- Mood is tagged from the same cleaned text: Frustrated, Disappointed, Hopeful, Neutral.

This is a documented taxonomy, not a trained classifier. Mixed comments can land on the wrong primary bar — that is why the flag columns exist.

---

## Three recommendations

1. **Rescue noisy new accounts.**  
   Onboarding plus a 48-hour reply SLA for tenure 0–6 months with 4+ tickets. That cell is 37.5% churn and only 144 accounts in the notebook cut. Short list, high rate.

2. **Split Price from Value. Do not blanket-discount.**  
   Price = too expensive / renewal surprise. Value = no ROI / unused features. Discount only Price. For Value, send usage and a one-page ROI note before renewal.

3. **Weekly save queue: every account that crosses 4 tickets.**  
   0–3 tickets stay near the company rate. 4+ tickets jump to ~30% and ₹0.42M lost ARR in the notebook cut. CS can run that list without a new tool.

---

## Metric definitions

- **Churn rate (notebook)** = churned / accounts with a known `Churn_Flag`.
- **Churn rate (dashboard)** = churned / accounts in the current Power BI filter. The unfiltered model is 94,687 accounts after dropping `Unknown` plan.
- **Churned ARR** = sum of `Annual_Spend` on churned accounts. Not invoiced ARR from a billing system.
- **Reason share** = primary label count / 3,266 usable classified interviews.
- **Signup-year chart** = current churn by year the account signed up. Not a monthly hazard curve.

---

## Repo layout

```text
Customer-Churn-Analytics-in-SaaS-Industry/
├── README.md
├── requirements.txt
├── notebook code/
│   └── Customer Churn Analytics in SaaS Industry.ipynb
├── data/
│   ├── raw data/
│   │   ├── churn_customers_95k_dirty.csv
│   │   └── churn_exit_interviews_3500_dirty.csv
│   └── processed data/
│       ├── customer_clean.csv
│       └── interviews_classified.csv
├── power-bi file/
│   └── Customer Churn Analytics in SaaS Industry.pbix
└── bi dashboard images/
    ├── Dashboard - 1 SaaS Churn Overview.jpg
    ├── Dashboard - 2 Who Is High Risk.jpg
    ├── Dashboard - 3 Why Customers Leave.jpg
    └── Dashboard - 4 Mood of Departing Customers.jpg
```
