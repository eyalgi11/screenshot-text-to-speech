# Screenshot Text To Speech

Press Pause/Break, select a screen region with Spectacle, OCR the image with
Tesseract, synthesize the text locally with OmniVoice, and play it through
PipeWire. Press Pause/Break again while it is running to cancel capture, OCR,
speech generation, or playback.

This uses the core `k2-fsa/OmniVoice` model integration, not OmniVoice Studio.
The core model is the better fit here because it has a direct Python API for
`text + optional reference voice -> wav`.

## Platform

Tested on KDE Plasma X11 with Spectacle and PipeWire. The manual command should
work on any Linux desktop that can run Spectacle. The built-in Pause/Break
daemon is X11-only; on Wayland, use the KDE launcher shortcut or run the script
manually until a Wayland-specific hotkey path is added.

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
does not already exist.

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

## Desktop Integration

The preferred X11 integration is the user-level Pause/Break daemon:

```sh
scripts/install-pause-daemon
```

It logs to:

```sh
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

The checker does not download OmniVoice model weights. The first real generation
may download the model from Hugging Face into the normal cache.

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
