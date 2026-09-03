# Readmission Risk in Diabetic Inpatients

Identifying diabetic inpatients at risk of 30-day readmission at the point of
discharge, so that limited discharge-planning resources reach the patients most
likely to bounce back.

> **Status:** in development. This is a research and portfolio project. It is
> not a clinically validated tool and must not be used to guide patient care.

---

## The clinical problem

A diabetic patient is admitted, stabilised, and discharged. Weeks later they
return — DKA, a foot infection, uncontrolled hyperglycaemia, a hypoglycaemic
episode from a regimen nobody explained.

Many of these readmissions are predictable at the point of discharge. The
barrier is not clinical knowledge; it is that discharge decisions are made
under time pressure, by whoever is on duty, with beds needed. There is no
systematic way to flag the patients who need more attention while there is
still time to give it.

## What this model does — and does not do

**Does:** ranks discharged diabetic patients by estimated risk of readmission
within 30 days, so a discharge team can prioritise a limited number of
interventions — medication reconciliation, a diabetes educator session, a
follow-up appointment booked before discharge, a 72-hour phone check.

**Does not:** diagnose, recommend treatment, or make any decision autonomously.
The model allocates clinical attention. Clinicians make clinical decisions.

## Dataset

UCI Diabetes 130-US Hospitals (1999–2008) — 101,766 inpatient encounters for
patients with diabetes across 130 US hospitals.

See [`data/README.md`](data/README.md) for download instructions. Raw data is
not committed to this repository.

## Cohort definition

<!-- Replace with your own audit trail from notebook 01 -->
_To be completed from `01_cohort_definition.ipynb`._

| Step | n | Removed |
|---|---|---|
| Raw dataset | 101,766 | — |
| ... | | |

**Exclusions and rationale:** _(your writing)_

## Label definition

_(your writing — what the positive class means clinically, and what happened
to the `>30` group)_

## Methodology

_To be completed._

## Results

_To be completed._

## Limitations

This dataset describes US hospital encounters from 1999–2008. Case mix,
discharge practices, coding standards, and follow-up infrastructure differ
substantially from a Kenyan Level 4/5 facility. A model trained here should
not be assumed to transfer.

_(expand — which features would not exist in a Kenyan record? what is the
local equivalent of a readmission when a patient presents to a different
facility entirely?)_

## Repository structure

```
readmission-risk/
├── data/               # gitignored; see data/README.md
├── notebooks/
│   └── 01_cohort_definition.ipynb
├── src/
├── requirements.txt
└── README.md
```

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

## Author

Dr. Franklin Karimi, MBChB — clinician working at the intersection of clinical
medicine and machine learning.
