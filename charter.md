# Audit Charter: Store FAQ Bot Attention Heads

## Specimen

**What tool is broken?**  
Store FAQ bot that picks an answer for shopper questions

**What goes wrong if this never gets fixed?**  
Shoppers get the wrong policy and leave the cart

**How will you know it is fixed?**  
If someone asks about refunds, the answer is about refunds — not shipping

**What the real inputs look like:**  
Short mobile questions with product names in the middle

### d_model ÷ h Arithmetic

When attention heads split the work, each head operates on a slice of the embedding dimension. If d_model = 768 and h = 12, each head sees 64 dimensions. The question: are those 64-dimension slices actually learning different patterns, or are they collapsing into redundant copies?

---

## Pasted Sentence Set

**Source:** Store help-desk chat logs

1. how long do i have to return the Nova Buds after they ship
2. Nova Buds delivery says Friday — can i still cancel
3. refund for wrong size on the Trail Jacket, not a shipping question

---

## Five Split Findings

| Check | Rating | Per-Head Measurement |
|-------|--------|---------------------|
| Room | 0 | Variance in head output norms across the sentence set |
| Copies | 0 | Cosine similarity between head attention patterns |
| Unowned | 0 | Coverage of input tokens by at least one head's top-k attention |
| Stitch | 0 | Correlation between head outputs and final layer representations |
| Ablation | 4 | Change in output logits when each head is zeroed out |

**Deciding check:** ablation

---

## Severity Story

The Trail Jacket shopper asked for a refund for the wrong size but the bot replied with shipping times, so the shopper is stuck holding a cart they cannot check out during the sale. Extra drift check.

---

## The Call

Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

---

## The Tripwire

Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## Builder's Run

This audit was built from a real inspection. The specimen sentences came from store help-desk chat logs. The ablation check scored highest because zeroing out specific heads changed the bot's answer classification — the heads aren't splitting refund-detection from shipping-detection cleanly.

The worked example above is embedded in the interrogator so it can walk strangers through the same discipline on their own attention setups.
