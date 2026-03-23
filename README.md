# AEGIS (Ambient Emergency Guardian)

Browser-based situational intelligence prototype for emergency monitoring. AEGIS combines live camera frames, motion analysis, speech recognition, geolocation, and AI vision analysis to detect risk, escalate events, and simulate dispatch workflows.

## What It Does Today

- Captures live video and microphone input from the browser.
- Computes motion intensity via frame differencing.
- Sends image + context prompts to Claude Vision for scene and threat analysis.
- Tracks voice/gesture check-in flows before escalating to dispatch.
- Shows a simulated emergency dispatch transcript and responder map.
- Keeps API keys local to the browser session (not stored server-side).

## Planned/Extensible AI Stack

Current code uses Claude Vision directly from the browser. The following components are included in the technical roadmap and provider abstraction design:

- `YOLOv26` for local/object-level detection and confidence grounding.
- `Whisper` for higher-quality speech-to-text and multilingual robustness.
- `Ollama` for local/on-device inference fallback and privacy modes.
- `Claude Vision` for high-fidelity multimodal threat assessment.
- `Backtrack session storage` for replayable timeline/session state and incident auditing.

See `TECH_SPEC.md` for the detailed architecture and integration plan.

## Tech Stack

- Vite 6
- Vanilla JavaScript (ES Modules)
- Browser APIs: WebRTC/getUserMedia, Web Speech API, Geolocation, SpeechSynthesis
- Anthropic Messages API (`claude-sonnet-4-20250514`) for image+text analysis
- Leaflet + OpenStreetMap for responder visualization

## Project Structure

```txt
src/
  main.js         # app bootstrapping and UI wiring
  camera.js       # media device lifecycle
  analysis.js     # AI scan orchestration + escalation logic
  speech.js       # browser speech recognition + TTS
  voiceCheck.js   # yes/no + gesture confirmation flow
  motion.js       # frame-diff motion computation
  dispatch.js     # simulated dispatch pipeline + map rendering
  location.js     # geolocation and nearby service lookup
  capture.js      # frame/evidence extraction
  state.js        # shared in-memory runtime state
  config.js       # prompts, colors, constants
```

## How To Run

### Requirements

- Node.js 18+ (recommended 20+)
- A modern Chromium-based browser (for best SpeechRecognition support)
- Anthropic API key

### Install

```bash
npm install
```

### Start Development Server

```bash
npm run dev
```

Then open the local Vite URL (usually `http://localhost:5173`).

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

## Usage Flow

1. Open app and enter Anthropic API key in the modal.
2. Click `INITIALIZE FEED` or `CONNECT`.
3. Allow camera, microphone, and location permissions.
4. Trigger manual `SCAN` or enable `AUTO`.
5. Review live threat classification, events, and dispatch panel.

## GitHub Push Readiness

- Includes `.gitignore` for `node_modules`, Vite output, logs, and local env files.
- No functional code paths changed by this documentation update.
- API keys are intended to remain local and should not be committed.

## Security Notes

- Do not commit real API keys or sensitive incident media.
- Browser-direct API calls are convenient for prototyping but should be replaced with a backend proxy for production.

## License

No license file is currently included. Add one before distributing publicly if needed.
