<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="assets/banner-light.svg">
  <img alt="Saran Teja Mallela — Data Engineer" src="assets/banner-light.svg" width="100%">
</picture>

<p align="center">
  <a href="https://www.linkedin.com/in/saranteja2002"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white" /></a>&nbsp;
  <a href="mailto:stmallela.us@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=flat-square&logo=gmail&logoColor=white" /></a>&nbsp;
  <a href="https://github.com/Nerdboss-stm"><img src="https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white" /></a>&nbsp;
  <a href="https://www.researchgate.net/publication/372388571"><img src="https://img.shields.io/badge/ResearchGate-00CCBB?style=flat-square&logo=researchgate&logoColor=white" /></a>&nbsp;
  <a href="https://ieeexplore.ieee.org/document/10101155"><img src="https://img.shields.io/badge/IEEE-00629B?style=flat-square&logo=ieee&logoColor=white" /></a>
</p>

---

M.S. Data Science, **University of Houston** (4.0 GPA) · B.Tech CS · **3 published papers** (IEEE, ResearchGate) · **AWS Solutions Architect** certified.

I build end-to-end data platforms — from Kafka ingestion to dimensional models to serving layers. The three flagship projects below use **deliberately different architectures** because their data behaves differently. Lambda for stateful order lifecycles. Kappa for append-only sensor streams. Multi-agent pipeline for clinical AI. Data Vault where schemas conflict. Star where dimensions are flat. Snowflake where hierarchies run deep.

Choosing the right pattern matters more than knowing every tool.

Currently **Software Engineer at Url Systems Inc** · Previously University of Houston.

---

## Architecture Portfolio

Three production-grade platforms. Different architectures, different modeling, different cloud providers — each chosen for a reason.

<table>
<tr>
<td width="33%" valign="top">

### 🍔 GhostKitchen

[![Repo](https://img.shields.io/badge/repo-ghostkitchen-181717?style=flat-square&logo=github)](https://github.com/Nerdboss-stm/ghostkitchen)
![Lambda](https://img.shields.io/badge/Lambda_Architecture-0969da?style=flat-square)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=flat-square)

**Real-Time Dark Kitchen Intelligence Platform**

50 kitchens · 10 cities · 3–5 brands per kitchen  
Orders from Uber Eats, DoorDash, OwnApp  
Kitchen IoT · Delivery GPS · Menu CDC

</td>
<td width="33%" valign="top">

### 🫀 PulseTrack

[![Repo](https://img.shields.io/badge/repo-pulsetrack-181717?style=flat-square&logo=github)](https://github.com/Nerdboss-stm/pulsetrack)
![Kappa](https://img.shields.io/badge/Kappa_Architecture-0969da?style=flat-square)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square)

**Wearable Health Analytics & Anomaly Detection**

100K patients · 4 device types · 3 data sources  
Wearable telemetry · EHR FHIR · Pharmacy CDC  
HIPAA-compliant · Personal baseline anomaly detection

</td>
<td width="33%" valign="top">

### 🧬 HERA v4

[![Repo](https://img.shields.io/badge/repo-hera--healthcare--ai-181717?style=flat-square&logo=github)](https://github.com/Nerdboss-stm/hera-healthcare-ai)
![Pipeline](https://img.shields.io/badge/9--Stage_Pipeline-0969da?style=flat-square)
![AWS](https://img.shields.io/badge/AWS_App_Runner-FF9900?style=flat-square)
![Tests](https://img.shields.io/badge/104_tests-brightgreen?style=flat-square)

**Healthcare Reasoning & Analytics Platform**

Multi-agent clinical AI + data engineering  
23 FastAPI endpoints · 7 DE systems · 9-stage pipeline  
Live on AWS · Prometheus + Grafana · FHIR R4

</td>
</tr>
</table>

### Why different architectures?

```
                    GhostKitchen                   PulseTrack                     HERA v4
                    ──────────────                 ──────────────                 ──────────────
  Data nature       Stateful (order lifecycle)     Append-only (sensor readings)  Multi-stage AI pipeline
  Architecture      Lambda (batch + streaming)     Kappa (streaming only)         9-stage Command Center
  Why?              Orders get cancelled/refunded  Heart rate of 72 bpm doesn't   Clinical reasoning needs
                    → need batch recomputation     get "corrected" — one          sequential stages: NER →
                    for exact revenue numbers      pipeline handles everything    agents → ML → RAG → eval

  Silver model      Data Vault 2.0                 Normalized + metric explosion  Star schema warehouse
  Gold model        Star Schema                    Snowflake Schema               Hourly aggregations

  Late data         DLQ → nightly reconciliation   Same pipeline, no DLQ          DLQ in event streaming
  Airflow role      Conductor (orchestrates batch)  Janitor (maintenance only)    Custom 16-task ETL DAG
  Cloud             AWS (S3, Redshift, DynamoDB)   Azure (Blob, Cosmos DB)        AWS App Runner + ECR
  Serving           Metabase                       Apache Superset                Grafana + Prometheus
```

---

### GhostKitchen — Deep Dive

<details>
<summary><b>Lambda Architecture — dual-path data flow</b></summary>
<br/>

```mermaid
flowchart LR
    subgraph SRC["Data Sources"]
        O["Orders\n3 platforms\n500/min"]
        S["IoT Sensors\n2,000/min"]
        G["GPS Pings\n5,000/min"]
        M["Menu CDC\n200/day"]
        F["Feedback\nBatch CSV"]
    end

    subgraph SPEED["⚡ Streaming Path — Speed Layer"]
        K[Apache Kafka]
        SS[Spark Structured\nStreaming]
        B1[Bronze\nDelta Lake]
        S1[Silver\nData Vault]
        G1["Gold\nStar Schema\n(approximate)"]
    end

    subgraph BATCH["📦 Batch Path — Truth Layer"]
        AF[Airflow\nDaily 4AM]
        SB[Spark Batch]
        S2[Silver\nData Vault]
        G2["Gold\nStar Schema\n(exact)"]
    end

    subgraph SERVE["Serving"]
        MB[Metabase]
        DDB[DynamoDB API]
        RS[Redshift SQL]
    end

    O --> K
    S --> K
    G --> K
    M --> K
    K --> SS --> B1 --> S1 --> G1

    F --> AF
    B1 --> AF --> SB --> S2 --> G2

    G2 -- "Reconciliation 6AM\nbatch overwrites streaming" --> G1

    G1 --> MB
    G1 --> DDB
    G1 --> RS
```

**Why Lambda here?** Orders transition through states: `placed → confirmed → preparing → ready → picked_up → delivered` (or cancelled at any stage). A delivered order might be refunded 2 hours later, changing final revenue. The streaming path gives dashboards fast-but-approximate numbers. The daily batch path reprocesses 48 hours of Bronze to produce exact numbers and overwrites the streaming Gold. Delta Lake MERGE makes this atomic overwrite safe.

</details>

<details>
<summary><b>Data Vault 2.0 (Silver) — why not Star Schema here?</b></summary>
<br/>

Three platforms define the same entities differently:

| Field | Uber Eats | DoorDash | OwnApp |
|-------|-----------|----------|--------|
| Customer ID | `customer_uid` | `dasher_customer_id` | `user_id` |
| Order total | `total_amount` (float) | `order_value` (float) | `amount_cents` (integer!) |
| Timestamp | `order_timestamp` | `created_at` | `timestamp` |

Star Schema would force premature alignment. Data Vault solves this:
- **Hubs** unify identity — one `hub_customer` row per real person, regardless of platform
- **Satellites** preserve source-specific attributes with full SCD2 history
- **Links** capture relationships (which order → which customer → which kitchen+brand)

Gold then transforms Data Vault → Star Schema for analyst-friendly 2-table joins.

**Key tables:**

```
HUBS                    LINKS                       SATELLITES
hub_customer            link_order_customer          sat_customer_profile (SCD2)
hub_order               link_order_kitchen_brand     sat_order_details
hub_kitchen                                          sat_order_status (state machine)
hub_menu_item                                        sat_menu_item_details (SCD2)
```

</details>

<details>
<summary><b>Identity resolution — 3 platforms → 1 customer</b></summary>
<br/>

```
customer_identity_bridge
├── platform: uber_eats / doordash / own_app
├── platform_customer_id: ue_cust_44821
├── email_hash: MD5(normalized email)
├── phone_hash: MD5(phone) — nullable
├── match_confidence: 1.0 (exact) ... 0.6 (fuzzy)
└── match_method: exact_email / fuzzy_name_address / manual_override
```

**Algorithm:**
1. Normalize emails (lowercase, trim)
2. Group all platform IDs by `email_hash`
3. Assign one `customer_key` per email group
4. Null emails → fuzzy match on `name + delivery_address`
5. Store `match_method` + `confidence` for auditability

A customer ordering from Uber Eats AND DoorDash AND the direct app resolves to **one row** in `dim_customer`. Their cross-platform behavior is now visible: average order value per platform, platform loyalty, cannibalization analysis.

</details>

<details>
<summary><b>Dimensional model (Gold) — fact tables & grains</b></summary>
<br/>

| Fact Table | Grain | Why this grain |
|------------|-------|---------------|
| `fact_order` | 1 row per order (final state) | CEO needs: count, revenue, delivery time |
| `fact_order_state_history` | 1 row per order × status change | Ops needs: bottleneck analysis per state |
| `fact_sensor_hourly` | 1 row per kitchen × sensor × zone × hour | 1.3M raw events/day → hourly rollup for dashboards. Atomic stays in Silver for ML. |
| `fact_delivery_trip` | 1 row per delivery | Aggregated from raw GPS pings. Pings too granular for dashboards. |

**Dual-grain strategy:** Atomic events in Silver for correctness + pre-aggregated in Gold for performance. Every fact table has a deliberate grain decision documented.

**SCD types used:** Type 0 (zones — never change) · Type 1 (kitchens — overwrite) · Type 2 (menu items, customers — full history)

**Bridge tables:** `bridge_kitchen_brand` (M:M — one kitchen runs multiple brands) · `customer_identity_bridge` (cross-platform resolution)

</details>

---

### PulseTrack — Deep Dive

<details>
<summary><b>Kappa Architecture — single streaming engine</b></summary>
<br/>

```mermaid
flowchart LR
    subgraph SRC["Data Sources"]
        W["Wearable Devices\n4 types"]
        P["Pharmacy\nCDC Events"]
        E["EHR\nFHIR Batches"]
    end

    subgraph KAPPA["🔄 Single Streaming Pipeline"]
        K[Apache Kafka]
        SS["Spark Structured\nStreaming"]
        B[Bronze\nDelta Lake]
        SL["Silver\nNormalized\nMetric Explosion"]
        GL["Gold\nSnowflake Schema"]
    end

    subgraph MAINT["🔧 Airflow — Maintenance Only"]
        IR["Identity Resolution\nDaily 2AM"]
        DQ["Data Quality\nEvery 6h"]
        CM["Compaction\nDaily"]
        HA["HIPAA Audit\nWeekly"]
    end

    subgraph SERVE["Serving"]
        SU[Superset]
        CO[Cosmos DB API]
        AZ[Azure Functions\nAlerts]
    end

    W --> K
    P --> K
    K --> SS --> B --> SL --> GL
    E --> B

    SL -.-> IR
    SL -.-> DQ
    GL -.-> CM
    GL -.-> HA

    GL --> SU
    GL --> CO
    GL --> AZ
```

**Why Kappa here?** A heart rate reading of 72 bpm at 2:34pm doesn't get "corrected" later. It's an immutable measurement. No need for Lambda's dual-pipeline complexity. One streaming engine handles real-time, late-arriving data (30% of events arrive hours late from batch sync), AND backfills (reset Kafka offsets, replay through same code). Same pipeline, same logic, one truth.

**Airflow's role is completely different here.** In GhostKitchen, Airflow is the **conductor** — it orchestrates the batch ETL path. In PulseTrack, Airflow is the **janitor** — it handles maintenance (identity resolution refresh, data quality sweeps, Delta compaction, HIPAA audits) while the streaming pipeline runs continuously on its own.

</details>

<details>
<summary><b>Metric explosion — the core Silver transformation</b></summary>
<br/>

Bronze stores one row per device reading with nested metrics:
```json
{"device_id": "SW-001", "metrics": {"heart_rate_bpm": 72, "spo2_pct": 98, "hrv_ms": 45}}
```

Silver **explodes** this to one row per metric:
```
SW-001 | heart_rate_bpm | 72.0 | ✅ valid
SW-001 | spo2_pct       | 98.0 | ✅ valid
SW-001 | hrv_ms         | 45.0 | ✅ valid
```

Why? Anomaly detection, dashboards, and aggregations all operate on individual metrics. Nested JSON forces complex extraction on every query. Explosion is done once in Silver, benefitting every downstream consumer.

**Quality flags (not dropped):**

| Metric | Valid Range | Outside range → |
|--------|-------------|-----------------|
| heart_rate_bpm | 30–220 | `is_valid = false` (kept for anomaly detection) |
| spo2_pct | 70–100 | `is_valid = false` |
| blood_glucose_mgdl | 40–400 | `is_valid = false` |
| skin_temp_celsius | 30–42 | `is_valid = false` |

Invalid readings are **never dropped** — they're flagged and kept because out-of-range values may indicate genuine medical events that anomaly detection should evaluate.

</details>

<details>
<summary><b>Snowflake Schema (Gold) — why not Star here?</b></summary>
<br/>

Healthcare dimensions are **deeply hierarchical:**
- ICD-10 codes: Chapter → Category → Specific Code (3 levels)
- Medications: Drug Class → Medication → Dosage → Patient Assignment
- Devices: Device Type → Firmware Version → Patient Reassignment

Star Schema would create a "kitchen sink" `dim_patient` with 50+ columns. Snowflake normalizes these hierarchies into separate dimension tables with foreign keys.

**Fact tables:**

| Table | Grain |
|-------|-------|
| `fact_vital_reading` | 1 row per patient × metric × timestamp (atomic) |
| `fact_vital_daily_summary` | 1 row per patient × metric × day (pre-aggregated) |
| `fact_activity_session` | 1 row per exercise session |
| `fact_lab_result` | 1 row per lab test |
| `fact_prescription_fill` | 1 row per pharmacy dispense |
| `fact_anomaly_alert` | 1 row per detected anomaly |

**Dimension tables with SCD types:**

| Dimension | SCD | Why |
|-----------|-----|-----|
| `dim_patient` | Type 2 | Lean — no conditions/meds crammed in |
| `dim_medication` | Type 2 | Hardest SCD2: start/stop/dosage change/restart |
| `dim_condition` → `dim_condition_category` | Type 2 → Type 0 | ICD-10 hierarchy normalized |
| `dim_device` | Type 2 | Firmware versions + patient reassignment |
| `dim_drug_class` | Type 0 | Classification doesn't change |

</details>

<details>
<summary><b>Identity resolution — 7 identifier types → 1 patient</b></summary>
<br/>

The hardest engineering problem in PulseTrack. A single patient may appear as:

```
device_account_id  →  email  →  hospital_mrn  →  pharmacy_id  →  insurance_id  →  phone_hash  →  ssn_hash
```

**Multi-hop graph matching:** `device_account` links to `email`, `email` links to `hospital_mrn`, `hospital_mrn` links to `pharmacy_id` — all resolving to one unified `patient_key` via the `patient_identity_bridge` table (1,159 rows for 100K user dataset).

This is significantly more complex than GhostKitchen's 3-platform identity resolution because healthcare identifiers are fragmented across completely independent systems (hospitals, pharmacies, insurers, device manufacturers) with no shared login.

</details>

<details>
<summary><b>Anomaly detection — personal baselines, not population thresholds</b></summary>
<br/>

A runner with resting HR 52 showing HR 85 is **more concerning** than a sedentary person at HR 85. Population-level thresholds miss this.

**Per-patient rolling statistics** (mean, σ for each metric over 30 days). New reading compared against personal baseline. Deviation > 2σ = anomaly alert.

- **State:** Keyed by `(patient_key, metric_name)`. Each key stores running stats + circular buffer.
- **Backed by:** RocksDB state store in Spark Structured Streaming.
- **TTL:** 90 days for inactive patients.
- **Output:** `fact_anomaly_alert` in Gold → Azure Functions → email/SMS notifications.

</details>

<details>
<summary><b>HIPAA compliance — built in, not bolted on</b></summary>
<br/>

- **De-identification:** Gold uses `age_bracket` (not exact age), `city` (not address), no names
- **Encryption:** Delta Lake files encrypted at rest
- **Access control:** `data_engineer` (all layers), `clinical_analyst` (Gold only, PII masked), `researcher` (Gold, fully de-identified)
- **Deletion:** Delta Lake DELETE across all layers → verification sweep → audit log
- **Retention:** HIPAA requires 7-year retention. Patient deletion = de-identify PII but keep analytical record.
- **Audit:** Weekly `dag_hipaa_audit` scans for PII leakage across all layers

</details>

---

### HERA v4 — Deep Dive

[![CI/CD](https://github.com/Nerdboss-stm/hera-healthcare-ai/actions/workflows/ci.yml/badge.svg)](https://github.com/Nerdboss-stm/hera-healthcare-ai/actions)
[![Repo](https://img.shields.io/badge/GitHub-hera--healthcare--ai-181717?style=flat-square&logo=github)](https://github.com/Nerdboss-stm/hera-healthcare-ai)
[![Live Demo](https://img.shields.io/badge/Live_Demo-AWS_App_Runner-FF9900?style=flat-square)](https://3wihdymitc.us-east-1.awsapprunner.com)
![Tests](https://img.shields.io/badge/104_tests_passing-brightgreen?style=flat-square)
![Endpoints](https://img.shields.io/badge/23_REST_endpoints-0969da?style=flat-square)

Production-grade healthcare AI platform in two layers: **AI Layer** (multi-agent clinical reasoning, RAG, biomedical NER, risk prediction, FHIR R4) and **Data Engineering Layer** (event streaming, column-level lineage, data quality, star schema warehouse, ETL, CDC, data catalog). Both unified through a **Command Center** that runs all 9 stages as a single pipeline per patient encounter. Deployed on **AWS App Runner** via GitHub Actions CI/CD.

<details>
<summary><b>9-stage Command Center pipeline</b></summary>
<br/>

```mermaid
flowchart TD
    INPUT["Patient Vitals + Clinical Notes + FHIR Bundle"]
    INPUT --> S1["Stage 1: NER Extraction\n33 meds · 28 conditions · 25 procedures"]
    S1 --> S2["Stage 2: Knowledge Graph\nNetworkX DiGraph · 3 edge types\n20 treatment · 5 contraindication · 9 diagnostic"]
    S1 --> S3
    S2 --> S3["Stage 3: Multi-Agent Reasoning"]

    subgraph AGENTS["Three Agents — Sequential"]
        T["Triage Agent\nESI v4 · 6 vital thresholds"]
        D["Diagnostic Agent\nICD-10 · 6 complaint categories"]
        TR["Treatment Agent\n8 protocols · drug interactions\nallergy cross-reference"]
        T --> D --> TR
    end

    S3 --> AGENTS
    AGENTS --> S4["Stage 4: Risk Prediction\nRandom Forest + SHAP\n9 features · explainable"]
    AGENTS --> S5["Stage 5: RAG Retrieval\nFAISS + MiniLM-L6-v2\n15 curated guidelines"]
    S5 --> S6["Stage 6: Clinical Summarization\nT5 Transformer · beam search\nauto-fallback to pretrained"]
    S6 --> S7["Stage 7: Safety Evaluation\n4-axis LLM-as-Judge\nfactual · hallucination · accuracy · safety"]
    S7 -- "score < threshold" --> S6
    S7 --> S8["Stage 8: FHIR R4 Export\nPatient · Observation · RiskAssessment\nLOINC-coded vitals"]
    S4 --> S9
    S8 --> S9["Stage 9: Data Engineering\n7 integrated systems"]
```

One `POST /api/command-center` runs all 9 stages per patient encounter. No manual orchestration. Stage 7 (Safety Evaluation) has a **feedback loop** — if the score is below threshold, it triggers re-summarization with RAG context injection.

**Consensus score:** `0.3 × risk + 0.4 × confidence + 0.3 × evidence_grade`

</details>

<details>
<summary><b>Multi-agent clinical reasoning — why agents, not a monolithic LLM</b></summary>
<br/>

Clinical reasoning is multi-step. A single LLM call can't provide the auditability, separation of concerns, and protocol adherence required in healthcare.

| Agent | Input | Output | Method |
|-------|-------|--------|--------|
| **Triage (ESI v4)** | Vitals + complaint | ESI level 1–5 | 6 vital thresholds × 4 levels + high-acuity keywords + resource estimation |
| **Diagnostic (ICD-10)** | Triage + NER entities | Differential diagnoses with probabilities | Evidence overlap − rule-out penalty + acuity boost + age factor |
| **Treatment** | Diagnosis + patient history | Treatment plan | 8 protocols by ICD-10 + drug interaction checking (5 pairs) + allergy cross-reference |

The orchestrator chains all three and records a **full audit trail** (JSONB in PostgreSQL). Every decision is traceable.

**Key design decision:** RAG over fine-tuning for medical knowledge. Guidelines change — RAG enables hot-swap without retraining, and citations provide provenance.

</details>

<details>
<summary><b>Data Engineering layer — 7 systems, zero external dependencies</b></summary>
<br/>

Each system is ~200–500 lines of focused Python implementing production-faithful patterns:

| System | Lines | What it does |
|--------|-------|-------------|
| **Event Streaming** | 422 | Kafka-style with schema registry (5 schemas), MD5 partitioning, consumer groups, DLQ |
| **Column-Level Lineage** | 506 | DAG tracking 36+ nodes across 8 pipeline stages, `impact_analysis()`, PII flagging |
| **Data Quality** | 349 | 12 checks across 5 categories (schema, completeness, accuracy, consistency, freshness) |
| **Star Schema Warehouse** | 394 | `fact_clinical_encounters` (22 cols) + 4 dimensions + hourly aggregation |
| **ETL Orchestrator** | 437 | 16-task DAG with topological sort, retry, SLA monitoring, upstream failure propagation |
| **CDC** | 246 | Before/after snapshots, SHA-256 checksums, field-level diffs, event replay |
| **Data Catalog** | 499 | 12 datasets with PII tracking, freshness SLAs, relevance-scored search |

**Why custom over Airflow/Kafka/dbt?** Zero external dependencies. Production-faithful patterns that run with just `pip install`. Demonstrates understanding of the internals, not just tool configuration.

**ETL DAG execution flow:**
```
ingest → validate → extract_entities → build_kg
                  → run_triage → run_diagnosis → run_treatment
                  → predict_risk
                                  run_diagnosis → retrieve_rag → generate_summary → evaluate_safety
                                                                                          ↓
export_fhir ← [run_treatment, predict_risk, generate_summary, evaluate_safety]
    ├── load_warehouse → emit_cdc → update_catalog
    └── track_lineage → update_catalog
```

</details>

<details>
<summary><b>Safety evaluation — LLM-as-Judge over ROUGE</b></summary>
<br/>

ROUGE can't detect hallucinations or clinically dangerous errors. HERA uses a 4-axis evaluation:

| Axis | Weight | Method |
|------|--------|--------|
| Factual consistency | 0.30 | Keyword overlap + 7 contradiction pair detection |
| Hallucination | 0.25 | Entity precision/recall + fabricated claim detection |
| Medical accuracy | 0.25 | 157 valid terms + dangerous dosage checks (5 drugs) |
| Clinical safety | 0.20 | 3 dangerous regex patterns + allergy cross-check |

Overall score = weighted sum. If below threshold → **feedback loop** re-triggers summarization with RAG context.

</details>

<details>
<summary><b>Infrastructure — deployed, monitored, tested</b></summary>
<br/>

**Deployment pipeline:**
```
Push to main → GitHub Actions CI (lint + 104 tests + Docker build) → ECR → AWS App Runner auto-deploys
```

**Observability (Prometheus + Grafana):**
- 17-panel Grafana dashboard (auto-provisioned): System Health, Clinical Risk & Triage, API Performance, Pipeline & Data Engineering
- Prometheus scrapes `/metrics` every 5s: `hera_requests_total`, `hera_request_failures_total`, `hera_request_latency_seconds`
- Alert rule: `TooManyHighRisk` fires when `high_risk_predictions > 3` for 30s

**PostgreSQL** (5 tables): `patient_predictions`, `summaries`, `clinical_reasoning_sessions`, `ner_extractions`, `evaluation_reports` — all populated automatically per API call.

**Middleware:** API key auth + rate limiting (30/min anon, 60/min auth) + HIPAA audit trail with trace IDs.

**Cost:** AWS App Runner 1 vCPU, 2GB RAM, auto-scales to 0 → ~$5–7/mo.

</details>

<details>
<summary><b>Key design decisions</b></summary>
<br/>

| Decision | Rationale |
|----------|-----------|
| Multi-agent over monolithic LLM | Clinical reasoning is multi-step. Agents provide separation of concerns and auditability. |
| RAG over fine-tuning for knowledge | Medical guidelines change. RAG enables hot-swap without retraining. Citations provide provenance. |
| FHIR R4 for interoperability | Mandated for US healthcare data exchange (21st Century Cures Act). |
| Random Forest + SHAP over deep learning | 9 features, binary target. "SpO2 and MAP drove this High Risk" is more valuable than marginal accuracy from a black box. |
| Column-level over table-level lineage | Field-level provenance for HIPAA compliance and impact analysis. Table-level is too coarse for healthcare governance. |
| Custom DE systems over Airflow/Kafka/dbt | Demonstrates understanding of internals. Zero dependencies. Production-faithful patterns in ~300 lines each. |
| Unified Command Center over microservices | One POST runs all 9 stages. Atomic processing. Decompose into microservices at 100K+ patients/day. |

</details>

---

## Other Projects

| Project | Stack | What it does |
|---------|-------|-------------|
| [**Real-Time Stock Pipeline**](https://github.com/Nerdboss-stm/Real-Time-Stock-Price-Pipeline) | Kafka · Spark · Airflow · S3 · Redshift · Tableau | Streaming stock market data → cloud warehouse → analytics dashboards |
| [**Quantum Image Processing**](https://github.com/Nerdboss-stm/Quantum-Computing-in-Image-Processing) | Qiskit · Python · Jupyter | Quantum computing approaches to classical image processing problems |
| [**Financial News Sentiment**](https://github.com/Nerdboss-stm/Financial-News-Sentiment-Classifier) | NLP · Python | Sentiment classification for financial news articles |

---

## Tech Stack

**Streaming & Ingestion**&nbsp;&nbsp;
![Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white)
![Spark Streaming](https://img.shields.io/badge/Spark_Structured_Streaming-E25A1C?style=flat-square&logo=apachespark&logoColor=white)
![CDC](https://img.shields.io/badge/CDC-444?style=flat-square)

**Batch Processing**&nbsp;&nbsp;
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat-square&logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=flat-square&logo=delta&logoColor=white)

**Orchestration**&nbsp;&nbsp;
![Airflow](https://img.shields.io/badge/Apache_Airflow-017CEE?style=flat-square&logo=apacheairflow&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

**Data Modeling**&nbsp;&nbsp;
![Data Vault](https://img.shields.io/badge/Data_Vault_2.0-0969da?style=flat-square)
![Star Schema](https://img.shields.io/badge/Star_Schema-0969da?style=flat-square)
![Snowflake Schema](https://img.shields.io/badge/Snowflake_Schema-0969da?style=flat-square)
![SCD](https://img.shields.io/badge/SCD_0%2F1%2F2-0969da?style=flat-square)
![Medallion](https://img.shields.io/badge/Medallion_Architecture-0969da?style=flat-square)

**Data Quality**&nbsp;&nbsp;
![Great Expectations](https://img.shields.io/badge/Great_Expectations-FF6D00?style=flat-square)
![Data Contracts](https://img.shields.io/badge/Data_Contracts-444?style=flat-square)
![HIPAA](https://img.shields.io/badge/HIPAA_Compliance-444?style=flat-square)

**Cloud**&nbsp;&nbsp;
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonwebservices&logoColor=white)
![S3](https://img.shields.io/badge/S3-569A31?style=flat-square&logo=amazons3&logoColor=white)
![Redshift](https://img.shields.io/badge/Redshift-8C4FFF?style=flat-square&logo=amazonredshift&logoColor=white)
![DynamoDB](https://img.shields.io/badge/DynamoDB-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)
![Lambda](https://img.shields.io/badge/Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)
![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat-square&logo=databricks&logoColor=white)
![Cosmos DB](https://img.shields.io/badge/Cosmos_DB-0078D4?style=flat-square)

**Storage**&nbsp;&nbsp;
![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=flat-square&logo=delta&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?style=flat-square&logo=minio&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)

**Serving & BI**&nbsp;&nbsp;
![Metabase](https://img.shields.io/badge/Metabase-509EE3?style=flat-square&logo=metabase&logoColor=white)
![Superset](https://img.shields.io/badge/Apache_Superset-20A6C9?style=flat-square&logo=apachesuperset&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-E97627?style=flat-square&logo=tableau&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat-square&logo=powerbi&logoColor=333)

**ML & AI**&nbsp;&nbsp;
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)
![BioGPT](https://img.shields.io/badge/BioGPT-444?style=flat-square)
![LSTM](https://img.shields.io/badge/LSTM-444?style=flat-square)

**Languages**&nbsp;&nbsp;
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-336791?style=flat-square)
![C++](https://img.shields.io/badge/C++-00599C?style=flat-square&logo=cplusplus&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=333)

**Infrastructure & APIs**&nbsp;&nbsp;
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000?style=flat-square&logo=flask&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)

---

## Publications

| Paper | Venue | Link |
|-------|-------|------|
| Classification of Parkinson's Disease in Brain MRI Images Using Deep Residual CNN | ResearchGate | [Read →](https://www.researchgate.net/publication/372388571_CLASSIFICATION_OF_PARKINSON'S_DISEASE_IN_BRAIN_MRI_IMAGES_USING_DEEP_RESIDUAL_CONVOLUTIONAL_NEURAL_NETWORK) |
| Efficient Smart Micro-Scale Solar Power Management System for Rechargeable Nodes | IEEE | [Read →](https://ieeexplore.ieee.org/document/10101155) |
| Estimation of Periodontal Bone Loss Using SVM and Random Forest | IEEE | [Read →](https://ieeexplore.ieee.org/document/9835869) |

---

## Certifications

| Certification | Issuer |
|--------------|--------|
| Databricks Data Engineer Associate | Databricks |
| AWS Certified Solutions Architect | Amazon Web Services |
| Tableau Desktop Specialist | Tableau / Salesforce |
| UiPath Automation Developer Associate | UiPath |


---

## Education

| Degree | Institution | GPA |
|--------|------------|-----|
| **M.S. Engineering Data Science** | University of Houston (2023–2025) | **4.0 / 4.0** |
| **B.Tech Computer Science** | VR Siddhartha Engineering College | |

---

## GitHub Activity

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/Nerdboss-stm/Nerdboss-stm/blob/output/github-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/Nerdboss-stm/Nerdboss-stm/blob/output/github-snake.svg" />
  <img alt="contribution snake animation" src="https://github.com/Nerdboss-stm/Nerdboss-stm/blob/output/github-snake.svg" width="100%" />
</picture>

<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=nerdboss-stm&show_icons=true&theme=default&hide_border=true&include_all_commits=true&count_private=true" width="48%" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=nerdboss-stm&layout=compact&theme=default&hide_border=true&langs_count=8" width="48%" />
</p>
<p align="center">
  <img src="https://streak-stats.demolab.com?user=nerdboss-stm&theme=default&hide_border=true" width="55%" />
</p>

---

<p align="center">
  <a href="mailto:stmallela.us@gmail.com"><b>stmallela.us@gmail.com</b></a>&nbsp;&nbsp;·&nbsp;&nbsp;
  <a href="https://www.linkedin.com/in/saranteja2002"><b>LinkedIn</b></a>&nbsp;&nbsp;·&nbsp;&nbsp;
  <a href="https://github.com/Nerdboss-stm"><b>GitHub</b></a>
</p>
