WorkforceIQ is a decision interface for HR Managers at Medika Nusantara.
It is not just a dashboard — it is a decision product, enabling HR Managers to take immediate action based on what they see.

-Key Features
| Feature                 | Description                                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------------------------- |
| **Dataset Toggle**      | Switch between Dataset A (Messy) and Dataset B (Clean) to directly observe differences in insight quality |
| **Dashboard**           | Overview of high-risk employees, average confidence, skill gaps, and pending reviews                      |
| **Employee Risk List**  | Filter by risk tier and view attrition score + skill profile per employee                                 |
| **Identity Resolution** | Explorer showing resolved, flagged, and unresolved records across 3 systems                               |
| **Data Quality Impact** | Side-by-side comparison of messy vs clean data — the core argument of the case                            |
| **Data Architecture**   | 5-layer model from source systems to AI output                                                            |

-Confidence Signaling
| Confidence | UI Representation                                           |
| ---------- | ----------------------------------------------------------- |
| ≥ 0.80     | Score fully displayed with a green badge                    |
| 0.60–0.79  | Score displayed with an uncertainty note and an amber badge |
| < 0.60     | Score **hidden** → “Review Needed”                          |


Principle: It is better to hide a prediction than to show one that cannot be trusted.

-About the Dataset
Dataset A — Messy (Enterprise Reality)

Represents the real condition of Medika Nusantara’s data at initial access:

| Issue                                | Count          |
| ------------------------------------ | -------------- |
| Missing skill fields                 | 48% of records |
| Missing performance history          | 44% of records |
| Missing location                     | 17% of records |
| Conflicting join dates (HRIS vs ATS) | 30 records     |
| Near-duplicate records               | 10 records     |
| Inconsistent job titles              | 23+ variants   |

Dataset B — Clean (Post-Processing)

After identity resolution, skill inference, and confidence scoring:

| Metric                        | Value            |
| ----------------------------- | ---------------- |
| Identity resolved             | 200 / 200 (100%) |
| Avg data confidence score     | 0.71             |
| HIGH risk employees           | 1                |
| MEDIUM risk employees         | 31               |
| Records flagged for HR review | 37               |
| Skills inferred from role     | 91 records       |

-Key Columns in Dataset B
canonical_id              → Unique ID after identity resolution
job_title_normalized      → Standardized role (6 canonical roles)
job_title_raw             → Original title from source system
skills_normalized         → Pipe-separated skills (CRM|Negotiation|...)
skills_source             → sourced / partial_enriched / inferred_from_role_title
attrition_risk_score      → 0.0 – 1.0
risk_tier                 → HIGH / MEDIUM / LOW
data_confidence_score     → 0.0 – 1.0
review_flag               → True if confidence < 0.60
review_reason             → Reason(s) for flag (pipe-separated)

The score is calculated using behavioral proxies, replacing Oracle Payroll data which was blocked due to legal constraints (Day 30 constraint):

Base score: 0.05

+ 0.25  if avg performance < 3.2
+ 0.10  if avg performance < 3.8
+ 0.20  if manager_changes >= 3
+ 0.20  if tenure >= 4 years AND no promotion
+ 0.15  if absent_days_last_year > 15
+ 0.10  if tenure < 2 years (new employee)

Tier:
  >= 0.65 → HIGH
  >= 0.40 → MEDIUM
  <  0.40 → LOW
-Data Confidence Score
Base: 1.0

- 0.30  if no skills data
- 0.15  if skills < 3 (partial)
- 0.25  if no performance history
- 0.10  if no location
- 0.10  if join date conflicts

< 0.60 → review_flag = True → score hidden in UI
 Data Architecture (5 Layers)
[Source Systems]
  SAP SuccessFactors (HRIS) · Workable (ATS) · Moodle (LMS) · Custom Perf Tool
  ❌ Oracle Payroll — BLOCKED (privacy risk, legal constraint, Day 30)
        ↓
[Ingestion Layer]
  Batch CSV/API export · Schema mapping · Raw landing zone · Audit log
        ↓
[Processing Layer]
  Identity resolution (Jaro-Winkler fuzzy matching)
  Job title normalization → 6 canonical roles
  Skill inference (4-tier: sourced → enriched → inferred → review)
  Confidence scoring per record
        ↓
[Feature Store]
  Employee 360 record · Skill vector · Behavioral signals · Confidence matrix
        ↓
[AI Output]
  Skill gap score · Directional attrition signal · Confidence-aware decision surface
-AI Tools Used
Tool	Purpose
Claude (Anthropic): data architecture design, prototype code generation, synthetic data logic
