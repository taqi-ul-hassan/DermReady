# Person B — Complete Product & Application Work Plan

**Owner:** Person B (this is Taqi's workstream)  
**Primary environment:** local VS Code for app code; Google Colab for optional model smoke tests.  
**Goal:** Make DermReady usable from upload to feedback, even before the trained classifier is ready.

## Definition of done

- The Gradio app starts with one command.
- It accepts common image formats, validates input, and fails gracefully.
- It displays measurable quality indicators and specific, cautious feedback.
- Integration with Person A's model is behind a simple function and can be disabled if their model is delayed.
- It exposes uncertainty clearly and avoids diagnosis or claims of clinical adequacy.
- The app is demonstrable on prepared, permitted examples without needing downloads during the demo.

## B0. First 15 minutes — coordinate with Person A

1. Read the README and `docs/TEAM_WORKFLOW.md`.
2. Create branch `person-b-app`.
3. Message Person A and agree the image input type and feature dictionary.
4. Ask Person A to prioritize the stable quality-feature API early; you can build the app against a temporary placeholder until their code arrives.
5. Do not wait for the training to start the UI.

## B1. Initialize the Python app skeleton

**Deliverable:** `app.py`, `requirements.txt`, and a working minimal Gradio page.

1. Create/activate a virtual environment (or use Colab temporarily):
   ```bash
   python -m venv .venv
   # Windows PowerShell:
   .venv\Scripts\Activate.ps1
   python -m pip install --upgrade pip
   pip install -r requirements.txt
   ```
2. Use Python 3.10 or 3.11 if compatible with the selected package versions. Verify rather than assume package compatibility.
3. Start with a minimal Gradio app that displays the project name, a brief purpose statement, one image upload control, and a result area.
4. Run the app locally before adding model code.

**Checkpoint B1:** You can launch the app and upload a sample image.

## B2. Build upload validation and image preview

**Deliverable:** reliable image input path and clear validation errors.

Handle:
- no file uploaded;
- corrupted/unreadable image;
- supported RGB, grayscale, and RGBA images;
- unusually small images;
- large image file/array if relevant to your environment;
- common formats such as PNG and JPEG.

Requirements:
- Convert images to a consistent representation before passing them to features/model.
- Don't modify or overwrite the original upload in a way that could hide capture issues.
- Don't print private content or image bytes in logs.
- For a public demo, use permitted example images and confirm hosting/caching behavior. Avoid persisting user uploads by default.

**Checkpoint B2:** Invalid inputs return a helpful message rather than a traceback.

## B3. Integrate Person A's quality features

**Deliverable:** quality panel in the app.

Call `extract_quality_features(image)` from Person A (or the agreed shared module). Display:
- width × height;
- blur/focus proxy with a plain-language explanation;
- brightness/exposure signals;
- contrast if implemented.

Do not label a single numeric focus value "sharpness probability." It is a heuristic. Avoid universal thresholds until Person A's evaluation supports them.

**Checkpoint B3:** A known blurry image and a known sharp image produce different measurements in expected directions, with limitations noted.

## B4. Implement understandable feedback

**Deliverable:** `src/feedback.py` or equivalent.

Create feedback from evidence that is actually measured:
- if an image is tiny, suggest moving closer or uploading a higher-resolution image;
- if the focus proxy is below a documented threshold, suggest holding the camera steady and tapping to focus;
- if brightness signals suggest an issue, suggest even lighting and avoiding harsh glare/shadows;
- if multiple signals disagree or are unavailable, return an uncertain/general suggestion.

Keep messages specific, short, non-diagnostic, and conditional. Avoid suggesting that a photo proves a skin condition is harmless or serious.

The final three UI states specified in the proposal are:
1. "Likely suitable for review"
2. "May need a retake"
3. "Uncertain — follow your clinician's instructions"

However, these labels must be driven by an agreed, documented validation policy. They are prototype states, not medically validated verdicts. If the evidence/model is not ready, show "Image checks completed — suitability not validated" rather than fabricate confidence.

**Checkpoint B4:** Every message can be traced to a measured feature, model output, or explicit uncertainty rule.

## B5. Integrate Person A's model only behind a small adapter

**Deliverable:** a stable prediction adapter that the UI can call.

Before wiring the model, obtain from Person A:
- whether it predicts case-level gradability or uses weak image-level supervision;
- the meaning and orientation of the positive class;
- required image size/transforms;
- expected input representation;
- checkpoint/config path and dependencies;
- whether the score is calibrated (do not assume it is);
- validation-selected thresholds, limitations, and evaluation results.

Example contract (adjust with Person A; do not pretend this function exists until implemented):
```python
def predict_gradability(image) -> dict:
    return {
        "status": "available",  # or "unavailable"
        "score": None,
        "predicted_label": None,
        "label_unit": "case-or-weak-image-supervision",
        "model_version": "documented-version",
    }
```

Keep inference separate from UI logic. If inference errors, show the quality-feature results and explain the model is unavailable; do not silently invent a prediction.

**Checkpoint B5:** One UI handler can use the real model if ready and the baseline-only mode if not.

## B6. Test end-to-end behaviour

**Deliverable:** `tests/` or a written manual test checklist and screenshots.

Test:
- valid JPEG and PNG;
- grayscale and RGBA image;
- invalid/corrupt file;
- no upload;
- very small image;
- model/checkpoint missing;
- unexpected prediction output;
- feature extraction error;
- repeated upload;
- slow first inference if a model is used.

Confirm that the disclaimer remains visible with every result, not hidden on a separate page.

**Checkpoint B6:** The app does not crash on common failure paths and never presents disease diagnosis.

## B7. Deployment and demo backup

**Deliverable:** a live demo if reliable, plus a local/recorded backup.

1. Keep deployment optional until the local app is stable.
2. Use only allowed data/assets in the demo.
3. Verify whether the host caches/logs files; avoid durable storage of uploads where possible.
4. Prepare several permitted demo examples stored locally/in an approved private space, not committed raw into the public repo.
5. Record a 3-minute video covering:
   - problem and intended user;
   - one upload and the measured indicators;
   - one controlled blur/brightness comparison;
   - actual held-out metrics from Person A;
   - uncertainty state;
   - non-diagnostic limitation.
6. Ensure the video does not show patient-identifying information or restricted sample imagery.

**Checkpoint B7:** You can demonstrate the project even if network hosting/downloads fail on demo day.

## B8. Person B handoff package

- [ ] `app.py` and supporting app modules
- [ ] Gradio upload UI and image preview
- [ ] Input validation and clear error messages
- [ ] Quality indicators wired to Person A's feature function
- [ ] Feedback rules and three-state policy (or honest baseline-only wording)
- [ ] Model adapter with documented fallback behaviour
- [ ] Test checklist/results
- [ ] Setup instructions and package requirements
- [ ] Screenshots/demo recording
- [ ] Short list of remaining limitations for the README

## Learning goals for Person B

Be able to explain: Python modules/functions; virtual environments; Gradio inputs/outputs; image arrays and color modes; error handling; function contracts; model inference vs training; thresholds vs calibrated probabilities; and how the user interface communicates uncertainty.

## If blocked

The UI should work before a model is ready. If the classifier is late, ship a useful upload + feature + feedback flow and honestly state that it is a research prototype whose clinical suitability decisions are not validated. Do not make up model predictions or metrics.
