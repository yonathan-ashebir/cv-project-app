# Sign2Text Frontend (ArSL Word-Level)

Single-page web client that captures webcam video, extracts MediaPipe hand landmarks in the browser, buffers a 32-frame sequence, and sends it to the FastAPI `/predict` endpoint for Arabic Sign Language word-level classification.

## Quick Start
- Prerequisites: modern desktop browser (WebGL enabled) + backend running at `http://localhost:8000/predict` (or your own origin).
- Start backend (example):
  ```bash
  uvicorn arsl-word-level-detection.api_server:app --reload
  ```
- Serve the static files:
  ```bash
  cd frontend
  python -m http.server 4173
  ```
- Open `http://localhost:4173`, allow webcam access, leave Backend URL as-is (or point to your server), click **Start Recording**, then **Show Prediction** once enabled.

## How It Works
- **Capture & Landmarks:** Uses `navigator.mediaDevices` + MediaPipe Hands (via CDN) to stream frames and detect up to 2 hands.
- **Normalization:** Each hand’s 21 landmarks are mean-centered and L2-normalized; missing hands are zero-padded.
- **Buffering:** A 5s recording collects raw frames, then resamples to a fixed 32-frame buffer to match the training pipeline (`SEQUENCE_LENGTH`, `FEATURE_DIM` in `main.js`).
- **Inference:** Sends `POST { sequence: buffer }` to the backend; displays top prediction + confidence in the UI and a dialog.
- **Visualization:** Draws landmarks/connectors over the mirrored video feed and shows progress during recording.

## Files
- `index.html` — Layout and MediaPipe script tags (Camera Utils, Drawing Utils, Hands).
- `main.js` — Capture pipeline, landmark normalization, resampling, API call, and UI state.
- `styles.css` — Dark theme, layout for video canvas, controls, progress bar, and dialog.

## API Contract
- Endpoint: `POST /predict`
- Body:
  ```json
  {
    "sequence": [[...126 floats...], "...32 rows total..."]
  }
  ```
- Response (expected):
  ```json
  { "prediction": "WORD_LABEL", "confidence": 0.92 }
  ```

## Customization
- Backend URL: change default in `index.html` (`#api-url` input value) or set manually in the UI.
- Recording length / sequence size: adjust `RECORD_DURATION_MS` and `SEQUENCE_LENGTH` in `main.js`; keep them aligned with backend expectations.
- Styling: edit `styles.css` for colors/layout; canvas uses a mirrored feed to match user motion.
- Hosting: any static host (Netlify, GitHub Pages, S3). If cross-origin, enable CORS on the FastAPI app and use HTTPS for both sides.

## Troubleshooting
- Empty prediction / disabled button: ensure recording completes (progress bar fills) so the buffer reaches 32 frames.
- Hand not detected: increase lighting or tweak `minDetectionConfidence` / `minTrackingConfidence` in `main.js`.
- CORS errors: allow the frontend origin in the FastAPI CORS config or serve both from the same host/port.
