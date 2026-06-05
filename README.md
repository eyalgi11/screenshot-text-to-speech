# Screenshot Text To Speech

Press Pause/Break, select a screen region with Spectacle, OCR the image with
Tesseract, synthesize the text locally, and play it through PipeWire. Press
Pause/Break again while it is running to cancel capture, OCR, speech generation,
or playback.

English/default speech uses the core `k2-fsa/OmniVoice` model integration, not
OmniVoice Studio. Hebrew-heavy OCR text can be routed to BlueTTS/Blue ONNX, a
local Hebrew-first TTS model, for better Hebrew pronunciation.

## Platform

Tested on KDE Plasma with Spectacle and PipeWire. The manual command should
work on any Linux desktop that can run Spectacle.

Pause/Break integration is session-aware:

- X11 uses a small Xlib daemon for the physical Pause/Break key.
- KDE Wayland uses KDE's global shortcut system, because Wayland blocks raw
  global key grabs by design.

## Quick Install

```sh
git clone https://github.com/eyalgi11/screenshot-text-to-speech.git
cd screenshot-text-to-speech
scripts/install-system-deps --install
scripts/install --hotkey
```

For Hebrew OCR too:

```sh
scripts/install-system-deps --install --hebrew
scripts/install --hebrew-blue
```

For an NVIDIA GPU with CUDA 12.8 PyTorch wheels:

```sh
scripts/install --cuda128 --hotkey
```

For CPU-only setup:

```sh
scripts/install --cpu
```

The installer creates `.venv`, installs OmniVoice from
`https://github.com/k2-fsa/OmniVoice`, installs `python-xlib` for the X11 hotkey
daemon, and creates `config/screenshot-tts.env` from `config/example.env` if it
does not already exist. With `--hotkey`, it installs a user systemd service that
uses the X11 daemon on X11 and the KDE global shortcut on KDE Wayland.
With `--hebrew-blue`, it also creates `.venv-blue`, installs BlueTTS/Blue ONNX,
and downloads the local Hebrew model files.

## Run

Manual test:

```sh
bin/screenshot-read
```

Cancel an active run:

```sh
bin/screenshot-read --cancel
```

Hotkey flow:

1. Press Pause/Break.
2. Drag a region in Spectacle.
3. Wait for OCR and OmniVoice synthesis.
4. The generated speech plays automatically.
5. Press Pause/Break again to cancel.

## Reference Voice

Record a short local reference voice:

```sh
scripts/record-reference-voice
```

The helper records about eight seconds into `voices/reference.wav` and writes
the prompt into `voices/reference.txt`. OmniVoice works best with a clean 3-10
second clip in the same language as the text you want it to read.

Then edit `config/screenshot-tts.env`:

```sh
STTS_REF_AUDIO=/absolute/path/to/voices/reference.wav
STTS_REF_TEXT_FILE=/absolute/path/to/voices/reference.txt
```

You can also leave the reference fields blank and use voice design:

```sh
STTS_OMNIVOICE_INSTRUCT="male, elderly, very low pitch, American accent"
```

## Hebrew TTS

For better Hebrew, install the dedicated Hebrew route:

```sh
scripts/setup-hebrew-blue
```

Then set:

```sh
STTS_HEBREW_TTS_ENGINE=blue
STTS_OCR_LANG=eng+heb
```

After OCR, the app counts Hebrew letters without logging the captured text. If
the Hebrew ratio is at least `STTS_HEBREW_ROUTE_THRESHOLD` and there are at
least `STTS_HEBREW_ROUTE_MIN_CHARS` Hebrew characters, it uses
`bin/blue-hebrew-say`; otherwise it keeps using OmniVoice.

Default model locations:

```sh
models/blue-onnx-v2
models/blue-voices/female1.json
models/renikud/model.onnx
```

## Desktop Integration

The preferred integration is session-aware:

```sh
scripts/install-desktop-integration
```

It installs a user-level service:

- on X11, it removes KDE's global Pause grab and runs the Xlib daemon;
- on KDE Wayland, it installs the KDE global shortcut and stays alive as a
  simple status service.

The historical direct installer still exists:

```sh
scripts/install-pause-daemon
```

Logs:

```sh
~/.local/state/screenshot-text-to-speech/session-hotkey-service.log
~/.local/state/screenshot-text-to-speech/pause-hotkey-daemon.log
```

If you prefer KDE's shortcut system:

```sh
scripts/install-kde-shortcut
```

If KDE does not activate the binding immediately, open System Settings ->
Keyboard -> Shortcuts, search for "Read Selected Screen Text", assign `Pause`,
and click Apply.

## Configuration

Edit `config/screenshot-tts.env`.

Important options:

- `STTS_OCR_LANG=eng` or `eng+heb`.
- `STTS_REF_AUDIO` and `STTS_REF_TEXT_FILE` for a local reference voice.
- `STTS_OMNIVOICE_INSTRUCT` for voice design without cloning a real person.
- `STTS_OMNIVOICE_DEVICE=cuda:0` to force a GPU, or leave empty for auto.
- `STTS_OMNIVOICE_SPEED=1.0`.
- `STTS_OMNIVOICE_NUM_STEP=16` for faster, rougher synthesis; `32` for default quality.
- `STTS_HEBREW_TTS_ENGINE=auto`, `blue`, or `omnivoice`.
- `STTS_HEBREW_ROUTE_THRESHOLD=0.35`.
- `STTS_KEEP_ARTIFACTS=1` to keep temporary screenshots/OCR/audio while debugging.

## Checks

Portable repo checks:

```sh
scripts/check-portable
```

Local machine readiness:

```sh
scripts/check-system
```

The checker does not download OmniVoice model weights. The first real OmniVoice
generation may download the model from Hugging Face into the normal cache.
BlueTTS model files are downloaded by `scripts/setup-hebrew-blue`.

## Uninstall

```sh
scripts/uninstall
```

Optional cleanup:

```sh
scripts/uninstall --remove-venv --remove-config --remove-state
```

## Privacy

By default, screenshots, OCR text, and generated WAV files are temporary and
stay under `XDG_RUNTIME_DIR`. Persistent logs intentionally do not include OCR
text. Local config and reference voices are gitignored.

## License

MIT for this wrapper project. OmniVoice, PyTorch, Tesseract, Spectacle, and
other dependencies have their own licenses.
