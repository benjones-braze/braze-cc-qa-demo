# Braze Connected Content — QA-Abort Demo

Mock data + email snippet to prove a size-check + `abort_message()` on a
Connected Content response catches a **clean-200-but-incomplete** upstream
response (silently missing products) — entirely on the Braze side, with a
logged, queryable abort reason.

All data here is generic sample content. No client or production data.

## Files

| File | Purpose |
|------|---------|
| `mock_aem_full.json` | 6 products — Scenario A (passes the size check, email sends) |
| `mock_aem_partial.json` | 2 products — Scenario B (fails the check, message aborts) |
| `email_snippet_with_qa_abort.html` | Connected Content + abort logic to paste into the Braze HTML editor |
| `images/P001.jpg` … `P006.jpg` | Generic 300×300 sample product images, self-hosted here |

## Raw URLs for Connected Content

```
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/mock_aem_full.json
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/mock_aem_partial.json
```

Point the snippet at **full** for Scenario A, then swap to **partial** for Scenario B.

## Notes

- **GitHub raw is CDN-cached (~5 min).** After swapping full→partial, give it a few
  minutes or the old response keeps serving (looks like the abort failed when it didn't).
- The `${first_name}` in `greeting` is **Adobe-style** personalization syntax, not Braze
  Liquid — a `:rerender` pass will **not** resolve it and it renders literally. To make
  the name actually render, change it to `{{${first_name}}}`.
- Scope: this logic only catches **count-level** incompleteness (fewer products than
  expected). A present-but-broken field (e.g. empty `image_url` at the right count) needs
  a per-product field check inside the `{% for %}` loop.
