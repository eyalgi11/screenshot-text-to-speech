# Screenshot Text To Speech Notes

Purpose: KDE Spectacle region screenshot -> Tesseract OCR -> OmniVoice local
TTS -> PipeWire playback.

Useful commands:

```sh
scripts/check-system
scripts/check-portable
scripts/install
scripts/setup-omnivoice
scripts/record-reference-voice
scripts/install-kde-shortcut
scripts/install-pause-daemon
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
2. `bin/screenshot-read` can select a region, OCR it, synthesize speech, and
   play the generated WAV.
3. `scripts/check-portable` passes before publishing.
4. The KDE launcher or X11 Pause daemon is installed, or a concrete desktop
   integration blocker is documented.
