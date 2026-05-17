# Part 4 — AI Solution Design for a Business Problem

![Domain](https://img.shields.io/badge/Domain-Healthcare-red) ![Task](https://img.shields.io/badge/AI%20Task-Multi--Class%20Classification-purple) ![Type](https://img.shields.io/badge/Type-Solution%20Design-blue) ![License](https://img.shields.io/badge/License-MIT-brightgreen)

---

## 📌 Project Overview

This project presents a **complete AI solution design** for automating patient triage in a hospital Emergency Department. Rather than building a toy model, it delivers a **production-grade AI blueprint** covering problem definition, data strategy, model architecture, evaluation plan, and responsible AI framework.

| Item | Detail |
|---|---|
| **Domain** | Healthcare — Emergency Department |
| **Problem** | Manual triage has 6.2% under-triage rate and 28 min wait for Level 2 patients |
| **AI Solution** | Hybrid NLP + tabular classifier for ESI Triage Level prediction (1–5) |
| **Primary Model** | DistilBERT (text) + Dense network (vitals) → late-fusion ensemble |
| **Datasets Used** | `business_kpi_sample.csv` · `ai_usecase_reference_catalog.csv` |

---

## 📂 Repository Structure

```
part-4-ai-solution-design/
│
├── README.md                              ← You are here
├── solution_report.md                     ← ✅ Full 8-task written report
├── requirements.txt
├── business_kpi_sample.csv
├── ai_usecase_reference_catalog.csv
│
└── diagrams/
    ├── solution_architecture.png          ← ✅ System architecture diagram
    ├── kpi_dashboard.png                  ← Before vs After KPI comparison
    ├── evaluation_radar.png               ← Technical metric targets
    ├── roadmap_gantt.png                  ← 12-month implementation plan
    └── risk_matrix.png                    ← Responsible AI risk assessment
```

---

## 🏥 Task 1 — Business Domain: Healthcare

An urban hospital Emergency Department processes 400+ daily visits. Triage nurses manually assign Emergency Severity Index (ESI) levels 1–5. This process is error-prone under fatigue and high patient volumes.

---

## 🩺 Task 2 — Business Problem

| Pain Point | Current Metric |
|---|---|
| Under-triage rate | 6.2% — high-risk patients sent to wrong queue |
| Over-triage rate | 11.4% — unnecessary resource allocation |
| Time to physician (Level 2) | 28 minutes |
| 30-day readmission | 8.1% |

**Stakeholders:** Triage nurses · ED physicians · Hospital administrators · Patients · Payers

---

## 🤖 Task 3 — AI Task Type: Multi-Class Classification

Each patient presentation → one of 5 ESI levels. Multi-class classification matches the clinical framework, is interpretable to clinicians, and produces actionable routing decisions.

---

## 📋 Task 4 — Data Requirements

| Feature Group | Examples | Type |
|---|---|---|
| Vital signs | BP, HR, SpO₂, temperature, RR | Numerical |
| Chief complaint | Free text reason for visit | Unstructured |
| Patient demographics | Age, sex, BMI | Categorical/Numerical |
| Clinical context | Arrival mode, comorbidities, meds | Categorical |
| EHR history | Past ED visits, diagnoses | Numerical |

**Target:** ESI Level (1–5) from historical nurse assignments  
**Minimum:** 50,000 records (10,000 per class)

---

## 🧠 Task 5 — Model Architecture

```
Chief Complaint ──► DistilBERT ──► 768-dim embedding
                                         │
Vital Signs + Demographics ─────────►  Concatenate (768 + N features)
                                         │
                                     Dense(256, ReLU) → Dropout(0.3)
                                         │
                                     Dense(5, Softmax)
                                         │
                                  ESI Level 1–5 Prediction
```

**Three-tier deployment:**
- Month 1–2: XGBoost baseline (structured features only)
- Month 3–5: Hybrid DistilBERT + tabular ensemble
- Month 8+: LSTM waiting-room deterioration monitor

---

## 🎯 Task 6 — Evaluation Plan

### Technical Metrics
| Metric | Target |
|---|---|
| Recall — Level 1 (Immediate) | ≥ 99% |
| Recall — Level 2 (Emergent) | ≥ 95% |
| Adjacent accuracy (±1 level) | ≥ 94% |
| Overall Accuracy | ≥ 82% |
| Inference latency | < 200 ms |

### Business Metrics
| KPI | Current → Target |
|---|---|
| Under-triage rate | 6.2% → ≤ 1.5% |
| Time to physician | 28 min → ≤ 18 min |
| 30-day readmission | 8.1% → ≤ 5.5% |

---

## ⚖️ Task 7 — Responsible AI

| Risk | Mitigation |
|---|---|
| Demographic bias in labels | Quarterly stratified fairness audits |
| Over-confidence on rare presentations | OOD detector; threshold failsafe |
| Privacy breach (sensitive health data) | De-identification + DISHA compliance |
| Automation complacency | Nurse retains full authority; regular unaided drills |
| Model drift | Monthly monitoring; auto-alert if recall drops |
| Impact on nurses | Co-design; reduce workload framing, not replacement |

---

## 📄 Task 8 — One-Page Summary

See [`solution_report.md`](solution_report.md) — Task 8 section.

**Expected ROI:** 120 adverse events avoided/year = **₹18 Crore annual risk reduction**

---

## 🚀 How to Run the Analysis Notebook

```bash
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

The notebook loads the provided KPI and reference CSV files and generates all 5 diagrams in the `diagrams/` folder.

---

## 📚 References
- Vaswani et al. (2017). *Attention Is All You Need.* NeurIPS.
- Devlin et al. (2019). *BERT.* NAACL.
- Fernandes et al. (2020). *Clinical Decision Support for Triage.* EMJ.
- India DISHA Healthcare Data Act, Draft 2022.
