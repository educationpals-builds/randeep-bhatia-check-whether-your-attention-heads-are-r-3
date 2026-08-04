# The Head-Map Interrogator

An audit tool for checking whether attention heads are really splitting the work — built from a live inspection of a Store FAQ bot.

## The Specimen

Store FAQ bot that picks an answer for shopper questions

## The Verdict

Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

## The Tripwire

Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## One-Paste Rebuild Block

```
Specimen: Store FAQ bot that picks an answer for shopper questions
Standard: If someone asks about refunds, the answer is about refunds — not shipping
Deciding check: ablation
Call: Hold
Tripwire: >10% disagreement rate between standalone ask-type test and bot's final live answer on refund-tagged messages
Owner: ML engineer
```

---

## Using This Tool

A stranger describes any attention setup they're about to rely on — config, task, real inputs — and pastes a few of their own sentences. The tool interviews them for specimen, stakes, standard, and reality, walks the five splits conversationally, proposes candidate per-head findings and the measurement that would confirm each, and returns a scored audit with a severity story, a call, and a tripwire.

See [charter.md](charter.md) for the full audit and [METHOD.md](METHOD.md) for the five-check framework.

<!-- educationpals-build-verified -->
