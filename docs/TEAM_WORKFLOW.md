# DermReady Team Workflow

## 1. One goal, two workstreams

- **Person A:** data audit, labels, case-level split, quality features, model baseline, evaluation.
- **Person B:** Gradio app, upload validation, feedback, integration, tests, demo.
- **Both:** agree the scientific task definition, examine real results, confirm licensing/disclaimer, and rehearse.

## 2. Integration contracts

Keep these simple. Person A owns the feature and optional model functions; Person B owns UI and feedback.

### Quality features
```python
extract_quality_features(image) -> dict
```
Minimum agreed keys should be stable and documented. Suggested keys: `width`, `height`, `laplacian_variance`, `mean_brightness`, `dark_pixel_fraction`, `bright_pixel_fraction`, `contrast_std`. Values and definitions must be confirmed by Person A. Do not rename keys silently.

### Model prediction (optional)
```python
predict_gradability(image) -> dict
```
Suggested fields: `status`, `score`, `predicted_label`, `label_unit`, `model_version`. Document exactly what the score means, which class is positive, the transform used, and whether it is calibrated. Person B must not infer score meaning from the number alone.

### Feedback
```python
make_feedback(state, features) -> str
```
Feedback should be conditional and traceable to measured signals or explicit uncertainty. It must never diagnose disease.

### Evaluation
```python
evaluate(model, split) -> dict
```
Metrics must include sample counts, label unit/policy and split details. Thresholds are selected on validation data only.

## 3. Critical scientific decision: case vs image

SCIN's dermatologist gradability labels are case-level. One case can contain up to three images, and the official repository notes duplicate image occurrences. Never randomly split image rows. Split by case ID and audit duplicates. The team must explicitly choose:
- case-level modeling/aggregation where feasible; or
- single-image training with case labels as weak/noisy supervision, described as such.

Neither option creates a direct ground-truth label for blur, exposure, or resolution. Those indicators and the gradability model answer different questions.

If this cannot be resolved within the first 1–2 hours, do not keep adding features. Prioritize:
1. data audit and label-unit documentation;
2. quality-feature baseline;
3. usable upload/feedback UI;
4. reproducible controlled-corruption tests;
5. optional classifier only when defensible.

## 4. Git flow for two beginners

Recommended:
1. Each person clones the repo.
2. Person A works on branch `person-a-data-ml`.
3. Person B works on branch `person-b-app`.
4. Commit frequently with messages describing the change.
5. Push the branch and open a pull request to `main`.
6. Review and merge only after the changed files run. Avoid both contributors editing `README.md` simultaneously; nominate one owner for final README integration.

If the deadline makes pull requests too slow, each person can make changes in their branch and coordinate a controlled merge. Do not overwrite each other's work.

## 5. Meeting cadence

- Start: 10-minute call/message to agree task definition, data source, function signatures, owners, and deadline.
- Every 2–3 hours: each person posts: what works, what is blocked, evidence/screenshots/metrics, next step.
- At integration: freeze function signatures and agree which model/config is the one used by the app.
- Final 4–6 hours: no new stretch features; fix errors, verify metrics, documentation, submission link, and recording.

## 6. Submission checklist

- [ ] Official rules and exact portal deadline confirmed.
- [ ] Public repo is accessible and contains meaningful commits inside the hacking window.
- [ ] Data source, license, and attribution are documented.
- [ ] No restricted raw images, private uploads, or secrets in GitHub history.
- [ ] Label unit, consensus policy, and case-level split are documented.
- [ ] A working app exists; optional classifier is accurately described.
- [ ] Held-out results include confusion matrix and meaningful class-specific metrics.
- [ ] Two or more controlled quality perturbations are tested if feasible.
- [ ] No disease diagnosis or clinical-adequacy claim.
- [ ] A live demo or recording, repository link, and <=500-word write-up are ready.
- [ ] Required AI tool disclosure is completed honestly.
