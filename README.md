# PREMA
**PREMA** is a machine learning model that predicts **metabolic acidosis** in **preterm infants 6 hours before onset**, using only routinely monitored **heart rate (HR)** and **oxygen saturation (SpO₂)** data sampled at **1-minute intervals**.

## Overview
Preterm infants are vulnerable to life-threatening diseases, yet validated early warning tools remain limited.    
**Metabolic acidosis** — a quantifiable marker of tissue hypoperfusion and organ dysfunction — requires invasive blood sampling that exposes infants to iatrogenic risks including pain, blood loss, and skin injury.     
**PREMA** addresses this gap by providing noninvasive, proactive prediction of metabolic acidosis using standard bedside monitoring data, enabling:    
* Early detection of clinical deterioration 6 hours before onset
* Reduction of iatrogenic harm by safe deferral of blood sampling

This repository provides the official implementation of PREMA, including code, data samples, and the needed documentation. 

## Target Population
PREMA was developed and validated for preterm infants meeting either of the following criteria:
* Gestational age < 32 weeks, OR
* Birth weight < 1,500 g

## Outcome Definition
Metabolic acidosis is defined by both of the following criteria on blood gas analysis:
* Blood pH < 7.2
* Base Excess (BE) < -7mmol/L

## Input Data Requirement 
### Vital sign data
| Variable | Column | Type | Encoding | Sampling Rate | Minimum Duration | Allowed Range |
|---|---|---|---|---|---|---|
| Timestamp | `datetime` | numeric | `YYYY-MM-DD HH:MM` | 1-minute intervals | > 4 hours | — |
| Heart rate | `HR` | numeric | continuous (bpm) | 1-minute intervals | > 4 hours | 0–250 bpm |
| Oxygen saturation | `sat` | numeric | continuous (%) | 1-minute intervals | > 4 hours | 0–100 (%) |

- The Timestamp column is **required** for extracting temporal features.
- Outlier removal: Values outside the allowed ranges (HR: 0–250 bpm; SpO₂: 0–100%) should be treated as missing.
- Missing value handling: Missing values are **not imputed**. The proportion of missing values within each sliding window is included as a feature, preserving the integrity of the original physiologic recordings.
- Minimum duration: A continuous data stream of **more than 4 hours** is required to generate a single prediction.


### Demographic data
| Variable | Column | Type | Encoding | Description |
|---|---|---|---|---|
| Gestational age | `GA` | numeric | Continuous (days) | Gestational age at birth |
| Birth weight | `BW` | numeric | Continuous (kg) | Weight at birth |
| Sex | `male` | integer | `1` = Male, `0` = Female | Biological sex |
| Postmenstrual age | `PMA` | numeric | Continuous (days) | PMA at the time of prediction |

## Model 
PREMA is an **XGBoost** classifier trained on statistical features extracted from 4-hour segments of continuous vital sign data via a sliding-window approach.
- Each prediction segment spans **4 hours** of vital sign data
- Within each segment, **5 overlapping 2-hour observation windows** (shifted by 30 min each) are extracted
- Statistical and entropy-based features are computed from each observation window
- The window slides forward in **30-minute steps**, generating a new prediction at each step

### Quick Start
#### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/PREMA.git
cd PREMA
```

#### 2. Install dependencies

```bash
pip install pandas numpy scipy joblib antropy tqdm
```

> Python 3.8 or higher is recommended.

#### 3. Prepare your data

Your vital sign data should be a CSV file with the following columns:

| Column | Type | Description |
|---|---|---|
| `patient_id` | int / str | Patient identifier |
| `birthdate` | datetime | Date of birth (`YYYY-MM-DD`) |
| `GA` | int | Gestational age at birth (days) |
| `BW` | float | Birth weight (kg) |
| `male` | int | Sex (1 = male, 0 = female) |
| `Timestamp` | datetime | Measurement timestamp (`YYYY-MM-DD HH:MM`) |
| `HR` | float | Heart rate (bpm) |
| `sat` | float | Oxygen saturation / SpO₂ (%) |

A sample dataset is provided: [`vs_sample.csv`](vs_sample.csv)

#### 4. Run the notebook

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

Results are saved to `prema_predictions.csv` with the following columns:

| Column | Description |
|---|---|
| `patient_id` | Patient identifier |
| `group_id` | Window index (0-based) |
| `window_start` | Start time of the prediction window |
| `window_end` | End time of the prediction window |
| `predicted_prob` | Predicted probability of metabolic acidosis |


## Repository Structure
*Repository structure is subject to change. Please refer to the latest version.*

## Contact 
- **Ju Hyun Jin, MD** - jhjin122@yuhs.ac
- **Seng Chan You, MD, PhD** - chandryou@yuhs.ac
- **Jeong Eun Shin, MD, PhD** - golden-week@yuhs.ac
