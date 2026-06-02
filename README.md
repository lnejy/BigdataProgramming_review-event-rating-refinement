# BigdataProgramming_review-event-rating-refinement
2026 HSU BigdataProgramming

## Recent Updates

- Added probability calibration with `IsotonicRegression` to the proposed hybrid MLP pipeline.
- Stored calibrated probability metadata in model bundles so the same correction is reused in model selection, rating refinement, and demo inference.
- Re-executed notebooks `04SMOTE_5-fold_model_train.ipynb` to `08end_to_end_demo.ipynb` after calibration changes.
- Updated downstream comparison so calibrated probabilities are used consistently before thresholding and candidate-2 rating refinement.
