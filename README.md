# Milyonus Optic Tracking 2.4

Real-time, fully on-device facial optic tracking for the browser: a 478-point
face mesh, 3D head pose, per-face confidence scoring, multi-face tracking
with persistent IDs, and a focus-tracking system that samples and reports
attention once per second.

Nothing leaves the device — camera frames are processed locally with
WebAssembly/WebGL and never uploaded anywhere.

## Features

- **478-point face mesh** — full tesselation, contours and iris landmarks,
  rendered live over the mirrored camera feed.
- **3D head pose** — yaw / pitch / roll extracted from the raw rotation
  matrix, plus a relative depth estimate, visualized as an RGB axis gizmo at
  the nose tip.
- **Confidence score** — a real per-face detection-confidence value, read
  from a dedicated face detector and matched to each tracked face.
- **Face count & multi-face support** — tracks up to 4 faces per frame
  simultaneously.
- **Track ID** — a lightweight, from-scratch nearest-centroid tracker keeps a
  stable ID per face across frames (the underlying model does not expose one
  natively for multi-face scenes).
- **Focus Tracking Score** — a continuous 0–1 score derived from head yaw/
  pitch, plus a discrete gaze label (`center` / `left` / `right` / `up` /
  `down` / `none`). A "looking away" alert is logged whenever the subject
  turns away from the screen.
- **Per-second recording** — confidence and focus/gaze are sampled on a
  clean 1Hz timer, independent of the render loop.
- **Per-minute report** — all second-level samples are aggregated into a
  live-updating per-minute table (average confidence, % face detected, %
  focused, % looking away).
- **CSV export** — download the full per-second time series for offline
  analysis.
- **Expression matrix** — 8-bucket "vibe" reading (smile, sad, angry,
  surprised, fearful, disgusted, thinking, neutral) derived from 52
  ARKit-style blendshape coefficients, with arcade-style combo callouts.

## Quick start

Requires Node.js 18+ (or [Bun](https://bun.sh)).

```sh
git clone <this-repository-url>
cd milyonus-optic-tracking
npm install
npm run dev
```

Open the printed local URL, click **Start**, and grant camera access. Models
load from the network on first run (see [Privacy & models](#privacy--models)
below) and are cached by the browser after that.

Other scripts:

```sh
npm run build     # production build (outputs to dist/)
npm run preview   # preview the production build locally
npm run lint       # eslint
npm run format     # prettier --write
```

## How it works

The app runs two [MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe/solutions/vision)
graphs on every video frame, entirely in-browser:

- **`FaceLandmarker`** — 478 face landmarks, 52 blendshape coefficients, and
  a 4×4 facial transformation matrix per detected face (up to 4).
- **`FaceDetector`** — lightweight bounding boxes with a genuine detection
  confidence score, used to attach a real confidence value to each tracked
  face.

On top of the raw model output, this project adds:

- `src/lib/facePose.ts` — decomposes the column-major transformation matrix
  into yaw/pitch/roll (mirrors the standard `Matrix4.setFromRotationMatrix`
  "YXZ" Euler decomposition) plus a depth proxy from its translation.
- `src/lib/faceTracker.ts` — a minimal greedy nearest-centroid tracker that
  assigns a stable ID to each face across frames.
- `src/lib/blendshapeEmotion.ts` — heuristically combines the 52 blendshape
  coefficients into 8 expression scores.
- `src/lib/focusTracking.ts` — turns head pose into a gaze label and a 0–1
  focus score, records one sample per second, and aggregates the history
  into per-minute reports.

## Privacy & models

All inference runs locally in the browser via WebAssembly/WebGL — no video
frame or image is ever sent to a server. The only network requests this app
makes are to fetch the MediaPipe WASM runtime and the two model files (from
`jsdelivr.net` and `storage.googleapis.com`) on first load; after that,
everything runs offline from the browser cache.

## Tech stack

React 19, TypeScript, Vite, Tailwind CSS v4, and
[`@mediapipe/tasks-vision`](https://www.npmjs.com/package/@mediapipe/tasks-vision).

## Acknowledgements

Built on [MediaPipe Face Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/face_landmarker)
(Google AI Edge), used under its own license via the public
`@mediapipe/tasks-vision` package and hosted model files.

## License

[MIT](./LICENSE) — © Milyonus.
