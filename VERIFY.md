# Stranger Verification

This file describes how a stranger can verify that the Head-Map Interrogator works as designed.

---

## Verification Steps

### 1. Run the Seeded Specimen Through /play

Use the interrogator with the following specimen:

**Specimen:** Store FAQ bot that picks an answer for shopper questions

**Test sentences:**
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

### 2. Confirm the Unowned-Relationship Finding

The tool must surface the ablation finding — that zeroing out heads changes the ask-type classification. This is the "unowned relationship" between refund-intent and shipping-intent that the bot fails to separate.

**Expected behavior:** The interrogator identifies that the ablation check is the deciding factor (rated 4 out of 4) and explains why heads aren't cleanly splitting refund-detection from shipping-detection.

### 3. Demand a Per-Head Number

For the ablation finding, the tool must propose a specific per-head measurement:

> Change in output logits when each head is zeroed out

If the tool returns a vague finding like "attention seems off" without a measurable per-head number, verification fails.

---

## Expected Output Structure

A passing verification produces:

1. **Severity story** — A specific failure with a real sentence, wrong output, and who acts on it
2. **Call** — Ship / ship-with-conditions / hold, with checkable conditions and owners
3. **Tripwire** — A metric, a threshold, and who watches it

---

## Reference: The Builder's Audit

The builder's own audit is embedded as the worked example. Compare your verification run against:

- **Deciding check:** ablation
- **Call:** Hold
- **Tripwire:** >10% disagreement rate between standalone ask-type test and bot's final live answer on refund-tagged messages; ML engineer owns this check

If the interrogator walks you through the same discipline and demands the same rigor, verification passes.
