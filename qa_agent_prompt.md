# Braze QA Agent — Product Render Check (prompt)

Use this as the instruction/prompt for a Braze Canvas QA agent (or any LLM
QA step) that inspects the **rendered HTML** of a message before it sends and
flags it when the product recommendations are incomplete or broken.

The point of the HTML variant: unlike the JSON demo, there is no structured
`products` array to count in Liquid. The agent reads the HTML the customer
would actually receive and reasons about it the way a human QA reviewer would.

---

## System / role prompt

```
You are a QA agent for marketing email. You are given the fully rendered HTML
of a single email and a set of QA rules. Your only job is to decide whether the
product recommendation section is complete and fully rendered, and to return a
strict verdict. You do not rewrite or fix the email — you only inspect and report.
```

## Configurable inputs

```
EXPECTED_PRODUCT_COUNT = 6      # change per campaign (e.g. 2, 4, 6)
MIN_PRODUCT_COUNT      = 6      # minimum to allow send; often == expected
REQUIRED_FIELDS        = [image, name, price, product_link]
```

## Task prompt

```
Inspect the email HTML below.

1. COUNT the product cards in the recommendations section. A product card is
   each element carrying a data-product="..." attribute (fallback: each block
   that has a product image + a product name + a price together).

2. For EACH card, verify every REQUIRED_FIELD is present and non-empty:
   - image      : an <img> with a non-empty src that is an absolute https URL
   - name       : visible product name text, not blank and not a placeholder
                  (e.g. not "{{...}}", "null", "undefined", "PRODUCT_NAME")
   - price      : a visible price value, not blank / not a placeholder
   - product_link: an <a href="..."> wrapping the card with a non-empty https URL

3. Flag a card as BROKEN if any required field is missing, empty, or still a
   template placeholder / unresolved Liquid or ${...} token.

4. Compare the count of FULLY RENDERED cards against MIN_PRODUCT_COUNT.

Return ONLY this JSON, nothing else:

{
  "expected": <MIN_PRODUCT_COUNT>,
  "found_cards": <int>,
  "fully_rendered": <int>,
  "broken": [ { "position": <int>, "product_id": "<data-product or ''>", "missing": ["field", ...] } ],
  "pass": <true if fully_rendered >= MIN_PRODUCT_COUNT and broken is empty, else false>,
  "reason": "<one short sentence>"
}
```

## Expected results against the sample files

| File | found_cards | fully_rendered | MIN=6 → pass? |
|------|-------------|----------------|---------------|
| `product_block_6.html` | 6 | 6 | ✅ true |
| `product_block_4.html` | 4 | 4 | ❌ false ("only 4 of 6 products") |

Set `MIN_PRODUCT_COUNT = 2` and `product_block_4.html` passes — that's the
configurable threshold in action.

## How to wire it into Braze

- **Canvas:** send the message into a step whose action is gated on the agent
  verdict, OR run the agent as a pre-send QA step and branch on `pass`.
- **Feed the agent the rendered HTML**, not the template. Use the message
  preview / rendered output so Liquid and Connected Content are already resolved
  — otherwise the agent just sees `{{cc_html}}` and can't judge anything.
- On `pass:false`, route to a "hold / alert" path (e.g. notify a Slack channel,
  skip the send, or fall back to a static block) instead of mailing the gap.
