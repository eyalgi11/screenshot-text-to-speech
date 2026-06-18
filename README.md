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
- If KDE Wayland does not deliver the physical `KEY_PAUSE` shortcut, install
  the optional evdev system daemon. It listens only for Linux `KEY_PAUSE` on the
  configured keyboard event device and launches the same user command.

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

Optional stronger OCR fallback for difficult English/Latin screenshots:

```sh
scripts/setup-paddleocr
```

Then set:

```sh
STTS_OCR_FALLBACK_BIN=/absolute/path/to/bin/paddleocr-image
```

PaddleOCR's current PP-OCRv6 line is useful for many scene-OCR cases, but its
official language table does not list Hebrew. Keep Tesseract OCR for Hebrew
captures.

The installer creates `.venv`, installs OmniVoice from
`https://github.com/k2-fsa/OmniVoice`, installs `python-xlib` for the X11 hotkey
daemon, and creates `config/screenshot-tts.env` from `config/example.env` if it
does not already exist. With `--hotkey`, it installs a user systemd service that
uses the X11 daemon on X11 and the KDE global shortcut on KDE Wayland.
With `--hebrew-blue`, it also creates `.venv-blue`, installs BlueTTS/Blue ONNX,
and downloads the local Hebrew model files.

If Pause/Break does not fire on KDE Wayland but `evtest` shows `KEY_PAUSE`, add
the evdev fallback:

```sh
scripts/install-wayland-evdev-daemon --device /dev/input/by-id/usb-SEMICO_USB_Keyboard-event-kbd
```

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

On some KDE Wayland setups the physical Pause/Break key reaches the kernel as
`KEY_PAUSE` but KDE GlobalAccel does not trigger the launcher. Verify with:

```sh
sudo evtest /dev/input/by-id/usb-SEMICO_USB_Keyboard-event-kbd
```

If the press shows `code 119 (KEY_PAUSE)`, install the evdev fallback:

```sh
scripts/install-wayland-evdev-daemon --device /dev/input/by-id/usb-SEMICO_USB_Keyboard-event-kbd
```

This creates a root systemd service named
`screenshot-text-to-speech-wayland-evdev.service` because `/dev/input` is not
readable by normal users on many systems.

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
- `STTS_OCR_PREPROCESS=auto`, `off`, or `always`. `auto` keeps raw OCR when
  Tesseract confidence is healthy and retries a contrast-enhanced, sharpened
  image when OCR confidence is weak.
- `STTS_OCR_PREPROCESS_MIN_CONFIDENCE=50` and
  `STTS_OCR_PREPROCESS_MIN_WORDS=3` tune the automatic fallback.
- `STTS_OCR_QUALITY_GATE=normal`, `strict`, or `off`. `normal` blocks
  obviously unreliable OCR before TTS so bad captures do not get read aloud as
  nonsense.
- `STTS_OCR_QUALITY_MIN_CONFIDENCE=35`,
  `STTS_OCR_QUALITY_MAX_JUNK_RATIO=0.42`, and
  `STTS_OCR_QUALITY_MIN_AGREEMENT=0.60` tune that gate.
- `STTS_OCR_FALLBACK_BIN=/path/to/ocr-backend` optionally tries another OCR
  command when the primary OCR is blocked. The command must accept
  `--image`, `--output`, `--lang`, and `--psm`.
- `STTS_OCR_FALLBACK_ROUTES=default` limits fallback OCR to non-Hebrew text
  routes by default. Use `all` only for a fallback backend that you have tested
  on Hebrew.
- `STTS_PADDLEOCR_DEVICE=gpu:0` can be used with `bin/paddleocr-image` when
  PaddleOCR/PaddlePaddle is installed with GPU support. Leave empty for
  PaddleOCR's automatic device choice.
- `STTS_PADDLEOCR_ENGINE=transformers` matches `scripts/setup-paddleocr`,
  which uses the existing Torch/Transformers environment instead of requiring
  a separate PaddlePaddle runtime.
- `STTS_PADDLEOCR_LANG=auto`, `en`, or another PaddleOCR language code. `auto`
  maps `eng`/`eng+heb` OCR hints to PaddleOCR English because PaddleOCR does
  not list Hebrew support.
- `STTS_REF_AUDIO` and `STTS_REF_TEXT_FILE` for a local reference voice.
- `STTS_OMNIVOICE_INSTRUCT` for voice design without cloning a real person.
- `STTS_OMNIVOICE_DEVICE=cuda:0` to force a GPU, or leave empty for auto.
- `STTS_OMNIVOICE_SPEED=1.0`.
- `STTS_OMNIVOICE_NUM_STEP=16` for faster, rougher synthesis; `32` for default quality.
- `STTS_HEBREW_TTS_ENGINE=auto`, `blue`, or `omnivoice`.
- `STTS_HEBREW_ROUTE_THRESHOLD=0.35`.
- `STTS_BLUE_SPEED=1.05` and `STTS_BLUE_MAX_CHUNK_CHARS=80` to keep Hebrew paragraphs intelligible.
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

OCR stress test with generated hard text images:

```sh
scripts/stress-ocr
```

Targeted OCR bakeoff for the current challenge failures:

```sh
scripts/ocr-bakeoff
```

The stress test writes images, OCR text, route decisions, `report.md`, and an
inspectable self-contained `report.html` under `tmp/ocr-stress/latest`. Use
`scripts/stress-ocr --mode all` to compare raw OCR, always-preprocessed OCR,
and the app's automatic fallback.

The bakeoff writes `tmp/ocr-bakeoff/latest/report.md` and compares several
Tesseract page segmentation modes and preprocessing recipes on the failing
challenge cases. It can also compare an external backend:

```sh
scripts/ocr-bakeoff --external-bin paddleocr=bin/paddleocr-image
```

Use it before deciding whether to tune preprocessing or make a second OCR
backend part of the live fallback path.

Some generated images are required baseline cases; harder symbol-clutter,
graphic-overlay, photo-like, dark-overlay, and web-card images are challenge
cases. Challenge cases can show `CHALLENGE_FAIL` when OCR reads non-text UI
marks, chart/card/barcode graphics, decorative symbols, or meaning-changing
text as speech text, while the command only exits nonzero for required-case
failures.

The HTML report includes a sticky order control. Use it to view cases by
oldest/newest insertion order or by difficulty. The `difficulty_ladder_*`
challenge cases are generated in increasing difficulty from clean text through
low contrast, dense UI clutter, photo shadows, glare, and a severe Hebrew
notice.

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

Remove the optional Wayland evdev system service too:

```sh
scripts/uninstall --remove-system
```

## Privacy

By default, screenshots, OCR text, and generated WAV files are temporary and
stay under `XDG_RUNTIME_DIR`. Persistent logs intentionally do not include OCR
text. Local config and reference voices are gitignored.

## License

MIT for this wrapper project. OmniVoice, PyTorch, Tesseract, Spectacle, and
other dependencies have their own licenses.
