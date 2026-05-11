# PREMA: Prediction of Metabolic Acidosis in Preterm Infants

PREMA is a machine learning model that predicts **metabolic acidosis** in preterm infants **6 hours before onset**, using only routinely monitored heart rate (HR) and oxygen saturation (SpO₂) data sampled at 1-minute intervals.

---

## Overview

Preterm infants are vulnerable to life-threatening diseases, yet validated early warning tools remain limited. Metabolic acidosis — a quantifiable marker of tissue hypoperfusion and organ dysfunction — requires invasive blood sampling that exposes infants to iatrogenic risks including pain, blood loss, and skin injury.

PREMA addresses this gap by providing noninvasive, proactive prediction of metabolic acidosis using standard bedside monitoring data, enabling:

- Early detection of clinical deterioration **6 hours before onset**
- Reduction of iatrogenic harm by safe deferral of blood sampling

---

## Target Population

PREMA was developed and validated for preterm infants meeting either of the following criteria:

- Gestational age < 32 weeks, **or**
- Birth weight < 1,500 g

---

## Outcome Definition

Metabolic acidosis is defined by **both** of the following criteria on blood gas analysis:

- Blood pH < 7.2, **and**
- Base Excess (BE) < −7 mmol/L

---

## Model

PREMA is an XGBoost classifier trained on statistical features extracted from 4-hour segments of continuous vital sign data via a sliding-window approach.

- Each prediction segment spans **4 hours** of vital sign data
- Within each segment, **5 overlapping 2-hour observation windows** (shifted by 30 min each) are extracted
- Statistical and entropy-based features are computed from each window
- The segment slides forward in **30-minute steps**, generating a new prediction at each step

**Inputs**
- Time-series: 1-minute interval HR and SpO₂ (> 4 hours continuous)
- Static: gestational age (GA), birth weight (BW), sex, postmenstrual age (PMA)

**Output**
- Predicted probability of metabolic acidosis for each sliding window

---

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/PREMA.git
cd PREMA
```

### 2. Install dependencies

```bash
pip install pandas numpy scipy joblib antropy tqdm xgboost
```

> Python 3.8 or higher is recommended.

### 3. Prepare your data

Prepare a CSV file with the following columns:

**Vital signs** (1-minute interval, minimum 4 hours of continuous recording)

| Column | Type | Description |
|---|---|---|
| `patient_id` | int / str | Patient identifier |
| `Timestamp` | datetime | Measurement timestamp (`YYYY-MM-DD HH:MM`) |
| `HR` | float | Heart rate — allowed range: 0–250 bpm |
| `sat` | float | Oxygen saturation / SpO₂ — allowed range: 0–100% |

**Demographics** (one row per patient, repeated across timestamps)

| Column | Type | Description |
|---|---|---|
| `birthdate` | datetime | Date of birth (`YYYY-MM-DD`) |
| `GA` | int | Gestational age at birth (days) |
| `BW` | float | Birth weight (kg) |
| `male` | int | Sex (1 = male, 0 = female) |

> **Outlier handling:** Values outside the allowed ranges should be set to missing before input.  
> **Missing values:** Not imputed. The proportion of missing values per window is included as a feature.  
> **Minimum duration:** More than 4 hours of continuous data are required to generate at least one prediction.

A sample dataset is provided: [`vs_sample.csv`](vs_sample.csv)

### 4. Run the notebook

Open [`PREMA_code.ipynb`](PREMA_code.ipynb) and run all cells.

```
PREMA_code.ipynb
├── 1. Import libraries
├── 2. Load model         ← PREMA.pkl
├── 3. Load data          ← vs_sample.csv  (or your own CSV)
├── 4. Feature extraction
├── 5. Run prediction
└── 6. Save results       → prema_predictions.csv
```

Results are saved to `prema_predictions.csv`:

| Column | Description |
|---|---|
| `patient_id` | Patient identifier |
| `group_id` | Window index (0-based) |
| `window_start` | Start time of the prediction window |
| `window_end` | End time of the prediction window |
| `predicted_prob` | Predicted probability of metabolic acidosis |

---

## Repository Structure

```
PREMA/
├── LICENSE
├── README.md
├── PREMA.pkl            # Trained PREMA model
├── PREMA_code.ipynb     # Prediction notebook
├── vs_sample.csv        # Sample vital sign data (input)
└── gas_sample.csv       # Sample blood gas data (reference labels)
```

---

## Requirements

| Package | Version |
|---|---|
| Python | ≥ 3.8 |
| pandas | ≥ 1.3 |
| numpy | ≥ 1.21 |
| scipy | ≥ 1.7 |
| joblib | ≥ 1.1 |
| antropy | ≥ 0.1.4 |
| tqdm | ≥ 4.0 |
```

---

## Contact
 
For questions or collaboration inquiries:
 
- **Seng Chan You, MD, PhD** — Department of Biomedical Informatics, Yonsei University College of Medicine ([chandryou@yuhs.ac](mailto:chandryou@yuhs.ac))
- **Jeong Eun Shin, MD, PhD** — Department of Pediatrics, Yonsei University College of Medicine ([golden-week@yuhs.ac](mailto:golden-week@yuhs.ac))
---
 
## License
 
*License information will be updated.*

