# AI Solution Design Report
## Intelligent Patient Triage and Risk Stratification — Healthcare Domain

**Part 4 | Module 5 — AI Solution Design for a Business Problem**  
**Author:** AI Business Analyst  
**Version:** 1.0

---

## Table of Contents
1. [Task 1 — Business Domain](#task-1--business-domain)
2. [Task 2 — Business Problem Definition](#task-2--business-problem-definition)
3. [Task 3 — AI Task Type](#task-3--ai-task-type)
4. [Task 4 — Data Requirement Plan](#task-4--data-requirement-plan)
5. [Task 5 — Model Recommendation](#task-5--model-recommendation)
6. [Task 6 — Evaluation Plan](#task-6--evaluation-plan)
7. [Task 7 — Responsible AI Considerations](#task-7--responsible-ai-considerations)
8. [Task 8 — Final Solution Summary (One Page)](#task-8--final-solution-summary)

---

## Task 1 — Business Domain

**Domain:** Healthcare — Hospital Emergency Department Operations

**Organisation Profile:**  
A mid-sized urban hospital handling 350–500 Emergency Department (ED) visits daily across three shifts. The hospital operates under chronic resource constraints — limited specialist availability, bed shortages, and high nurse-to-patient ratios.

**Why Healthcare?**

Healthcare is one of the highest-impact domains for AI: a single mis-triage decision can cost a human life. At the same time, manual triage is one of the most time-pressured, cognitively demanding tasks in medicine. AI that supports (not replaces) clinical judgment can save lives, reduce burnout, and reduce costs simultaneously.

---

## Task 2 — Business Problem Definition

### What Problem is Being Solved?

When a patient arrives at the ED, a **triage nurse** must rapidly assess urgency using the Manchester Triage System (MTS) or Emergency Severity Index (ESI) — assigning each patient to one of 5 priority levels (1 = immediate, 5 = non-urgent). This decision determines:
- How quickly they see a doctor
- Which resources are allocated
- Whether they are admitted or discharged

**Current process:** The triage nurse manually records vital signs (blood pressure, heart rate, oxygen saturation, temperature, respiratory rate), chief complaint, pain scale score, and patient history, then makes a clinical judgment — typically in 2–5 minutes.

### Stakeholders

| Stakeholder | Interest |
|---|---|
| Triage nurses | Reduce cognitive overload; avoid missed high-risk patients |
| ED physicians | Receive correctly prioritised patient queue |
| Hospital administrators | Reduce average length of stay; reduce adverse event liability |
| Patients | Faster, fairer, accurate triage |
| Insurance payers | Reduce unnecessary resource use |

### Current Process Limitations

| Limitation | Impact |
|---|---|
| **Human fatigue** | Triage accuracy degrades 25–30% after 6 hours without a break |
| **Subjectivity** | Inter-rater agreement on ESI levels is only ~60–70% for levels 2–3 |
| **Information overload** | Nurse cannot rapidly cross-reference patient history, comorbidities, and drug interactions |
| **Overcrowding blind spot** | During surges, triage is rushed — high-risk patients may be under-triaged |
| **Bias** | Studies show women, elderly, and minority patients are systematically under-triaged |
| **No real-time risk update** | Patient's condition may deteriorate in waiting room with no re-evaluation |

### Business Goals (KPIs)

| KPI | Current | Target |
|---|---|---|
| Under-triage rate (high-risk missed) | 6.2% | ≤ 1.5% |
| Over-triage rate (unnecessary Priority 1) | 11.4% | ≤ 5.0% |
| Time to physician (Priority 2 patients) | 28 min | ≤ 18 min |
| 30-day readmission rate | 8.1% | ≤ 5.5% |
| ED nurse triage time per patient | 4.2 min | ≤ 2.5 min |

---

## Task 3 — AI Task Type

### Primary Task: **Multi-Class Classification**

Each patient presentation is classified into one of 5 ESI triage levels:

- **Level 1 (Immediate):** Resuscitation required — e.g., cardiac arrest
- **Level 2 (Emergent):** High risk of deterioration — e.g., chest pain, stroke
- **Level 3 (Urgent):** Stable but needs multiple resources — e.g., abdominal pain
- **Level 4 (Less Urgent):** One resource needed — e.g., simple fracture
- **Level 5 (Non-Urgent):** Prescription refill, minor wound

### Secondary Task: **Anomaly Detection (Deterioration Alert)**

After initial triage, a secondary model monitors patients in the waiting room and flags any whose vital signs deviate significantly from their baseline — triggering immediate re-assessment.

### Why Multi-Class Classification?

| Alternative | Why Rejected |
|---|---|
| **Binary (urgent/not)** | Too coarse — loses critical distinction between Level 2 and Level 3 |
| **Regression (score)** | ESI levels are ordinal categories with clinical thresholds, not continuous |
| **Ranking** | Clinical protocols require hard category assignments for resource routing |
| **Object Detection** | Not an image task |

Multi-class classification maps naturally to the established ESI framework that all ED staff are trained on, ensuring AI outputs are immediately interpretable by clinicians.

---

## Task 4 — Data Requirement Plan

### Available Data Sources

| Data Source | Type | Structure | Accessibility |
|---|---|---|---|
| Electronic Health Records (EHR) | Patient history, comorbidities | Structured | HL7 FHIR API |
| Vital signs at triage | BP, HR, SpO₂, temp, RR | Structured (numerical) | Nurse workstation input |
| Chief complaint (free text) | Reason for visit | Unstructured text | Triage form |
| Pain scale (0–10) | Severity self-report | Numerical | Triage form |
| ESI level assigned (historical) | Target variable | Categorical (1–5) | EHR system |
| Lab results (if available pre-triage) | Blood counts, troponin | Structured | LIS integration |
| Disposition outcome | Admitted / discharged / ICU | Categorical | EHR |
| 30-day readmission flag | Re-attended within 30 days | Binary | EHR |

### Input Features (Model Inputs)

**Vital Signs (Numerical):**
- Systolic / diastolic blood pressure
- Heart rate (bpm)
- SpO₂ (oxygen saturation %)
- Temperature (°C)
- Respiratory rate (breaths/min)
- Pain score (0–10)
- Glasgow Coma Scale (GCS)

**Patient Demographics (Categorical/Numerical):**
- Age, sex
- BMI

**Clinical Context (Categorical):**
- Chief complaint (encoded via NLP)
- Arrival mode (walk-in, ambulance, transfer)
- Time of day, day of week
- Active comorbidities (diabetes, COPD, cardiac disease)
- Current medications count

**Historical (from EHR):**
- Number of ED visits in past 12 months
- Known allergies count
- Chronic condition flags

### Target Variable
- **ESI Level** (1–5) — historical nurse-assigned triage level as ground truth

### Minimum Data Requirements

| Requirement | Specification |
|---|---|
| Sample size | ≥ 50,000 triage events (10,000 per class minimum) |
| Class balance | Oversample Level 1 (rare) using SMOTE |
| Label quality | Only include triage events with confirmed final disposition |
| Temporal range | At least 3 years to capture seasonal variation |
| Missing rate | Impute vitals using median; flag for model confidence |

### Data Quality Risks

| Risk | Mitigation |
|---|---|
| Missing vital signs (5–15% of records) | Multiple imputation; flag to model as uncertainty |
| Subjective ESI assignment (label noise) | Use consensus labels; weight by senior nurse assignments |
| Data drift (new disease patterns post-COVID) | Rolling re-training window |
| PII in free-text complaints | De-identification pipeline before training |

---

## Task 5 — Model Recommendation

### Tier 1 — Baseline (Deploy Month 1–2)

**Gradient Boosted Trees (XGBoost)**

| Attribute | Detail |
|---|---|
| Input | Structured vital signs + demographics + encoded chief complaint |
| Training time | < 10 minutes |
| Inference | < 5 ms |
| Expected accuracy | 78–82% on ESI levels 2–4 |
| Advantage | Highly interpretable (SHAP values); no GPU needed |

### Tier 2 — Production Model (Deploy Month 3–5)

**Hybrid Architecture: NLP (DistilBERT) + Structured Tabular (XGBoost) → Ensemble**

```
Chief Complaint (text) ──► DistilBERT encoder ──► 768-dim embedding
                                                        │
Vital Signs + Demographics ──────────────────────────►  Concatenate
                                                        │
                                                    Dense(256, ReLU)
                                                    Dropout(0.3)
                                                        │
                                                    Dense(5, Softmax)
                                                        │
                                              ESI Level Prediction (1–5)
```

**Why this architecture?**
- Vital signs are tabular and well-handled by dense layers
- Chief complaint is free text — requires NLP (pre-trained DistilBERT captures medical terminology without domain-specific training data)
- Late fusion combines both modalities before the final classifier
- This mirrors how a human triage nurse integrates what they *read* (complaint) with what they *measure* (vitals)

### Tier 3 — Longitudinal Model (Month 8+)

**LSTM-based deterioration detector** — monitors sequential vital sign readings every 15 minutes for waiting patients. If sequence deviates significantly from initial assessment, triggers a re-triage alert.

### Technology Stack

| Component | Tool |
|---|---|
| Feature engineering | Python · Pandas · scikit-learn |
| NLP preprocessing | HuggingFace Tokenizers · spaCy (medical NER) |
| Model training | TensorFlow / PyTorch |
| Model serving | FastAPI + Docker |
| EHR integration | HL7 FHIR REST API |
| Monitoring | MLflow + Evidently AI |
| Explainability | SHAP + LIME |

---

## Task 6 — Evaluation Plan

### Technical Metrics

| Metric | Formula | Target | Note |
|---|---|---|---|
| **Overall Accuracy** | Correct / Total | ≥ 82% | All 5 levels |
| **Macro F1** | Mean F1 per class | ≥ 0.80 | Unweighted by class size |
| **Recall — Level 1** | TP₁ / (TP₁ + FN₁) | ≥ 99% | Missing a Level 1 = potential death |
| **Recall — Level 2** | TP₂ / (TP₂ + FN₂) | ≥ 95% | High-risk under-triage must be minimised |
| **Adjacent accuracy** | Pred within ±1 ESI level | ≥ 94% | Clinical leniency — ±1 is acceptable |
| **Inference latency** | Time to predict | < 200 ms | Must not slow triage workflow |

> **Critical design choice:** Levels 1 and 2 use a **low threshold (0.30)** for positive classification — accepting more false alarms (over-triage) to guarantee near-zero under-triage. Under-triage is catastrophic; over-triage is merely wasteful.

### Business Metrics

| Metric | Baseline | 12-Month Target | Measurement |
|---|---|---|---|
| Under-triage rate | 6.2% | ≤ 1.5% | Monthly audit |
| Over-triage rate | 11.4% | ≤ 5.0% | Monthly audit |
| Time to physician (Priority 2) | 28 min | ≤ 18 min | EHR timestamps |
| 30-day readmission | 8.1% | ≤ 5.5% | EHR follow-up |
| Triage time per patient | 4.2 min | ≤ 2.5 min | Workflow timer |
| Adverse events (in ED waiting) | Tracked | Reduce by 40% | Safety reports |

### Testing Protocol

1. **Offline evaluation** (Month 1–2): Train on 3-year historical data; evaluate on held-out 6-month test set. Report confusion matrix, per-class recall, adjacent accuracy.
2. **Shadow mode** (Month 3–4): AI runs silently alongside nurses — predictions logged but not shown. Compare AI labels vs. nurse labels post-hoc. IRB approval required.
3. **Decision support mode** (Month 5–6): Show AI suggestion and confidence to nurse after they complete initial assessment. Study influence on nurse decision.
4. **Live deployment** (Month 7+): AI suggestion shown upfront. Nurse can accept, modify, or override. All overrides logged.

### Failure Cases

| Failure | Probability | Impact | Mitigation |
|---|---|---|---|
| Level 1 patient classified as Level 3 | Low (target <0.5%) | Critical | Low threshold; mandatory human review for all Level 1 AI predictions |
| Model confident but wrong (overconfidence) | Medium | High | Calibrate probability outputs; show uncertainty intervals |
| Rare chief complaint (out-of-distribution) | Medium | Medium | OOD detector; fallback to nurse-only if confidence < 60% |
| Vital sign sensor error | High | Medium | Input validation; outlier flagging |

---

## Task 7 — Responsible AI Considerations

### Bias and Fairness

**Risk:** Historical triage data reflects existing biases — studies show women presenting with cardiac symptoms are 7× less likely to receive immediate triage than men. If AI learns from biased labels, it amplifies discrimination.

**Mitigation:**
- Audit model performance stratified by: age, sex, race/ethnicity, arrival mode, shift time
- If any group has recall gap > 3% for Level 1/2 vs overall, retrain with group-weighted loss
- Quarterly fairness audit report reviewed by clinical ethics committee

### Incorrect Predictions

**Risk:** A mis-triage caused by AI could result in patient harm or death — and create legal liability.

**Mitigation:**
- AI is advisory only — the triage nurse retains full clinical authority and legal responsibility
- All AI outputs display confidence score and top contributing factors (SHAP)
- Any prediction with confidence < 65% displays "Insufficient confidence — assess manually"
- Level 1 and Level 2 AI suggestions always trigger a mandatory second nurse verification

### Privacy and Data Protection

**Risk:** EHR data contains highly sensitive PII and health information.

**Mitigation:**
- All training data de-identified per DISHA (Digital Information Security in Healthcare Act) and ISO 27799
- Model does not store patient data — inference uses only the current session's inputs
- No patient identifiers in model weights, logs, or monitoring dashboards
- Data access logged and audited; role-based access control (RBAC) on all pipelines

### Over-Reliance on AI

**Risk:** Nurses may over-trust the AI, reducing their clinical vigilance ("automation complacency").

**Mitigation:**
- Framing: display as "AI suggestion" not "AI diagnosis"
- Regular simulation exercises where nurses triage without AI assistance
- Periodic accuracy benchmarking: if nurse accuracy declines when AI is present, intervention required
- Staff training module: "When to override the AI" — case studies of correct overrides

### Impact on Healthcare Workers

**Risk:** Nurses may fear job displacement; resistance to adoption may undermine the solution.

**Mitigation:**
- Co-design workshops with nursing staff throughout development
- Emphasise AI reduces cognitive load, not headcount
- AI handles routine pattern recognition; nurses handle human communication, empathy, edge cases
- Implementation tracked: workload, satisfaction, overtime — report quarterly

### Human Oversight

- **Override button** always visible; every override triggers a learning feedback loop
- **Confidence threshold failsafe:** predictions below 60% confidence not shown to nurse
- **Weekly model review board:** clinical lead + data scientist + ethics officer reviews unusual predictions
- **Incident response:** if two consecutive adverse events potentially linked to AI triage — automatic suspension pending investigation

---

## Task 8 — Final Solution Summary

---

### One-Page Executive Summary

**Organisation:** General City Hospital — Emergency Department  
**Document:** AI Solution Brief  
**Date:** May 2026

---

#### The Problem
The Emergency Department handles 400+ daily visits. Manual triage assigns each patient an Emergency Severity Index (ESI) level 1–5, but current accuracy is limited by fatigue, subjectivity, and cognitive overload — leading to a **6.2% under-triage rate** (high-risk patients missed) and **11.4% over-triage rate**, with Level 2 patients waiting an average of 28 minutes to see a physician.

#### The AI Solution
A **hybrid classification system** combining NLP (chief complaint analysis via DistilBERT) with structured tabular features (vital signs, demographics, EHR history) to recommend an ESI triage level in real-time — displayed to the triage nurse as a decision-support suggestion, not an autonomous decision.

#### Required Data
- 50,000+ historical triage records (vital signs + chief complaint + ESI label + disposition)
- EHR integration via HL7 FHIR API
- De-identified under DISHA; stored in hospital-controlled secure enclave

#### Model Architecture
- **Tier 1:** XGBoost on structured features (baseline, Month 1–2)
- **Tier 2:** DistilBERT (text) + Dense network (vitals) → late-fusion ensemble (production, Month 3–5)
- **Tier 3:** LSTM deterioration monitor for waiting room patients (Month 8+)

#### Expected Business Impact

| KPI | Current | 12-Month Target |
|---|---|---|
| Under-triage rate | 6.2% | **≤ 1.5%** |
| Over-triage rate | 11.4% | **≤ 5.0%** |
| Time to physician (Level 2) | 28 min | **≤ 18 min** |
| 30-day readmission | 8.1% | **≤ 5.5%** |
| Triage time | 4.2 min | **≤ 2.5 min** |
| Estimated adverse events avoided | — | **~120/year** |

**ROI:** At ₹15L average cost per adverse event avoidable, 120 events/year = **₹18 Crore annual risk reduction**.

#### Technical Quality Targets
- Recall (Level 1): **≥ 99%** · Recall (Level 2): **≥ 95%**
- Adjacent accuracy (±1 ESI level): **≥ 94%**
- Inference latency: **< 200 ms**

#### Risks and Mitigation

| Risk | Mitigation |
|---|---|
| Bias against demographic groups | Quarterly stratified fairness audit |
| AI over-confidence → missed Level 1 | Low classification threshold (0.30) for Levels 1–2 |
| Automation complacency | Nurse retains legal authority; regular un-aided drills |
| Privacy breach | De-identification pipeline; RBAC; no PII in model |
| Model drift | Monthly accuracy monitoring; automatic alert if recall drops below threshold |

#### Implementation Timeline

| Phase | Months | Milestone |
|---|---|---|
| Data preparation & baseline | 1–2 | XGBoost live in shadow mode |
| DistilBERT fine-tuning | 3–4 | Hybrid model shadow testing |
| Decision-support rollout | 5–6 | AI suggestion visible post-nurse assessment |
| Full advisory deployment | 7–8 | AI suggestion shown first; nurse confirms |
| LSTM deterioration monitor | 9–12 | Waiting room safety net active |

---

*This AI system is designed as a clinical decision-support tool. It does not diagnose, prescribe, or make autonomous medical decisions. The triage nurse retains full clinical and legal responsibility for every patient triage decision.*

---

## References

1. Vaswani et al. (2017). *Attention Is All You Need.* NeurIPS.
2. Devlin et al. (2019). *BERT: Pre-training of Deep Bidirectional Transformers.* NAACL.
3. Sanh et al. (2020). *DistilBERT.* EMC².
4. Fernandes et al. (2020). *Clinical Decision Support Systems for Triage in the ED: A Systematic Review.* Emergency Medicine Journal.
5. India Digital Information Security in Healthcare Act (DISHA), Draft 2022.
6. ESI Implementation Handbook (AHRQ, 4th Ed.).
