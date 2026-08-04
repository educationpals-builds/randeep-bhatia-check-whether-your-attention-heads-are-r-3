# Split-Walk Prompt Pack

Five standalone prompts for auditing attention-head splits. Each prompt can be pasted into any chat model. Run them in order or pick the split you need.

---

## Worked Example (Calibration)

**Specimen:** Store FAQ bot that picks an answer for shopper questions

**Standard:** If someone asks about refunds, the answer is about refunds — not shipping

**Real inputs look like:** Short mobile questions with product names in the middle

**Specimen sentences:**
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

**Decider split:** ablation

**Severity story:** The Trail Jacket shopper asked for a refund for the wrong size but the bot replied with shipping times, so the shopper is stuck holding a cart they cannot check out during the sale. Extra drift check.

**Call:** Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

**Tripwire:** Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## Prompt 1: Room Split

You are auditing whether attention heads have enough representational room to distinguish the concepts this task requires.

**Context from the worked example:**
- The specimen is a Store FAQ bot that picks an answer for shopper questions.
- Real inputs are short mobile questions with product names in the middle.
- The failure mode: shoppers ask about refunds but the bot latches onto product names and returns shipping info instead.

**Your task:**
1. List the distinct concepts the model must keep separate (e.g., "refund intent" vs. "product identifier" vs. "shipping inquiry").
2. For each concept pair that could collapse, describe what confusion would look like in the output.
3. Propose a test: feed inputs where only one concept varies and observe whether the heads maintain separation.

**Measurement this split demands:**
Count how many attention heads activate differently when "refund" appears vs. when only the product name appears, holding all other tokens constant. Report the count of heads that show > 0.1 activation difference. If fewer than 3 heads distinguish the concepts, room is insufficient.

---

## Prompt 2: Copies Split

You are auditing whether multiple attention heads are redundantly encoding the same information, wasting capacity.

**Context from the worked example:**
- The specimen is a Store FAQ bot that picks an answer for shopper questions.
- Real inputs are short mobile questions with product names in the middle.
- The failure mode: shoppers ask about refunds but the bot latches onto product names and returns shipping info instead.

**Your task:**
1. Identify which signal (product name, intent keyword, punctuation pattern) might be over-represented across heads.
2. Describe how redundant copies could crowd out the intent signal.
3. Propose a test: mask or ablate one head that encodes the product name and check if another head compensates identically.

**Measurement this split demands:**
Compute pairwise cosine similarity of attention patterns across all heads for the specimen sentences. Report the number of head pairs with similarity > 0.85. More than 2 such pairs indicates wasteful copying that may be starving intent detection.

---

## Prompt 3: Unowned Split

You are auditing whether any critical subtask has no head clearly responsible for it.

**Context from the worked example:**
- The specimen is a Store FAQ bot that picks an answer for shopper questions.
- Real inputs are short mobile questions with product names in the middle.
- The failure mode: shoppers ask about refunds but the bot latches onto product names and returns shipping info instead.

**Your task:**
1. List the subtasks required to answer correctly (e.g., detect ask-type, extract product, match policy, compose answer).
2. For each subtask, hypothesize which head or head group owns it.
3. Flag any subtask where ownership is unclear or distributed so thinly that no head dominates.

**Measurement this split demands:**
For the "detect ask-type" subtask, compute the maximum attention weight any single head places on intent-bearing tokens ("refund," "return," "cancel") across the specimen sentences. If no head exceeds 0.3 attention weight on these tokens, ask-type detection is unowned.

---

## Prompt 4: Stitch Split

You are auditing whether heads that must collaborate are actually passing information to each other.

**Context from the worked example:**
- The specimen is a Store FAQ bot that picks an answer for shopper questions.
- Real inputs are short mobile questions with product names in the middle.
- The failure mode: shoppers ask about refunds but the bot latches onto product names and returns shipping info instead.

**Your task:**
1. Map the information flow: which head's output must feed into which downstream head for correct behavior?
2. Identify any broken stitch—where Head A computes something Head B needs but the residual stream doesn't carry it forward.
3. Propose a test: trace whether the intent signal from early layers survives into the layer that selects the answer.

**Measurement this split demands:**
Track the residual-stream norm of the "refund" token embedding from layer 2 to the final layer. Report the percentage of norm retained. If less than 40% of the original norm survives to the answer-selection layer, the stitch is broken and intent is being overwritten.

---

## Prompt 5: Ablation Split (Decider)

You are auditing what happens when you remove or silence specific heads—this is the decider split for this specimen.

**Context from the worked example:**
- The specimen is a Store FAQ bot that picks an answer for shopper questions.
- Real inputs are short mobile questions with product names in the middle.
- The failure mode: shoppers ask about refunds but the bot latches onto product names and returns shipping info instead.
- **This split was rated highest priority.**

**Your task:**
1. Identify the head most suspected of causing the product-name fixation.
2. Ablate (zero out) that head and re-run the specimen sentences.
3. Document whether the failure persists, improves, or shifts to a different error.

**Measurement this split demands:**
Run the three specimen sentences with the suspected head ablated. Count how many of the three now return a refund-related answer instead of shipping. If ablation fixes 2 or more sentences, that head is the culprit. If ablation fixes 0, the bug is elsewhere—possibly downstream of ask-type classification, which matches the tripwire hypothesis (disagreement rate above 10% between standalone ask-type test and final live answer means the bug is downstream).

---

## Using This Pack

1. Paste any single prompt into your chat model along with your own specimen details.
2. Replace the worked-example context with your own tool, inputs, and failure mode.
3. Run the measurement at the end of each prompt and record the number.
4. The ablation split is the decider for the FAQ-bot specimen—start there if time is short.
5. After all splits, write your severity story on one real input, make your ship call, and set a tripwire with a threshold and an owner.

---

*Calibrated from: Store help-desk chat logs*
