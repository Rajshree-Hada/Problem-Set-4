# Problem-Set-4

# Store Revenue Model: Investigating a "0.94 R-squared, Validated" Handover

This project uses simulated store data to investigate what a handover note claiming an R-squared of 0.94 and validated model performance actually tells us. The goal is to evaluate how well the model predicts revenue, identify potential issues with its assumptions, and determine how reliable its reported performance really is.

## Data

The dataset contains 400 stores and 11 columns, including floor area, foot traffic, median income, years open, five irrelevant predictors, `catchment_index`, and revenue. The catchment index is highly correlated with foot traffic and median income, as it is constructed from these variables with added noise.

Revenue is generated using floor area, foot traffic, median income, years open, and random noise. The relationship between floor area and revenue is concave, while the noise increases with store size. Because the data-generating process is known, the irreducible RMSE is approximately 186.

## Key Results

| Phase | Finding | Result |
|---|---|---|
| 1.2 | The full-sample R-squared reproduces the original handover figure. | 0.933 |
| 1.2 | The model performs similarly on training and held-out data, with only a small difference between the two scores. | 0.936 / 0.926 (gap: 0.010) |
| 1.3 | Adding three irrelevant predictors slightly increases R-squared, but adjusted R-squared barely changes. | 0.9327 → 0.9334 / 0.9320 → 0.9322 |
| 2.1 | Prediction errors are substantially more variable for larger stores than smaller stores. | Residual SD is 2.65× higher for the largest store quarter. |
| 2.1 | HC3 robust standard errors slightly increase uncertainty around the floor-area coefficient without changing the estimated slope. | Classical: 0.0068; HC3: 0.0073; slope: 0.4527 |
| 2.2 | The model systematically overpredicts revenue for both the smallest and largest store groups. | Mean residual: −80.8 (−2.5%) / −71.2 (−1.4%) |
| 2.2 | The catchment index and its underlying variables exhibit substantial multicollinearity. | VIF: 211.5 (`catchment_index`), 177.9 (`foot_traffic`), 36.6 (`median_income`) |
| 3.1 | A degree-2 model using four real predictors achieves the lowest held-out MSE among the models tested. | 14 terms; MSE ≈ 37,930 |
| 3.1 | The 285-term model fits the training data almost perfectly but performs extremely poorly on held-out data, demonstrating overfitting. | Train MSE ≈ 0; held-out MSE ≈ 1.35 × 10⁷ |
| 3.2 | Five-fold cross-validation estimates an MSE of 37,103, with considerable variation across folds. | Approximate interval: 28,442–45,765 |
| 3.2 | Nested cross-validation and evaluation on an untouched sample produce similar estimates of predictive performance. | 37,103 and 37,928 |
| 4.1 | The random forest learning curve shows a large gap between training and validation RMSE, suggesting overfitting at 320 training observations. | Train ≈ 99; validation ≈ 226 |
| 4.2 | Selecting 300 noise predictors before cross-validation produces an artificially positive score, while selecting them within each fold reveals poor generalization. | 0.066 vs. −0.280 (gap: 0.346) |

## Conclusions

The original R-squared of 0.94 reflects in-sample performance rather than the model's ability to predict revenue for unseen stores. Its held-out R-squared of approximately 0.93 is still strong, and its RMSE of roughly 193 is relatively close to the known noise floor of 186. However, the overall performance masks systematic errors across store sizes, with the model failing to capture the concave relationship between floor area and revenue and producing substantially more variable errors among larger stores.

The analysis also shows that multicollinearity primarily affects coefficient reliability and standard errors rather than necessarily reducing predictive accuracy. Although `catchment_index` has a very high VIF, it does not add meaningful information and can be removed to improve interpretability. More importantly, the original validation approach involved data leakage: selecting predictors before cross-validation allowed irrelevant variables to appear predictive. When feature selection is performed within each fold, the model's performance on noise predictors becomes negative, revealing that the original validation results were misleading.

A degree-2 model marginally outperforms the linear specification on held-out data (MSE of 37,930 versus 37,960). This supports testing a nonlinear floor-area term, although the improvement is small and does not establish that the more complex model is necessary.
