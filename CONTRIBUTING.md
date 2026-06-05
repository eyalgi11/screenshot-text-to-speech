# Contributing

Thanks for helping make this more usable on real Linux desktops.

## Local Checks

Run these before opening a pull request:

```sh
scripts/check-portable
scripts/check-system
```

`scripts/check-portable` is the lightweight CI-style check. `scripts/check-system`
checks the local desktop tools and OmniVoice environment.

## Privacy

Do not commit screenshots, OCR text, generated speech, reference voices, or
machine-specific config. The repo ignores `config/screenshot-tts.env` and
everything under `voices/` except `.gitkeep` for that reason.

## Portability

Prefer installer-time path rendering over hardcoded user paths. Scripts should
derive the project root from their own location and respect `XDG_CONFIG_HOME`,
`XDG_STATE_HOME`, `XDG_DATA_HOME`, and `XDG_RUNTIME_DIR` where practical.
