# Person A — Complete Data & Machine Learning Work Plan

**Owner:** Person A (Data/ML)  
**Works with:** Person B (application/integration)  
**Primary environment:** Google Colab  
**Goal:** Deliver an auditable data pipeline, an image-quality-feature baseline, an optional pretrained classifier, and honest held-out evaluation results.

> Do not diagnose disease. Do not describe SCIN's dermatologist gradability labels as direct ground truth for blur, exposure, or image quality.

## Definition of done

Person A's handoff is complete when Person B can:
1. Call the agreed feature-extraction function and receive a stable dictionary of numeric quality indicators.
2. Load model metadata/checkpoint and call the agreed prediction function, if a classifier succeeds.
3. Read an evaluation summary that specifies the label unit, label policy, case split, metrics, limitations, and experiment configuration.
4. Reproduce the reported result from the documented notebook/script using permitted data access.

## A0. First 15 minutes — coordinate before coding

- Read the repo README, SCIN schema, and SCIN Data Use License.
- Contact Person B and agree function signatures listed in `docs/TEAM_WORKFLOW.md`.
- Confirm whether Person B wants to integrate using a local Python module or a saved model and JSON config.
- Create branch `person-a-data-ml`; do not overwrite Person B's files.
- Keep data and model checkpoints outside the public repository. Commit code, documentation, small metrics tables, and plots only when permitted by the dataset terms.

## A1. Data access, schema, and license audit

**Deliverable:** `notebooks/01_data_audit.ipynb` plus an audit summary.

1. Use the official sources:
   - Dataset repository: https://github.com/google-research-datasets/scin
   - Schema: https://github.com/google-research-datasets/scin/blob/main/dataset_schema.md
   - License: https://github.com/google-research-datasets/scin/blob/main/LICENSE
2. Read the current dataset instructions rather than assuming yesterday's schema or file paths are still current.
3. Load the cases table and labels table; inspect column names and data types.
4. Inspect `case_id`, available image path fields (`image_1_path` through `image_3_path`, where present), shot-type fields, and all `dermatologist_gradable_for_skin_condition_*` fields.
5. Verify which table contains each field and how the tables join. Check join cardinality and unmatched case IDs.
6. Display a few example records and load a handful of images. Do not publish the images in GitHub.
7. Report:
   - number of cases;
   - number of image paths and successfully loaded images;
   - missing and unreadable images;
   - missing gradability labels;
   - counts by available label and counts of annotator disagreement;
   - number of cases with one, two, and three images;
   - possible duplicate images and duplicate case IDs;
   - any unexpected schema or parsing issues.
8. Record dataset access/version details, random seeds, software versions, and the date of the audit.

**Checkpoint A1:** Person A can explain in plain language what each target label means, where it comes from, and what it does *not* mean.

## A2. Agree on the label unit and build leakage-safe splits

**Deliverable:** documented label policy and saved split manifest.

SCIN gradability labels refer to a case's set of images, not unambiguously to an individual image. A case can include up to three images. The repository also documents duplicate image occurrences. This must affect the experiment design.

1. Preserve `case_id` throughout data processing.
2. Summarize each annotator's available gradability response. Do not silently turn missing labels into negatives.
3. Before training, discuss the analysis plan with Person B. Recommended first policy:
   - Use majority vote among the available dermatologist responses when a true majority exists.
   - If two available responses disagree, mark the case as disagreement/uncertain, not as a fabricated majority.
   - Keep missing-label cases out of supervised training/evaluation unless a separate justified rule is approved.
   - Report counts before and after exclusions; perform a disagreement summary.
4. Consider the scientifically cleaner option of **case-level modeling/aggregation** when multiple images are available. If time forces a single-image model trained with case-level labels, explicitly call it weak/noisy image-level supervision in every report and do not present its output as objective photo-quality ground truth.
5. Split by `case_id`, never by image row, into train/validation/test, e.g. 70/15/15 or another documented ratio appropriate to the data size and class balance.
6. Fix and record a random seed. Save case IDs and split membership so the experiment can be reproduced.
7. Check for exact duplicate images across splits where feasible. Keep related images and detected duplicate content from contaminating evaluation; document any duplicate-removal/exclusion policy.
8. Do not use the final test set to pick thresholds, tune the model, or choose which experiment to report.

**Checkpoint A2:** A saved manifest proves that a case, and related images associated with it, cannot appear in both training and final testing.

## A3. Implement the interpretable image-quality baseline

**Deliverable:** `src/quality_features.py` and tests or a notebook demonstrating its behavior.

Implement a function with a stable contract:

```python
def extract_quality_features(image) -> dict[str, float | int | bool]:
    ...
```

Initially calculate:
- width and height in pixels;
- grayscale variance of the Laplacian as an imperfect blur/focus proxy;
- mean brightness;
- proportions of very dark and very bright pixels;
- a basic contrast statistic (for example, standard deviation of grayscale intensities).

Requirements:
- Accept a documented input form, ideally a PIL image or NumPy array; convert inputs consistently.
- Handle grayscale/RGB/RGBA images, empty arrays, and invalid inputs clearly.
- Avoid hard-coded claims such as "this image is clinically usable." A feature value is a measurement/proxy, not a diagnosis.
- Document units, definitions, any thresholds, and known limitations.
- Add tests for ordinary RGB, grayscale, invalid/empty inputs, and synthetic blur/brightness changes.
- Do not implement an anatomical-framing score unless a reliable method is actually built and evaluated.

Thresholds are provisional. Select any thresholds used for status messages on validation data or label them explicitly as unvalidated prototype rules.

**Checkpoint A3:** Person B can call the function on an uploaded image and render the returned values without changing them.

## A4. Establish simple baselines

**Deliverable:** baseline report and saved predictions.

At minimum:
1. Run feature extraction over usable images/cases.
2. Report feature distributions and missing values.
3. If the target-label policy supports it, train a small scikit-learn model (e.g. logistic regression or random forest) on explicitly defined features.
4. Be clear about whether each observation represents an image or a case. Never mix units unnoticed.
5. Record hyperparameters, random seed, split manifest, label policy, and exact metrics.

Important: simple blur/brightness/resolution features do not directly have real SCIN quality labels. Their expected response to controlled corruptions can be tested separately; do not use SCIN gradability as if it were a perfect blur label.

**Checkpoint A4:** The simple system runs reproducibly and provides a fair baseline for deciding whether the neural model is worth the time.

## A5. Optional pretrained image classifier (only after A1–A4)

**Deliverable:** model checkpoint stored outside the repo, training script/config, validation summary.

1. If the label unit is case-level, first consider a case-level representation that uses all available images; if time prevents that and a single-image classifier is used, document the weak supervision explicitly.
2. Use ImageNet-pretrained MobileNetV3-Small via torchvision or `timm`, if available and compatible.
3. Use a train-only augmentation policy that does not create implausible images or change the target meaning.
4. Use the saved case-level split. Monitor validation performance. Save the best checkpoint and configuration outside GitHub.
5. Use class weighting or another documented method if classes are imbalanced.
6. Don't run many experiments against the test set. Open the test set only after choosing the model/threshold on validation.
7. If training fails due to time/compute, stop and keep the working simple baseline. An honest baseline is better than an untested/broken deep model.

**Checkpoint A5:** Person B receives a reproducible prediction wrapper and clear status about model readiness—not just an opaque checkpoint file.

## A6. Evaluation and error analysis

**Deliverable:** `src/evaluate.py`, metrics CSV, confusion matrix/plots, and a short written interpretation.

On held-out test cases report applicable metrics:
- confusion matrix;
- recall/sensitivity for ungradable-labelled cases;
- false acceptance rate: ungradable-labelled cases predicted/treated as suitable divided by all ungradable-labelled test cases;
- specificity and false rejection rate for gradable-labelled cases;
- F1 and balanced accuracy;
- ROC-AUC or PR-AUC only when appropriate for the prediction format and class distribution;
- sample counts and confidence intervals where feasible.

Define which class is positive and each denominator in the report. Don't report accuracy alone. Clearly distinguish "gradable label prediction" from "measured image-quality indicator."

Threshold selection:
- use only the validation set to choose thresholds;
- report threshold-specific trade-offs;
- include an uncertain state if the model/indicator combination should not force every case into yes/no.

Error analysis:
- inspect false acceptances and false rejections via metadata/metrics;
- report label disagreements and possible leakage/duplicate issues;
- avoid publishing restricted example images.

## A7. Controlled robustness tests

**Deliverable:** reproducible table/plots showing blur, brightness, downsampling, and/or JPEG compression severity against quality-feature/model-score changes.

1. Use held-out images only after the model is fixed.
2. Preserve the original, and create controlled transformed copies in memory or in a private workspace.
3. Record transformation type/severity and seed where applicable.
4. Evaluate how quality indicators change and whether model scores move consistently.
5. Explain that the original gradability label is only a stress-test assumption after transformation; severe corruption can change interpretability.
6. Do not claim these simulated changes reproduce real patient behavior or establish clinical benefit.

## A8. Person A handoff package

Send Person B:
- [ ] Notebook: data audit and data access steps
- [ ] Documented label policy and case-level split manifest
- [ ] `extract_quality_features(image)` with a stable output schema
- [ ] Trained-model wrapper/checkpoint/config if it works
- [ ] Prediction output definition, e.g. label name, score meaning, model version
- [ ] Metrics CSV and confusion-matrix plot
- [ ] At least two controlled-corruption results
- [ ] Summary of limitations and known issues
- [ ] Instructions to reproduce the notebook/scripts
- [ ] A brief written handoff: what works, what doesn't, what Person B should call, and what claims the app must avoid

## Learning goals for Person A

Be able to explain: dataset/schema; labels and missingness; case-level leakage; majority vote and disagreement; features vs labels; train/validation/test; baseline vs pretrained classifier; precision/recall/specificity; false acceptance/rejection; threshold selection; and why controlled corruptions are not clinical validation.

## If blocked

- If official data access/schema is unresolved after the first 1–2 hours, tell Person B immediately.
- If neural training isn't working by the agreed cutoff, stop. Deliver the audit, feature extraction, simple baseline, and clear report.
- Never invent metrics, create labels from guesses, or put restricted images/credentials in GitHub.
