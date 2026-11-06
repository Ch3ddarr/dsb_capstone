# Machine Learning Network Intrusion Detection System (ML-NIDS)

**Data Science / Machine Learning Capstone Project — General Assembly**

## Overview

Working in cybersecurity, I have seen how limited staffing can make continuous network monitoring and investigation difficult. I wanted to explore whether machine learning could help analysts distinguish potentially malicious network traffic from normal activity.

This project is a **proof of concept**, not a production-ready intrusion detection system. It uses the UNSW-NB15 dataset to train and compare three supervised classification algorithms, evaluate their errors, and examine whether the results could be useful in a Security Operations Center (SOC).

**Research question:** *Can machine learning classify network flows as normal or malicious, and what trade-offs would that create for security analysts?*

## Dataset

The project uses the **[UNSW-NB15 dataset](https://research.unsw.edu.au/projects/unsw-nb15-dataset)**, developed at UNSW Canberra. It contains laboratory-generated network traffic with normal activity and simulated attacks. The benchmark dates from 2015.

| Dataset | Records | Purpose |
|---|---:|---|
| `UNSW_NB15_training-set.csv` | 175,341 | Model development and training |
| `UNSW_NB15_testing-set.csv` | 82,332 | Final evaluation |

Each row describes a **network flow**, represented by features such as protocol, connection duration, packet counts, byte counts, and TTL values.

- **Target:** `label` (`0` = normal, `1` = attack).
- **Input features:** 42 columns after excluding `id`, `label`, and `attack_cat`.
- **Attack category:** `attack_cat` is retained for analyzing detection performance by attack type, **not** used as a model input.

The task is **binary classification** (normal vs. attack), not prediction of the specific attack category.

## Approach

1. **Exploratory data analysis:** inspected class distributions, attack categories, protocols, connection durations, and other traffic features.
2. **Preprocessing:** used `ColumnTransformer` with `StandardScaler` for numeric features and `OneHotEncoder(handle_unknown='ignore')` for categorical features (`proto`, `service`, and `state`). Preprocessing is included inside scikit-learn pipelines.
3. **Validation:** split the designated training file into 80% training and 20% validation data, preserving class proportions with `stratify=y` and using `random_state=42`.
4. **Baseline:** compared results against a simple strategy of labeling every flow as an attack (approximately **68.1% accuracy** on the original training distribution).
5. **Model comparison:** trained **Logistic Regression**, **Decision Tree**, and **Random Forest**, and explored different maximum depths for Decision Tree.
6. **Final evaluation:** selected Random Forest based on validation performance, refitted it on all 175,341 designated training records, and evaluated it on the separate 82,332-record official test set.

## Results

### Validation set

All three algorithms were evaluated on the same 35,069 validation records.

| Model | Validation accuracy |
|---|---:|
| Logistic Regression | 93.64% |
| Decision Tree (unrestricted depth) | 94.94% |
| **Random Forest (100 trees)** | **95.95%** |

Random Forest provided the best overall validation accuracy and was selected for the final test. Logistic Regression had a higher attack recall on validation, showing that the choice involves a trade-off between missed attacks and false alarms.

### Final Random Forest test results

The selected pipeline was refitted on the full training set and evaluated on the official, previously held-out test set.

| Metric | Final test result |
|---|---:|
| Accuracy | **87.11%** |
| Attack precision | **81.78%** |
| Attack recall | **98.54%** |
| Attack F1-score | **89.38%** |
| False positive rate (normal traffic) | **26.91%** |

**Confusion matrix (82,332 test flows):**

| Actual / Predicted | Normal | Attack |
|---|---:|---:|
| **Normal** | 27,045 | 9,955 |
| **Attack** | 660 | 44,672 |

The model detected most of the recorded attacks, but it also flagged **9,955 normal flows** as malicious. That false-alarm volume is a significant limitation for an operational SOC. Final accuracy was also substantially lower than validation accuracy, highlighting the need to test generalization beyond one development split.

### Analysis of missed attacks

Of the **660 attacks missed** in the final test, **606 were categorized as Fuzzers** (approximately **91.8% of all missed attacks**). The model's recall for Fuzzers was approximately **90.0%**, compared with **98.54% recall** across all attack flows.

The notebook also examines Random Forest feature importance. Among the highest-ranked features were `sttl`, `ct_state_ttl`, `dload`, `rate`, and `dttl`. These scores describe the trained model's use of features; they do **not** establish causal relationships.

## Limitations

- UNSW-NB15 is a **2015 laboratory benchmark**. Results should not be assumed to generalize to today's enterprise networks or newly emerging attacks.
- The dataset's attack prevalence and traffic characteristics differ from what an operational SOC may encounter.
- **False positives remain high:** 26.91% of normal flows in the official test set were flagged as attacks.
- The internal random train/validation split does not rule out similarity between related network flows, so validation results may be optimistic.
- This notebook classifies **pre-extracted flow features**. It does not capture live traffic, process raw PCAPs, integrate with a SIEM, or automatically respond to incidents.

## Reproducing the notebook

**Tools:** Python, Jupyter Notebook / Google Colab, pandas, NumPy, Matplotlib, seaborn, and scikit-learn.

1. Download the **training** and **testing** CSV files from the [official UNSW-NB15 dataset page](https://research.unsw.edu.au/projects/unsw-nb15-dataset).
2. Open [`01_EDA_(1)(1).ipynb`](./01_EDA_%281%29%281%29.ipynb) in Google Colab or Jupyter.
3. The notebook currently reads CSV files from `/content/`, so upload both CSVs there when using Colab, or update the two `pd.read_csv(...)` paths for your environment.
4. Run all notebook cells in order. The saved notebook includes the EDA plots, model reports, confusion matrices, attack-category analysis, and feature-importance chart.

If packages are missing from your Python environment, install them with:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn notebook
```

The notebook is a learning and evaluation artifact; **no web app or deployment service is required** to reproduce its results.

## Conclusion

Machine learning can distinguish normal and malicious flows in this benchmark with high attack recall. However, **high detection rates alone do not make a usable SOC alerting system**. The much lower performance on the held-out test set and the number of false alarms show why independent evaluation and error analysis matter as much as model accuracy.

For me, the main outcome of this capstone is a reproducible workflow for investigating network intrusion detection with machine learning and a clearer understanding of what would need to improve before operational use.

## Dataset reference

Moustafa, N., & Slay, J. (2015). *UNSW-NB15: A comprehensive data set for network intrusion detection systems (UNSW-NB15 network data set).* Military Communications and Information Systems Conference (MilCIS). https://doi.org/10.1109/MilCIS.2015.7348942

Dataset source and usage terms: [UNSW Research](https://research.unsw.edu.au/projects/unsw-nb15-dataset).
