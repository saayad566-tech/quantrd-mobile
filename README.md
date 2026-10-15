# Quant R&D Hub — Mobile (.apk)

The phone half of an offline-first quantitative research instrument. It runs a
local LLM on the device, speaks and listens through the browser's own speech
engine, and can join a desktop peer as one cluster.

**This repository is the MOBILE build.** The desktop build lives in
[`quantrd-desktop`](https://github.com/saayad566-tech/quantrd-desktop). Both share
one source tree — the split is by *packaging target*, not by feature.

## Why two repositories

| | Mobile (this repo) | Desktop |
|---|---|---|
| Artifact | signed `.apk` | signed `.exe` (NSIS) |
| Packaging | Capacitor + Gradle | Electron + electron-builder |
| Packs | Origin Private File System | OS app-data folder via IPC |
| Default role in the cluster | `mobile` (smaller models) | `desktop` (larger models) |

## What it does

- **On-device local LLM** — Ollama or an OpenAI-compatible server on loopback. The
  tiering model sizes the resident model to the device's real RAM, with a
  phone-specific reserve fraction so Android's low-memory killer stays out of it.
- **Voice talk mode** — real `SpeechRecognition` + `SpeechSynthesis`. One tap to
  speak; the intent is routed by rules first, so it works with the model switched
  off. Every state is handled and named: listening, speaking, denied, no-speech,
  unsupported.
- **Agent team** — the same four agents as the desktop, sharing one codebase.
- **Device cluster** — pair with the desktop by exchanging a pairing code, then
  send heavy jobs to the machine that can hold the bigger model. A device that has
  refused a peer cannot be re-accepted by that peer.
- **Resource packs** — chunked, resumable, hash-verified downloads with a Wi-Fi-only
  gate and a free-space pre-check. A missing pack degrades one feature; it never
  breaks the app.
- **Self-healing** — named failure kinds with recovery plans and a degradation
  ladder: full team → fewer models → one tiny model → template mode → manual mode.

## Build

```bash
npm ci
npm run build
npx cap add android
npx cap sync android
cd android && ./gradlew assembleRelease
```

See `packaging/mobile/BUILD-APK.md` for signing and the versionCode rules.

## Verify

```bash
npm run typecheck
npm test
```

## Honest limits

A real microphone, a real local model, and real backtest numbers cannot be
exercised in a headless build environment. Each has a working implementation and a
test proving the code path — the voice loop is driven against a spec-shaped mock,
and the IndexedDB path against a real IndexedDB implementation. The app says what
it cannot verify rather than inventing it.

MIT licensed. Research tool — not financial advice.
