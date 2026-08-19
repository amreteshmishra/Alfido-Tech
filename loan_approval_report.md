# Loan Approval Model — Summary Report

**Dataset:** 614 historical loan applications, 12 borrower/application features, target = `Loan_Status` (approved/denied). Base rate: 69% approved, 31% denied.

## What we built

A preprocessing pipeline (median/mode imputation, one-hot encoding, standard scaling) feeding six model configurations — Logistic Regression, Decision Tree, Random Forest, and XGBoost — each combined with either **class-weighting** or **SMOTE** to correct for the moderate class imbalance. Models were compared on a held-out 20% test set and validated with 5-fold cross-validation.

## Results

| Model | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|
| **Logistic Regression (SMOTE)** | 0.862 | 0.882 | 0.872 | **0.875** |
| Logistic Regression (class-weighted) | 0.872 | 0.882 | 0.877 | 0.856 |
| Random Forest (SMOTE) | 0.837 | 0.906 | 0.870 | 0.811 |
| Random Forest (class-weighted) | 0.839 | 0.918 | 0.876 | 0.809 |
| XGBoost (scale_pos_weight) | 0.843 | 0.824 | 0.833 | 0.780 |
| Decision Tree (class-weighted) | 0.822 | 0.871 | 0.846 | 0.735 |

*(Precision/Recall/F1 reported for the "approved" class at the default 0.5 threshold.)*

**Logistic Regression came out on top**, and by a meaningful margin on ROC-AUC. This is a signal that the relationship between the features — especially credit history — and the approval outcome is close to linear, so the added flexibility of tree ensembles doesn't pay off on a dataset this small (491 training rows). 5-fold cross-validation confirmed the result wasn't a fluke of one particular train/test split.

## What actually drives approvals

`Credit_History` is by far the strongest predictor in every model tested — applicants with a documented good credit history are approved at a much higher rate, independent of income or requested loan amount. `Property_Area` (semi-urban applicants tend to do somewhat better than rural ones) and marital status have smaller but consistent effects. Income and loan amount carry surprisingly little independent signal on their own — likely because affordability screening already happened before this data was recorded, filtering out applications where the loan amount didn't fit the applicant's income.

## Trade-offs and threshold recommendation

Model precision and recall trade off directly against the decision threshold used to convert predicted probability into an approve/deny call:

- **Raising the threshold** (e.g., to ~0.6–0.65) increases precision — fewer risky loans get funded — at the cost of recall: more legitimately good applicants get declined or pushed to manual review.
- **Lowering the threshold** (e.g., to ~0.4–0.45) does the opposite: more good applicants get through, but more bad loans slip in.

Which direction to move depends on which mistake costs the business more. For most lenders, a bad loan (false approval) is more expensive than a missed good customer (false denial), so we recommend:

- **Auto-approve above ≈ 0.6** predicted probability.
- **Auto-deny below ≈ 0.35.**
- **Route the ≈ 0.35–0.6 band to manual underwriting** — this is where the model is least confident and most error-prone, and where human judgment adds the most value per case reviewed.

This keeps the model's job to what it's actually good at (confidently sorting the clear cases) rather than forcing a single binary cutoff onto every application.

## Caveats before production use

- **Small sample size.** 614 rows total means test-set metrics carry real sampling noise; the 5-fold CV results should be treated as the more reliable estimate, and both should be revisited as more data accumulates.
- **Fairness review needed.** Since the model leans heavily on credit history, and credit history itself can encode historical lending bias, we'd recommend an explicit fairness audit (approval/error rates broken out by gender, marital status, and property area) before this model influences real decisions.
- **The model imitates past decisions, not necessarily optimal ones.** Training labels come from the lender's own historical approvals, so the model will reproduce whatever patterns — good or bad — were already present in that process.

*Full code, plots, and threshold sweep are in the accompanying notebook (`loan_approval_modeling.ipynb`).*
