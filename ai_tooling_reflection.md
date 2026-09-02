# AI tooling reflection

**SMG Stage 2 — accompanying note**  
Alongside: *Booking lifecycle and contention*

The brief asked how I directed, evaluated and corrected AI, not whether I could work from a blank page. This is that note.

I used VS Code for the source Markdown, ChatGPT for first-pass structuring and race lists, and Grok to review drafts against the brief and to pressure-test the allocation path. Gemini was used briefly for an early data-model sketch, then discarded when it started to look like a generic booking schema.

---

## Where it was genuinely useful, and how I prompted it

AI was useful as a **structure and adversary**, once I already had an angle.

I did not prompt “design a retail media platform.” I prompted with constraints:

- the four questions from the brief;
- “availability is not a stored field”;
- “do not invent named physical slots”;
- “walk two traders taking the last unit, then expiry versus confirm, then retry after commit”;
- “keep authorised extra capacity distinct from accidental over-allocation.”

That produced:

- a stable skeleton for the four required answers;
- a first list of races I could then accept or reject;
- consistent domain language (position vs campaign, hold vs confirm, case vs incident);
- a review pass on later drafts that caught `PARTIAL` on a Hold response — which would have meant silent quantity shrink.

The useful pattern was: **state the invariant, ask what breaks it, keep the output as a draft.**

---

## Where it was wrong, and I had to push back

Several answers were fluent and incorrect. These are the ones I would still reject in a live exercise.

**Confirm as lock-free.**  
The model argued that because `held − 1` and `confirmed + 1` leave total consumed unchanged, Confirm need not lock `inventory_position`. That loses the aggregate pair a concurrent Hold is reading. Confirm is shorter than Hold. It is not lock-free.

**One “oversell” number.**  
Physical overcapacity, authorised extra, uncovered overallocation and confirmed shortfall were collapsed into a single risk field. That is how a defect gets filed as a commercial decision.

**Confirmed flowing back to Released.**  
An early state diagram allowed confirmed inventory to move back into released. The brief says a confirmed booking stands.

**Silent shrink.**  
Given requested 4 and bookable 2, drafts offered `Hold 2` or a bulk result of `PARTIAL` as if the allocator could choose the split. The advertiser’s consent is the split. The command either takes the consented quantity or writes nothing and returns a new offer.

**Allowance as more physical panels.**  
Extra capacity was treated as changing physical capacity from 6 to 7. Physical stays 6. The allowance is separate, prior, and auditable.

**Coverage as a “cryptographic” link.**  
A foreign key plus an audit row is enough. Calling it cryptographic is decoration.

**False edit instructions.**  
A review pass told me to delete a Quantity-section lead-in that did not exist. I had to check the file rather than trust the comment. That is the same habit the interview will test.

I also did not let the tool decide that “exception” in the brief meant hold-stealing. I listed three readings (displace a hold / sell above physical / bend hold policy) and chose the oversell reading because the brief ties exceptions to accepting fulfilment risk. AI can enumerate. It should not close an ambiguity the brief left open.

---

## What I deliberately did not use AI for

**Choosing the angle.**  
Booking lifecycle and contention was my call. The tool will happily write all three example angles at equal length. Depth is a human rationing decision.

**What counts as inventory truth.**  
Demand is not stock. Availability is a snapshot. The write transaction is authority. I would defend those without a prompt.

**The handwritten working notes.**  
Actors, “but I really need one → exception?”, and the first domain sketch were done on paper before I asked a model to tidy anything.

**What to cut.**  
The Confirm section grew into a second paper. I cut the repeated “what Confirm is allowed to do” and the ten-bullet “what this prevents” list so Hold and Confirm were the same kind of artefact. A model will add sections. It will not spend your two hours.

**Live SMG org charts.**  
I did not ask a model to infer the trader’s employer from current careers pages. The brief left that unspecified. The design only needs: a human operator assembling demand against one retailer’s estate.

---

## How I would work in the interview

I would keep the same loop:

1. State the invariant in one sentence.
2. Ask the tool for a race or a transaction sketch against that sentence.
3. Check whether the output invents a second source of truth, a silent shrink, or a lock it cannot justify.
4. Keep or throw away by that test.

The question I care about is not whether the assistant can write Markdown. It is whether I still know what the business is allowed to promise when the last aisle barrier is gone.
