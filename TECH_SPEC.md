# AEGIS Technical Specification

## 1) Scope

AEGIS is a browser-first situational intelligence system for early emergency detection and guided escalation. This spec documents:

- Current architecture and behavior implemented in this repository.
- Provider abstraction for `YOLOv26`, `Whisper`, `Ollama`, and `Claude Vision`.
- Backtrack session storage design for replay, auditability, and incident forensics.

No runtime behavior is changed by this document.

## 2) Product Goals

- Detect hazards and risk patterns from live visual/audio context.
- Reduce false positives through motion + voice/gesture confirmation.
- Escalate quickly when hazard confidence is high or user is unresponsive.
- Preserve enough context to reconstruct decisions after an incident.

## 3) Current Implementation (As-Is)

### 3.1 Runtime Environment

- Frontend-only app built with Vite and vanilla JavaScript.
- Browser APIs handle media input, STT (Web Speech), TTS, geolocation.
- Anthropic Messages API performs multimodal reasoning from frame + prompt.

### 3.2 Core Modules

- `src/main.js`: wiring, tabs, initialization controls.
- `src/camera.js`: media stream lifecycle and scan controls.
- `src/motion.js`: pixel-diff motion signal and trend history.
- `src/speech.js`: live audio transcript buffering and TTS safeguards.
- `src/analysis.js`: scan prompt generation, Claude call, decision tree.
- `src/voiceCheck.js`: voice/gesture confirmation flow before dispatch.
- `src/dispatch.js`: simulated emergency call timeline and responder map.
- `src/location.js`: geolocation + nearby station discovery.
- `src/state.js`: in-memory app state (session-lifetime only).

### 3.3 Data Flow

1. Acquire camera/mic/location permissions.
2. Compute motion percentage continuously.
3. Capture compressed frame for scan.
4. Build prompt from audio summary + motion + escalation context.
5. Send to Claude Vision and parse strict JSON response.
6. Update UI, escalation levels, events, and optional dispatch workflow.

## 4) Target Multi-Model Architecture

### 4.1 Provider Layers

Define a model-router abstraction with interchangeable providers:

- `visionProvider`: `claudeVision` (default), `yolov26` (detector), hybrid mode.
- `audioProvider`: `webSpeech` (current), `whisper` (local/remote).
- `reasoningProvider`: `claudeVision` or `ollama` local LLM fallback.

Suggested interface:

```ts
interface AnalysisProviders {
  detectObjects(frame: Base64Image): Promise<DetectionSet>;      // YOLOv26
  transcribeAudio(chunk: AudioChunk): Promise<Transcript>;       // Whisper
  analyzeScene(input: SceneContext): Promise<ThreatAssessment>;  // Claude/Ollama
}
```

### 4.2 YOLOv26 Integration Role

- Local object/hazard primitives: person count, posture hints, fire/smoke/weapon priors.
- Output used as grounded signals for reasoning prompts.
- Can run on-device (WebGPU/WebAssembly wrapper) or sidecar service.

### 4.3 Whisper Integration Role

- Replace or augment Web Speech API.
- Better confidence and multilingual handling.
- Supports buffered chunk transcription and confidence thresholding.

### 4.4 Ollama Integration Role

- Local privacy-preserving reasoning fallback.
- Offline/air-gapped operation mode.
- Policy: use Claude for high-risk confidence paths when cloud available; Ollama for continuity/fallback.

### 4.5 Claude Vision Integration Role

- High-fidelity multimodal scene reasoning.
- Produces structured JSON threat model consumed by UI and escalation logic.
- Remains primary cloud analysis path in current implementation.

## 5) Backtrack Session Storage

### 5.1 Objective

Capture a replayable incident timeline without altering active decision paths.

### 5.2 Storage Model

- **Session metadata**: session id, start/end, device, model provider versions.
- **Time-series events**: scan outputs, motion values, transcripts, escalation changes.
- **Evidence pointers**: frame hashes/URIs, optional encrypted blobs.
- **Decision trace**: why dispatch/de-escalation happened (rule + threshold + source).

### 5.3 Proposed Schema (Conceptual)

```json
{
  "session_id": "uuid",
  "started_at": "iso8601",
  "timeline": [
    {
      "t": "iso8601",
      "type": "analysis_result",
      "motion": 23,
      "threat_level": "HIGH",
      "hazard_detected": "smoke",
      "provider": "claude-vision"
    }
  ],
  "dispatch": [],
  "version": 1
}
```

### 5.4 Persistence Options

- **Local-first**: `IndexedDB` for browser session continuity.
- **Sync target**: optional API endpoint for durable cloud retention.
- **Backtrack UI**: filter/search by time, severity, and provider.

## 6) Non-Functional Requirements

- **Latency**: visual scan response target < 2.5s median on healthy network.
- **Reliability**: graceful degradation across provider outages.
- **Privacy**: minimal retention by default; explicit retention policy gates.
- **Auditability**: traceable model/provider output tied to each escalation.

## 7) Security & Compliance Considerations

- Never commit secrets; API keys remain local/user-supplied.
- Avoid browser-direct production keys long term; introduce signed backend proxy.
- Encrypt stored evidence/session records at rest where possible.
- Implement retention windows and purge controls for sensitive data.

## 8) Deployment Readiness (Docs/Repo)

- Add `.gitignore` and avoid checking in generated artifacts.
- Keep architecture docs explicit about current vs planned providers.
- Preserve functional parity while improving documentation quality.

## 9) How To Run (Current Repo)

```bash
npm install
npm run dev
```

Open Vite URL (typically `http://localhost:5173`), enter Anthropic API key, then connect camera/mic/location.

## 10) Roadmap (No-Code Changes)

1. Introduce provider interfaces and feature flags.
2. Add YOLOv26 detector adapter and confidence fusion.
3. Add Whisper adapter with transcript confidence metadata.
4. Add Ollama fallback for offline/private reasoning mode.
5. Implement IndexedDB-based backtrack session store.
6. Build replay/audit UI panel.
