# Chiller LOF + Local Feature Perturbation (Case Study 11)

**Name:** Lakshya Singh | **Reg. No.:** 24BDS0054
**Track:** Explainability, benchmarking and robustness | **Method:** Local Outlier Factor (LOF) + local feature perturbation

## Contents
| Path | Description |
|---|---|
| `chiller_lof_explainability.ipynb` | Main notebook for the case study. This notebook can be executed using seed 42 and cleans the raw log file, extracts features, runs Local Outlier Factor (LOF) on the extracted features, creates an explainer for the LOF model, evaluates the LOF model on various metrics, and creates figures to present the results. |
| `Analyse_Chillers_AI_Course.xls` | Raw instructor log (522 hourly records, 5 chillers); required input, not modified |
| `requirements.txt` | Python dependencies |
| `outputs/chiller_report.pdf` | Final report (same as the submitted PDF) |
| `outputs/preprocessing_notes.md` | Data-quality findings and cleaning decisions |
| `outputs/model_config.json`, `outputs/metrics.json` | Model settings (k=25, threshold 1.389) and all reported metrics |
| `outputs/tables/*.csv` | Result tables (scores, k selection, stability, injection test, case review) |
| `outputs/figures/*.png` | Figures 1-8 of the report |

## Environment
This code was tested using Python 3.13, with packages scikit-learn 1.9 and pandas 3.0. This code is expected to work on Python 3.11+.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## How to run
1. Unzip the file in the same folder where the notebook chiller_lof_explainability.ipynb and the instructor log file Analyse_Chillers_AI_Course.xls are located.
2. Start Jupyter **from that folder** (the notebook reads the data with a relative path):
   ```bash
   jupyter notebook chiller_lof_explainability.ipynb
   ```
   then choose *Kernel > Restart & Run All*.
   Or run headless:
   ```bash
   jupyter nbconvert --to notebook --execute --inplace chiller_lof_explainability.ipynb
   ```
3. The notebook will then write the required figures to outputs/figures/ and tables to outputs/tables/. Note that the random seed used in this notebook is fixed to 42 so that the results can be easily reproduced. However, there may be slight differences when running on other machines due to the use of other versions of libraries.

## Pipeline summary
1. Log file read. (Two title rows skipped). Timestamp created. Data quality audited.
2. Rule-based cleaning for implausible values of some variables, compressor-off rows not scored, redundant variables dropped.
3. Chronological split of the data into train, validation and test, with all statistics based on the training data only.
4. Train-only imputation, |Spearman| > 0.90 pruning (23 features kept), RobustScaler.
5. LOF (novelty mode, k=25, Euclidean); alert threshold = 97.5th percentile of training scores.
6. Local feature perturbation: For each feature or subsystem group, we measure the score decrease when that feature is replaced by values from the 25 nearest neighbors in the training set, averaged over 50 draws.
7. Evaluation: explanation stability, sufficiency, plausibility, injection test, and local case review.

## Notes
- There are no fault labels in the data set, hence no precision/recall values are reported. The flagged records are potential anomalies, not verified faults.
- Efficiency is assessed with thermodynamic proxies (lift, ideal Carnot COP); the log has no power or flow readings.