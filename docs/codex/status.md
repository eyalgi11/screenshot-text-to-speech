# Codex Status

## 2026-06-18 OCR Fallback Bakeoff

- Added optional PaddleOCR fallback plumbing through `bin/paddleocr-image`,
  `scripts/setup-paddleocr`, and `scripts/ocr-bakeoff --external-bin`.
- Verification passed with `scripts/check-portable`, `scripts/check-system`, and
  `scripts/stress-ocr`.
- Latest bakeoff report: `tmp/ocr-bakeoff/latest/report.md`.
- PaddleOCR helped on several difficult English/Latin challenge images,
  especially `ambiguous_symbol_panel`, `graphic_receipt_barcode`, and
  `message_with_avatar_graphics`.
- Keep PaddleOCR route-limited to `default` and do not enable it for Hebrew:
  the official PP-OCRv6 language table does not list Hebrew, and the bakeoff
  misrouted Hebrew challenge images as default text.
- Good next step: improve the OCR fallback gate so it catches semantic/noise
  failures before turning PaddleOCR on for live reading.
