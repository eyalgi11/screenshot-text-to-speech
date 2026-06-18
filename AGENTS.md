# Screenshot Text To Speech Notes

Purpose: KDE Spectacle region screenshot -> Tesseract OCR -> OmniVoice local
TTS -> PipeWire playback.

Useful commands:

```sh
scripts/check-system
scripts/check-portable
scripts/install
scripts/setup-omnivoice
scripts/setup-hebrew-blue
scripts/setup-paddleocr
scripts/stress-ocr
scripts/record-reference-voice
scripts/install-desktop-integration
scripts/install-kde-shortcut
scripts/install-pause-daemon
scripts/install-wayland-evdev-daemon
scripts/uninstall
bin/screenshot-read
bin/screenshot-read --cancel
```

Privacy rule: do not persist screenshots, OCR text, or generated speech by
default. The runtime keeps temporary artifacts under `XDG_RUNTIME_DIR`; use
`STTS_KEEP_ARTIFACTS=1` only when intentionally debugging.

Important local config:

```text
config/screenshot-tts.env
voices/reference.wav
voices/reference.txt
```

Done means:

1. `scripts/check-system` reports Spectacle, OCR, playback, and OmniVoice ready.
2. Hebrew-heavy OCR routes to BlueTTS when `STTS_HEBREW_TTS_ENGINE=blue`.
3. `bin/screenshot-read` can select a region, OCR it, synthesize speech, and
   play the generated WAV.
4. `scripts/check-portable` passes before publishing.
5. The KDE launcher, X11 Pause daemon, or Wayland evdev Pause daemon is
   installed, or a concrete desktop integration blocker is documented.
