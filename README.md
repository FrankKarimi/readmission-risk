# Readmission Risk in Diabetic Inpatients

Identifying diabetic inpatients at risk of 30-day readmission at the point of discharge, so that limited discharge-planning resources reach the patients most likely to bounce back.

**Status:** complete, six notebooks. This is a research and portfolio project. It is not a clinically validated tool and must not be used to guide patient care.

---

## The clinical problem

A diabetic patient is admitted, stabilised, and discharged. Weeks later they return — DKA, a foot infection, uncontrolled hyperglycaemia, a hypoglycaemic episode from a regimen nobody explained.

Many of these readmissions are predictable at the point of discharge. The barrier is not clinical knowledge; it is that discharge decisions are made under time pressure, by whoever is on duty, with beds needed. There is no systematic way to flag the patients who need more attention while there is still time to give it.

## What this model does and does not do

**Does:** ranks discharged diabetic patients by estimated risk of readmission within 30 days, so a discharge team can prioritise a limited number of interventions  including medication reconciliation, a diabetes educator session, a follow-up appointment booked before discharge, a 72-hour phone check.

**Does not:** diagnose, recommend treatment, or make any decision autonomously. The model allocates clinical attention. Clinicians make clinical decisions.

---

## What the project found

The model works, and the more useful result is what it could not do.

Set to flag the top 28% of discharges, it catches 49% of the patients who return. Ranking by a single number already written in the notes — how many times the patient has been admitted in the past year — catches 45%. The model buys 95 extra patients out of 19,634 discharges, a gain that holds up on a bootstrap but is far smaller than the effort implies.

That sent the project looking for why, and the answer is more actionable than the model:

- **Six features match thirty-seven.** Reducing the feature set from 37 to 6 changed the catch count from 1,019 to 1,009, a difference the bootstrap cannot distinguish from zero.
- **Two variables carry the model.** Prior inpatient admissions 50.1% of permutation importance, discharge disposition 27.3%. Nothing else reaches 5%. Six variables contribute exactly zero.
- **The model is weakest where risk is highest.** Discrimination falls from 0.737 in patients in their thirties to 0.601 in those over ninety, who have the second-highest base rate in the cohort. The same gradient appears under logistic regression, so it is a property of the data, not of model choice.
- **Aggregate calibration hides subgroup failure.** Overall calibration was near perfect (mean absolute decile gap 0.0071) while one racial group was over-predicted by 11% and one age band under-predicted by 22%. Both reproduced across two model families.

What decides whether a patient returns including whether they can afford next month's medication, reach a follow-up clinic, or have someone at home helping is not recorded anywhere. The strongest social signal in the dataset is discharge destination, which is not a measurement of the patient at all.

So the deliverable is a specification of what a hospital would need to start recording before a model is worth deploying. That is notebook 06, section 9.

---

## Headline numbers

| | |
|---|---|
| Cohort | 99,319 encounters, 69,980 patients |
| Outcome | Readmission within 30 days, base rate 11.4% |
| Final model | Gradient boosting, ROC AUC 0.670, Brier 0.0939 |
| Operating point | Flags 27.9% of discharges, catches 49.0%, precision 19.4% |
| Against the heuristic | +95 readmissions caught (bootstrap CI 57 to 137) |

---

## Dataset

UCI Diabetes 130-US Hospitals (1999–2008) — 101,766 inpatient encounters for patients with diabetes across 130 US hospitals.

See `data/README.md` for download instructions. Raw data is not committed to this repository.

## Cohort definition

| Step | n | Removed |
|---|---|---|
| Raw dataset | 101,766 | — |
| After discharge disposition exclusions | 99,319 | 2,447 |

Exclusions were justified individually rather than by a blanket rule. Deaths were excluded as label leakage: the outcome cannot occur. Hospice discharges were excluded as the wrong target population rather than as leakage, since flagging a comfort-care patient for discharge-planning intervention is clinically inappropriate. Two codes were excluded because the index encounter never ended, so the 30-day clock cannot start.

Transfer codes were retained. The concern was that a patient transferred outside the 130-hospital network could be readmitted invisibly, making them a false negative. Testing readmission rate by discharge destination showed rates rising with destination acuity, which is not what severe under-capture would look like. Noted as a one-way test: a low rate would have proven under-capture, a high rate does not prove full capture.

Repeat patients were kept rather than dropped. Patient-level correlation is handled at the train/test split with `GroupShuffleSplit` on patient number, which avoids discarding 30% of encounters concentrated in the high-event-rate group.

Full detail, with row counts after each step, is in notebook 01.

## Label definition

Binary: `readmitted == "<30"` is the positive class, everything else negative. Base rate 11.4%.

The `>30` group was deliberately kept in the negative class rather than dropped. Removing it would delete the patients whose existence proves the 30-day cap is arbitrary, inflate the base rate to a false 17.7%, and leave the model never having seen a slow returner.

The window itself is an administrative artefact of Medicare's readmission penalty, not a physiological one. The clinically correct formulation is time-to-event analysis, which this dataset cannot support because the outcome is supplied as three categories with no dates. The negative class is therefore heterogeneous by construction, mixing true non-returners with slow returners. The justification for keeping 30 days is that the discharge-planning intervention has a shorter horizon than the risk, so the prediction window matches the action window.

---

## Notebooks

| | Notebook | What it does |
|---|---|---|
| 01 | Cohort and label definition | Who belongs in the analysis and what the outcome measures. No modelling. |
| 02 | Exploratory Analysis| Missingness, variables that measure acuity rather than what their names suggest, diagnosis grouping. |
| 03 | Baseline model development | Logistic regression, patient-grouped split, calibration, coefficients. |
| 04 | Evaluation | Operating threshold, subgroup performance, fairness, first pass at risk tiers. |
| 05 | Model comparison | Gradient boosting and a reduced six-feature model, judged at a fixed review budget with bootstrap intervals. |
| 06 | Explainability, fairness and transferability | Subgroup calibration, permutation importance, risk tiers, a clinician-facing note, and what a Kenyan version would need instead. |

Read 01 and 06 if you are reading two.

---

## Limitations

This dataset describes US hospital encounters from 1999–2008. Case mix, discharge practices, coding standards, and follow-up infrastructure differ substantially from a Kenyan Level 4/5 facility. A model trained here should not be assumed to transfer.

Notebook 06 works through what transfer would require. In summary: prior admissions, half the model, cannot be counted across facilities without a shared patient identifier, and is understated most for patients who move between facilities seeking care they can afford. Discharge disposition, another quarter, does not survive at all — the skilled nursing, rehabilitation and home health destinations doing the work do not exist, and almost every patient goes home.

The outcome transfers worst of all. A readmission is only observed if it happens at the same facility. Locally, a patient who deteriorates may present elsewhere with no shared record, or may not present at all because the fare is more than they have. Both are recorded as patients who did well, which means a model trained on that label would partly learn to predict who can afford to come back.

Loss to follow-up at the one-month review is the outcome the same pipeline should be pointed at instead.

Other limitations:

- No vital signs, renal function or albumin in this extract. Their absence is part of why the ceiling sits where it does.
- Weight is missing in 96.9% of encounters and was dropped.
- Encounters are clustered within patients. The split is patient-grouped, but confidence intervals from encounter-level resampling are optimistic.
- Developed on a predominantly adult type 2 population. Under-20s are 0.9% of the cohort and predominantly type 1 in DKA; any paediatric estimate rests on 43 events.
- An unexplained, reproducible over-prediction in one racial group is documented in notebook 06 and not resolved.

---

## Repository structure

```
readmission-risk/
├── data/               # gitignored; see data/README.md
├── Notebooks/
│   ├── 01_cohort_definition.ipynb
│   ├── 02_exploration.ipynb
│   ├── 03_baseline_model.ipynb
│   ├── 04_model_evaluation.ipynb
│   ├── 05_model_comparison.ipynb
│   └── 06_explainability.ipynb
├── models/             # gitignored; rebuilt by the notebooks
├── requirements.txt
└── README.md
```

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Notebooks run in order. `test_predictions.csv` and a two-column `test_predictors.csv` are committed so notebooks 04 onward run from a clone without regenerating the cohort. Notebook 06 rebuilds the notebook 05 model and asserts it reproduces the same figures before analysing it.

---

## Author

Dr. Franklin Karimi, MBChB — clinician working at the intersection of clinical medicine and machine learning.

[LinkedIn](https://linkedin.com/in/franklin-karimi-189506354) · [GitHub](https://github.com/FrankKarimi)
