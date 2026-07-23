# Braze Connected Content — QA-Abort Demo

Mock data + email snippet to prove a size-check + `abort_message()` on a
Connected Content response catches a **clean-200-but-incomplete** upstream
response (silently missing products) — entirely on the Braze side, with a
logged, queryable abort reason.

All data here is generic sample content. No client or production data.

## Files

### JSON variant (Liquid size-check + abort)

| File | Purpose |
|------|---------|
| `mock_aem_full.json` | 6 products — Scenario A (passes the size check, email sends) |
| `mock_aem_partial.json` | 2 products — Scenario B (fails the check, message aborts) |
| `email_snippet_with_qa_abort.html` | Connected Content + abort logic to paste into the Braze HTML editor |

### HTML variant (CC returns rendered HTML + LLM QA agent)

| File | Purpose |
|------|---------|
| `product_block_6.html` | Full email (header/content/6 products/footer) — Scenario A |
| `product_block_4.html` | Same shell, only 4 products — Scenario B (incomplete) |
| `email_snippet_html_response.html` | Connected Content call that injects the HTML block |
| `qa_agent_prompt.md` | Prompt for a Braze Canvas / QA agent that inspects the rendered HTML |

### Shared

| File | Purpose |
|------|---------|
| `images/P001.jpg` … `P006.jpg` | Generic 300×300 sample product images, self-hosted here |

## Raw URLs for Connected Content

```
# JSON variant
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/mock_aem_full.json
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/mock_aem_partial.json

# HTML variant
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/product_block_6.html
https://raw.githubusercontent.com/benjones-braze/braze-cc-qa-demo/main/product_block_4.html
```

Point the snippet at **full/6** for Scenario A, then swap to **partial/4** for Scenario B.

## JSON vs HTML — which check catches what

- **JSON variant:** the response is structured, so Liquid can read
  `aem_response.products.size` and `abort_message()` natively. Precise, but only
  works when the feed returns parseable JSON.
- **HTML variant:** the response is a rendered blob. Liquid can only do a crude
  count (`split` on the `data-product="` marker — included in the snippet), and
  it **cannot** tell a present-but-broken card from a good one. That gap is what
  the **LLM QA agent** (`qa_agent_prompt.md`) is for: it reads the HTML the
  customer would receive and reasons about completeness the way a human QA would.

## Notes

- **GitHub raw is CDN-cached (~5 min).** After swapping full→partial, give it a few
  minutes or the old response keeps serving (looks like the abort failed when it didn't).
- The `${first_name}` in `greeting` is **Adobe-style** personalization syntax, not Braze
  Liquid — a `:rerender` pass will **not** resolve it and it renders literally. To make
  the name actually render, change it to `{{${first_name}}}`.
- Scope: this logic only catches **count-level** incompleteness (fewer products than
  expected). A present-but-broken field (e.g. empty `image_url` at the right count) needs
  a per-product field check inside the `{% for %}` loop — or, for the HTML variant, the QA agent.
- **Previewing the HTML files:** GitHub raw serves `.html` as `text/plain`, so the raw
  URL shows source, not a rendered page (fine for Connected Content — Braze just takes
  the body). To eyeball the design, open the file locally or use
  `https://htmlpreview.github.io/?<raw-url>`.
