# METHOD: The Five Checks

This file spells out the five principles for auditing whether attention heads are really splitting the work.

---

## P — Partition the Space

Each head should own a distinct slice of the representation space. If heads are fighting over the same dimensions, they're not partitioning — they're competing.

**Per-head measurement:** Variance in head output norms across the sentence set. Low variance means heads aren't carving out territory.

---

## R — Run in Parallel

Heads compute simultaneously. The question is whether that parallelism produces diverse signals or redundant ones.

**Per-head measurement:** Cosine similarity between head attention patterns. High similarity means heads are running the same computation in parallel — wasted capacity.

---

## I — Individuate the Pattern

Each head should specialize on a recognizable pattern: syntax, entity type, semantic role. If you can't name what a head does, it may not be doing anything distinct.

**Per-head measurement:** Coverage of input tokens by at least one head's top-k attention. Gaps mean some patterns go unowned.

---

## S — Stitch the Spectra

Head outputs must combine into the final representation. If a head's output doesn't correlate with downstream layers, it's not contributing to the stitch.

**Per-head measurement:** Correlation between head outputs and final layer representations. Low correlation means the head is orphaned.

---

## M — Map What Each Head Sees

The ultimate test: what happens when you remove a head? If nothing changes, the head wasn't doing work. If the wrong thing changes, the head was doing the wrong work.

**Per-head measurement:** Change in output logits when each head is zeroed out. This is the ablation check.

---

## The Anti-Pattern: Collapse to Monochrome

When heads stop splitting the work, they collapse to monochrome — all heads attend to the same tokens, produce the same outputs, and the model loses its ability to distinguish between different aspects of the input.

In the FAQ bot case, this collapse meant the bot couldn't distinguish "refund" questions from "shipping" questions when both contained product names. The product name dominated all heads equally, drowning out the actual ask-type signal.

The five checks above detect this collapse before it reaches production.
