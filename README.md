# DermReady
### Image-AI Quality Gate for Dermatology Photography

DermReady is a research prototype that helps people preparing photographs for a remote dermatology consultation notice potentially correctable image-capture problems before sending the photos for clinical review.

**DermReady does not diagnose skin disease, screen for cancer, or guarantee that an image is clinically adequate.** Its output is not a substitute for clinical judgement or medical advice.

## Project status

**Stage:** Initial repository setup — data audit and MVP implementation pending.

Our priority is a small, reproducible, honest system:
1. Audit the Skin Condition Image Network (SCIN) dataset and its data-use license.
2. Build interpretable image-quality indicators for blur, exposure/brightness, and resolution.
3. Compare those indicators with a pretrained image classifier if the data and time support it.
4. Evaluate on held-out cases and inspect errors.
5. Provide a simple Gradio application with cautious feedback and an uncertainty state.
6. Publish the methods, results, limitations, and a short demo.

## Important dataset-label limitation

SCIN's dermatologist gradability labels describe whether the skin condition can be determined from the images in a **case**. An ungradable label can reflect several reasons, including insufficient information, multiple conditions, no discernible pathology, or image-quality issues. This is **not** a pure label for blur, bad exposure, or low resolution.

The data audit must establish the unit of analysis and document any use of a case-level label as weak supervision for individual images. Keep related images from the same case in one split, and check for duplicate images across cases/splits.

## Expected demo

The user uploads an image and sees:
- Measured quality indicators where implemented.
- Retake suggestions supported by those indicators.
- One cautious status: **Likely suitable for review**, **May need a retake**, or **Uncertain**.
- A clear research-prototype disclaimer.

Statuses and thresholds must be based on documented validation results. Do not imply clinical validation.

## Repository structure

```text
app.py
requirements.txt
README.md
LEARNING_LOG.md
src/
  quality_features.py
  data_loader.py
  model.py
  train.py
  evaluate.py
  feedback.py
notebooks/
  01_data_audit.ipynb
  02_baseline.ipynb
docs/
  PERSON_A_TASKS.md
  PERSON_B_TASKS.md
  TEAM_WORKFLOW.md
reports/
  metrics_summary.csv
  figures/
tests/
```

We can add modules only when needed; this is a target structure, not a requirement to create every file before the MVP works.

## Setup

The initial dependency list is provisional. Verify compatibility in the actual Colab/local environment before pinning versions.

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

For the initial data audit and model training, use Google Colab if local compute is limited. Do not assume SCIN images are present in this repository; obtain the dataset through its official source and follow its license.

## Data and privacy rules

- Read the [SCIN Data Use License](https://github.com/google-research-datasets/scin/blob/main/LICENSE) before sharing any dataset content or derivatives.
- Read the [official dataset schema](https://github.com/google-research-datasets/scin/blob/main/dataset_schema.md).
- Do not commit raw dataset images, private user uploads, credentials, or model checkpoints/weights to this public repository.
- Do not attempt to re-identify dataset contributors.
- Avoid retaining user-uploaded images after inference; confirm hosting and caching behaviour before deploying a public demo.
- Do not claim clinical benefit, fairness, or clinical adequacy without appropriate evidence.

## Team ownership

- **Person A — data/ML:** schema and license audit, labels, leakage-resistant splits, quality-feature baseline, model training (if feasible), evaluation, and measured results.
- **Person B — product/integration:** Gradio UI, input validation, image-quality display, feedback copy, model integration, edge-case testing, and demo recording.
- **Both:** agree on label semantics and function contracts, verify results, document limitations and citations, test the end-to-end flow, and rehearse the demo.

Detailed task breakdowns: [Person A tasks](docs/PERSON_A_TASKS.md) · [Person B tasks](docs/PERSON_B_TASKS.md) · [Team workflow](docs/TEAM_WORKFLOW.md).

## Evaluation

Report more than accuracy. As appropriate to the final label policy, include a confusion matrix, recall for ungradable-labelled cases, false-acceptance and false-rejection rates, F1, balanced accuracy, and threshold-specific results. Choose thresholds on validation data—not on the final test set. Record the number of cases/images and the label policy alongside metrics.

Controlled blur, brightness, downsampling, and compression tests are stress tests; they do not reproduce all real-world image failures or prove clinical benefit.

## References

- [SCIN official repository](https://github.com/google-research-datasets/scin)
- [SCIN dataset schema](https://github.com/google-research-datasets/scin/blob/main/dataset_schema.md)
- [SCIN Data Use License](https://github.com/google-research-datasets/scin/blob/main/LICENSE)
- [Banana Hacks 2026 FAQ](https://www.bananahacks.tech/faq)
- [Banana Hacks submission rules](https://www.bananahacks.tech/code-of-conduct)

## AI-tools disclosure

Complete the required disclosure when submitting to Banana Hacks. The team should verify, understand, and take responsibility for all code, results, and claims.
