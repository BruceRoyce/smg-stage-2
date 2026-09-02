# Appendix — Working Interpretation of the Technical Brief

## Purpose

This appendix records the working interpretation of the Stage 2 Technical Task before designing the solution.

Its purpose is to make explicit:

- what the original brief **states**;
- what can reasonably be **derived** from those statements;
- what remains genuinely **ambiguous**;
- what **assumptions** are being adopted so that the exercise can proceed;
- and which consequences are merely **design implications**, rather than additional requirements.

The original technical brief remains the source of truth. If anything in this appendix conflicts with it, the original brief takes precedence.

The following labels are used throughout:

**BRIEF** — directly stated or unambiguously established by the original task.

**DERIVED** — a conclusion that follows from combining statements in the brief, but is not stated verbatim.

**ASSUMPTION** — a deliberate choice made because the brief leaves something unspecified.

**AMBIGUITY** — something for which more than one reasonable interpretation remains possible.

**DESIGN IMPLICATION** — a consequence worth considering when designing the solution, but not itself a requirement supplied by the brief.

---

# 1. The problem being presented

## BRIEF

SMG operates a retail media platform connecting Advertisers with Publishers, described in this scenario as large Retailers and their owned media channels.

A significant part of the inventory being sold is physical media space inside Retailer stores, including examples such as:

- trolley panels;
- aisle barriers;
- gondola ends;
- window vinyls.

A Store carries a number of units of a particular Format. The brief gives the example of six aisle barriers.

Each unit is independently bookable, meaning multiple Advertisers can use units of the same Format in the same Store during the same Cycle.

Media is sold only in fixed four-week Cycles aligned to the Retailer's trading calendar. A Campaign may run for one or more consecutive Cycles. Arbitrary start and end dates and mid-Cycle changes are outside the problem.

The brief specifically warns that although this initially resembles a calendar problem:

> it is not fundamentally a calendar problem.

The difficulty comes from booking finite inventory over time while other users may simultaneously be attempting to secure the same inventory.

## DERIVED

A more useful description of the technical problem is:

> **A contested finite-inventory booking system in which physical media capacity can be provisionally held, confirmed, released by expiry, partially secured, and potentially oversold through an explicit commercial exception.**

The Cycle determines **when** a booking applies.

The harder problem is determining **what inventory is legitimately committed at that moment and whether another booking may also acquire it.**

---

# 2. Why correctness matters commercially

## BRIEF

The booking data does not stop at the booking application.

The brief states that downstream:

- field teams use the data to determine what must physically be installed;
- finance uses it to determine what should be invoiced.

It explicitly warns that if bookings and physical reality diverge, an Advertiser can ultimately be charged for media that never ran.

## CLARIFICATION

This means inventory correctness is not simply an internal technical concern.

A false-positive booking can propagate:

```text
Incorrect booking
       │
       ▼
Installation expectation
       │
       ▼
Physical media unavailable
       │
       ▼
Campaign not delivered as expected
       │
       ▼
Incorrect / disputed invoicing
```

This is why deliberate overselling must be distinguishable from accidental over-allocation.

The system needs to preserve the difference between:

> **“The business knowingly accepted fulfilment risk.”**

and:

> **“The software accidentally sold the same finite capacity twice.”**

The first may be a permitted commercial decision.

The second is a correctness failure.

---

# 3. The actors

The original brief uses three important business terms:

- Advertiser;
- Trader;
- Retailer / Publisher.

Their exact organisational relationship is not completely defined.

---

# 4. Advertiser

## BRIEF

The brief states that:

> an Advertiser books a Format across a set of Stores for one or more Cycles.

It later says that a Trader assembles a Campaign provisionally while:

> the Advertiser decides.

## DERIVED

The Advertiser is therefore the commercial party whose advertising requirement or Campaign is being fulfilled.

A useful **mental analogy**, but not a formal requirement, is:

> Advertiser ≈ brand/customer whose advertising is to appear in stores.

For example, a soft-drink brand might want a particular Format across a selected Store estate during a launch period.

## IMPORTANT LIMIT

The original brief does not establish:

- who legally contracts with whom;
- whether the Advertiser directly pays SMG;
- whether an agency pays;
- how billing relationships work.

Pricing and invoicing mechanics are explicitly outside scope.

Therefore this design should not depend on calling the Advertiser “the payer.”

---

# 5. Trader

## Question raised during interpretation

Could “Trader” mean the Retailer that supplies the Store estate?

That interpretation initially appeared plausible from the word alone, but it does not fit the complete brief.

## BRIEF

The scenario contains:

- one Retailer;
- approximately 3,000 Stores;
- approximately 20–30 Traders working concurrently during peak periods.

It also says:

> a Trader assembles a Campaign over days or weeks, holding space provisionally while the Advertiser decides.

## DERIVED

“Trader” therefore describes a **human/operator role**, not the Retailer business entity itself.

A safe definition for this exercise is:

> **A Trader is a human commercial operator who assembles Campaigns and interacts with the booking process while working with Advertiser demand.**

It is reasonable to think of this role as performing work analogous to media planning, campaign trading or agency-style campaign management.

However:

> **Trader = Advertising Agency**

is **not** established by the brief and should not be treated as fact.

## AMBIGUITY

The brief does not identify who employs a Trader.

A Trader could theoretically be:

- an SMG employee;
- a Retailer-side operator;
- an agency employee;
- another authorised commercial user.

Nothing in the core inventory problem requires this to be decided.

## ASSUMPTION

For the exercise:

> **Trader will be modelled as a system actor/role rather than as an organisation.**

The design should therefore depend on what a Trader is permitted to do, not on who employs them.

---

# 6. Retailer / Publisher

## BRIEF

The scenario deliberately restricts the exercise to:

- one Retailer;
- a fixed Store estate;
- one Retailer trading calendar.

The Retailer owns the physical context in which the media exists.

## CLARIFICATION

A useful shorthand is:

> **Retailer ≈ supplier/owner of the physical media estate.**

The Retailer therefore provides the Store estate and the physical capacity that can ultimately be sold.

---

# 7. Store

## BRIEF

A Store is part of the fixed Retailer estate.

A Store carries physical units of one or more Formats.

## CLARIFICATION

A useful shorthand is:

> **Store ≈ physical host of the media inventory.**

The actual domain terminology should remain `Store`, because this is the term used by the brief.

---

# 8. Overall actor relationship

Based only on what is necessary to reconcile the brief, the working relationship is:

```text
ADVERTISER
commercial advertising requirement
        │
        │ decisions / campaign intent
        ▼
TRADER
human operator assembling the Campaign
        │
        │ attempts to hold / confirm inventory
        ▼
SMG BOOKING PLATFORM
        │
        │ operates against
        ▼
RETAILER MEDIA ESTATE
        │
        ▼
STORES / FORMATS / CYCLES
```

This diagram describes roles in the problem.

It does **not** prescribe employment, contractual or billing relationships.

---

# 9. Do Advertisers also use the application?

## Question raised during interpretation

If an Advertiser wants media and a Trader subsequently books it, does the system require:

1. an Advertiser-facing booking UI;
2. followed by a second Trader-facing booking UI;
3. where the Trader effectively enters the same requirement again?

Nothing in the brief requires this duplication.

## BRIEF

The brief specifically offers as one possible area of exploration:

> the interface or API a Trader assembles a Campaign through.

It also describes the Trader as assembling the Campaign while the Advertiser decides.

## ASSUMPTION

For this exercise:

> **The Trader is treated as the primary interactive user of the booking workflow. No Advertiser self-service application is assumed.**

Communication between Advertiser and Trader may happen outside the system or through functionality not relevant to this task.

This assumption limits scope; it does not assert that SMG has no Advertiser-facing systems in reality.

---

# 10. Campaign

## BRIEF

A Campaign:

- runs for one or more consecutive Cycles;
- is assembled by a Trader over days or weeks;
- may contain inventory held provisionally while the Advertiser decides.

## AMBIGUITY

The brief does not formally define Campaign ownership or its exact internal data structure.

It would therefore be too strong to state:

> “The Campaign belongs to the Advertiser”

or:

> “The Campaign belongs to the Trader.”

## DERIVED

What can safely be said is:

> **A Campaign represents advertising activity associated with an Advertiser and is assembled/managed through the booking process by a Trader.**

Any concept such as `CampaignLine` is a possible implementation model, not terminology mandated by the brief.

---

# 11. Trading Cycles

## BRIEF

Media is sold only in fixed four-week Cycles aligned to one Retailer trading calendar.

A Campaign runs for one or more **consecutive** Cycles.

Nothing starts or ends mid-Cycle.

## CLARIFICATION

It is misleading to say:

> “A Store has a Cycle.”

Cycles belong to the Retailer's trading calendar.

Conceptually:

```text
RETAILER
   │
   ├── STORE ESTATE
   │
   └── TRADING CALENDAR
          │
          ├── Cycle 1
          ├── Cycle 2
          ├── Cycle 3
          └── ...
```

Stores participate in bookings made against those common trading Cycles.

---

# 12. Formats, physical units and physical capacity

## BRIEF

A Store carries a set number of units of a Format.

Example:

```text
Store A
Format: Aisle Barrier
Physical units: 6
```

Each unit is independently bookable.

That means, for example, those six units could potentially support several Advertisers simultaneously during the same Cycle.

## IMPORTANT DISTINCTION

There are two related but different concepts:

### Physical capacity

The physical Store/Format fact:

```text
Store × Format
        │
        ▼
physical capacity = 6
```

### Cycle occupancy

Which of that physical capacity is committed during a particular trading Cycle:

```text
Store × Format × Cycle
           │
           ▼
held / confirmed / potentially available
```

Therefore:

> **Physical capacity is naturally a Store × Format fact, while booking availability is evaluated in the context of Store × Format × Cycle.**

This distinction avoids implying that the Store physically acquires another six units every Cycle.

---

# 13. Are individual physical units identifiable?

## Question raised during interpretation

The brief says each unit is independently bookable.

Does that require modelling specific physical objects such as:

```text
Aisle Barrier #1
Aisle Barrier #2
Aisle Barrier #3
...
```

and perhaps their exact physical position inside the Store?

## BRIEF

The task establishes only that units are independently bookable.

It does **not** say whether:

- different units of the same Format have different commercial values;
- Advertisers select particular positions;
- units have individual addresses;
- installation teams require unit-level identity within this exercise.

## ASSUMPTION

For the baseline two-hour model:

> **Units of the same Format within a Store are treated as fungible capacity unless unit identity becomes necessary to express a requirement.**

So six independently bookable aisle barriers may initially be represented as:

```text
capacity = 6
```

rather than six separately named physical records.

## IMPORTANT QUALIFICATION

This assumption does **not** change the brief's requirement that multiple units may be independently consumed.

A quantity-based implementation must still support, for example:

```text
capacity = 6

Advertiser A consumes 2
Advertiser B consumes 3
Advertiser C consumes 1
```

The simplification concerns **unit identity**, not whether quantities are independently bookable.

## WHERE THE ASSUMPTION WOULD BREAK

Individual units would need explicit identity if, for example:

- one physical position were more valuable than another;
- units differed physically;
- an Advertiser selected a specific location;
- installation logistics required exact unit assignment.

Installation logistics are explicitly outside the current exercise, so this additional detail is not required unless deliberately chosen.

---

# 14. Availability

## BRIEF

The brief explicitly says:

> availability is not a simple lookup.

Whether inventory can be booked depends on:

- how many units physically exist;
- what has already been Confirmed;
- what is currently on Hold.

It explicitly says:

> there is no single field to read.

## DERIVED

Availability should therefore be understood as **derived state**, not as a standalone business fact.

Under normal, non-oversold conditions:

```text
Physical capacity
      -
Confirmed quantity
      -
Active Hold quantity
      =
Current normal availability
```

Example:

```text
Physical capacity      6
Confirmed              2
Active Holds           3
                     ───
Normal availability    1
```

A system may cache or display availability for performance or usability.

That cached/displayed value must not be confused with the underlying facts from which authoritative availability is determined.

---

# 15. Hold

## BRIEF

A Trader can hold inventory provisionally while the Advertiser decides.

Holds:

- are temporary;
- expire;
- are intended to block the space.

## DERIVED

An active Hold therefore consumes normal availability even though the booking is not yet Confirmed.

Example:

```text
Physical capacity      6
Confirmed              2
Active Holds           3
                     ───
Available              1
```

The fact that a Hold is provisional does **not** mean another normal booking may ignore it.

---

# 16. Confirmation and partial outcomes

## BRIEF

The brief states:

> a booking may be confirmed for some Stores and rejected for others.

Therefore the Campaign-level operation is not necessarily all-or-nothing.

## DERIVED

The model must be capable of representing partial outcomes at an appropriate level of granularity.

For example:

```text
Requested

Store A      4 units
Store B      4 units
Store C      2 units


Possible result

Store A      4 secured
Store B      not fully secured
Store C      2 secured
```

Exactly how the UI represents, retries or resolves that partial result remains a design decision.

The important requirement is that the domain must not assume:

> one Campaign request = one indivisible success/failure result.

---

# 17. Hold expiry

## BRIEF

Holds expire.

The brief does not specify:

- Hold duration;
- how expiration is configured;
- whether Holds may be extended;
- whether different Advertisers or Formats receive different durations;
- whether expiry is processed synchronously or by a background process.

## IMPORTANT TIME DISTINCTION

The brief also says:

> Cycles are the only unit of time.

In context, this constrains **when advertising runs**: media does not start or end mid-Cycle.

However, the same brief says Campaigns are assembled over days or weeks and Holds expire.

Therefore the booking workflow necessarily has some notion of elapsed operational time even though media scheduling itself is Cycle-based.

## ASSUMPTION

For this exercise:

> **Advertising occupancy is Cycle-based, while Hold lifetime may use ordinary workflow time.**

A simple implementation might record an expiry deadline when a Hold is created.

For example:

```text
Hold created
     │
     ├── booking applies to Cycle 12
     │
     └── Hold expires at <workflow deadline>
```

The exact duration and expiry mechanism remain design choices.

---

# 18. Expiry worker versus expiry rule

## Question raised during interpretation

Does a background worker make a Hold expire and return inventory to availability?

## ASSUMPTION / DESIGN DIRECTION

A robust interpretation is:

> **The expiry rule determines whether the Hold is still active; a worker need not be the authority that makes inventory available.**

For example, if a Hold records an expiry deadline and that deadline has passed, an availability calculation can treat the Hold as inactive even if housekeeping has not yet updated its stored status.

A worker may still be useful for:

- changing `HELD` to `EXPIRED`;
- notifications;
- audit processing;
- housekeeping.

This is a proposed design approach, **not a requirement supplied by the brief**.

The final technical submission may choose a different mechanism if justified.

---

# 19. Who is actually competing?

## Question raised during interpretation

Common sense suggests that Advertisers compete for scarce advertising inventory.

The brief, however, says that:

> twenty to thirty Traders may work the same few Cycles and the same space is genuinely contested.

It also explicitly suggests examining:

> what happens when Traders compete for the same space.

Are those contradictory?

## CLARIFICATION

No.

They describe two perspectives on the same contention.

### Commercial perspective

Different Advertiser Campaigns want the same scarce physical inventory.

### System perspective

Different Traders are the human actors concurrently attempting to secure that inventory.

Conceptually:

```text
Advertiser A ← Trader A ──┐
                           │
                           ├── same Store / Format / Cycle
                           │
Advertiser B ← Trader B ──┘
```

Therefore:

> **Advertisers represent competing commercial demand; Traders are the concurrent system actors through whom that demand reaches the booking system.**

This reconciles the terminology without treating Advertiser and Trader as the same actor.

---

# 20. Availability observation versus securing inventory

## DERIVED

Because multiple Traders can act concurrently, merely observing availability cannot guarantee that the same inventory will still be available when a booking attempt occurs.

For example:

```text
One unit remains

Trader A                  Trader B
   │                          │
sees 1                     sees 1
available                  available
   │                          │
   └──── attempts ────────────┘
          same unit
```

Therefore there is an important conceptual distinction between:

```text
QUERY
"What currently appears available?"
```

and:

```text
COMMAND
"Attempt to secure this inventory."
```

The second operation must ultimately establish what was actually secured.

## DESIGN IMPLICATION

The eventual design needs an authoritative correctness boundary capable of resolving simultaneous attempts.

The brief deliberately leaves the implementation mechanism open.

Possible approaches belong in the technical solution, not in this clarification document.

---

# 21. “Exception” and overselling

This remains the most important unresolved business ambiguity.

## BRIEF

The brief states:

- Holds are intended to block space;
- Traders nevertheless “push hard for exceptions” during peak periods;
- commercial teams sometimes want to accept the risk of overselling.

It does not define the word **exception** more precisely.

## POSSIBLE INTERPRETATIONS

An exception could potentially involve:

1. allowing a booking despite insufficient normal availability;
2. overriding or displacing an existing Hold;
3. extending or changing a normal Hold rule;
4. explicitly authorising sales beyond physical capacity;
5. some other commercial rule not described in the task.

## WORKING ASSUMPTION

For this exercise, unless later clarified:

> **The exceptional path will be modelled primarily as an explicit commercial authorisation to permit booking exposure above normal physical capacity.**

This interpretation was chosen because the brief immediately links peak-period exceptions with the willingness to accept **overselling risk**.

It is still an assumption, not a fact stated by the brief.

---

# 22. Physical capacity must remain physical truth

## DESIGN CONSEQUENCE OF THE WORKING ASSUMPTION

Suppose a Store physically has:

```text
6 aisle barriers
```

and commercial teams deliberately authorise exposure equivalent to one additional booking.

The system should preserve:

```text
Physical capacity = 6
```

rather than rewriting physical capacity to seven.

Conceptually, two facts exist:

```text
Physical capacity             6
Exceptional authorised risk  +1
```

This preserves the distinction between:

> what physically exists

and:

> what the business has deliberately permitted itself to sell.

The exact data representation will be decided in the technical solution.

---

# 23. Does an oversell authorisation apply to Holds, Confirmations, or both?

## AMBIGUITY

The brief does not say.

It would therefore be too strong to assert a universal invariant such as:

```text
Active Holds + Confirmed
<=
Physical Capacity + Oversell Allowance
```

without first defining how exceptional capacity may be consumed.

Possible policies include:

- exceptional capacity can itself be Held;
- exceptional capacity can only be used during Confirmation;
- an exception authorises a specific Campaign rather than increasing a general capacity pool.

These have different semantics.

## ASSUMPTION BOUNDARY

The appendix therefore establishes only this:

> **Normal Holds consume normal availability, and any departure from normal physical capacity must occur through an explicit authorised exceptional path.**

The exact consumption semantics of that exceptional path are left for the solution to define and justify.

---

# 24. Who requests and who authorises an exception?

## BRIEF

The wording distinguishes:

> Traders push hard for exceptions

from:

> commercial teams sometimes want to accept the risk of overselling.

## DERIVED

This suggests that **requesting** exceptional treatment and **authorising** the associated commercial risk may be different actions.

## AMBIGUITY

The brief does not define:

- the approving role;
- the permission hierarchy;
- whether every Trader can authorise overselling;
- whether approval occurs inside or outside the system.

## ASSUMPTION

The solution should therefore model:

> **an identifiable human-authorised exceptional decision**

without inventing an exact organisational hierarchy.

The eventual system may require role-based permissions, but specific roles are not part of the supplied requirement.

---

# 25. What happens when a Hold expires?

## Question raised during interpretation

Should released inventory automatically go to:

- the highest-spending Advertiser;
- a waiting Campaign;
- a priority Advertiser;
- the next request?

## BRIEF

No such policy is specified.

The task deliberately excludes pricing.

It contains no requirement for:

- auctions;
- waiting lists;
- advertiser ranking;
- spend-based priority;
- automatic reassignment.

## ASSUMPTION

For this exercise:

> **Expired inventory simply returns to the normally available inventory pool.**

No automatic transfer to another Advertiser is assumed.

## DESIGN ASSUMPTION IF SIMULTANEOUS REQUESTS OCCUR

If several Traders attempt to acquire newly available inventory concurrently, the simplest baseline is that the authoritative contention mechanism determines which request succeeds.

This is sometimes informally described as “first successful acquisition wins.”

That is a proposed baseline policy, not something required by the brief.

A business-priority mechanism could be added later if such a rule were supplied.

---

# 26. Normal inventory invariant

## DERIVED

Before introducing any exceptional commercial policy, the brief implies a simple normal-capacity rule.

For a particular:

```text
Store × Format × Cycle
```

the quantity consumed by:

```text
Confirmed bookings
+
Active Holds
```

must not exceed the physical capacity available for that Store and Format.

Conceptually:

```text
Confirmed + Active Holds ≤ Physical Capacity
```

under the **normal booking path**.

If the system exceeds this accidentally because two operations race, that is a correctness failure.

If it exceeds physical capacity through the explicitly permitted commercial exception mechanism, that is intentional business risk and must be distinguishable from the failure case.

---

# 27. Auditability implied by the scenario

## BRIEF

The brief emphasises that booking state ultimately affects:

- what field teams install;
- what finance invoices;
- whether someone pays for media that physically ran.

It also explicitly asks candidates to consider:

- failure modes;
- data integrity;
- what happens when system state and physical reality disagree;
- how the system is operated rather than merely built.

## DESIGN IMPLICATION

Important state changes and exceptional decisions should therefore be explainable after the fact.

At minimum, a fuller design is likely to need to answer questions such as:

- who placed this Hold?
- when was it placed?
- when did it expire?
- who Confirmed this booking?
- what capacity was believed to exist?
- was an exception used?
- who authorised that exception?
- why did system bookings exceed physical capacity?

Exactly how auditability is implemented remains part of the technical design.

---

# 28. Conceptual domain picture

The problem can now be summarised as follows.

## Physical estate

```text
                         RETAILER
                            │
              ┌─────────────┴─────────────┐
              │                           │
         STORE ESTATE               TRADING CALENDAR
              │                           │
           STORES                       CYCLES
              │
           FORMATS
              │
        PHYSICAL CAPACITY
```

Physical capacity primarily answers:

> How many units of Format F physically exist in Store S?

---

## Booking context

```text
Store
  ×
Format
  ×
Cycle
  │
  ▼
BOOKABLE INVENTORY CONTEXT
  │
  ├── Active Holds
  ├── Confirmed bookings
  └── Exceptional commercial decisions
```

This context answers:

> What can still be secured for this Format in this Store during this Cycle?

---

## Commercial actors

```text
ADVERTISER
    │
    │ Campaign requirement / decisions
    ▼
TRADER
    │
    │ operates booking workflow
    ▼
BOOKING SYSTEM
    │
    ▼
RETAILER INVENTORY
```

---

## Contention

```text
Advertiser A ← Trader A ─┐
                          │
                          ├── same finite inventory
                          │
Advertiser B ← Trader B ─┘
```

The Advertisers represent separate commercial demand.

The Traders are the concurrent system actors.

---

# 29. Explicit assumptions adopted for the exercise

The following assumptions are intentionally adopted so that design work can proceed without silently inventing unspecified business rules.

| Topic | Working assumption | Status / reason |
|---|---|---|
| Trader | Human/operator assembling Campaigns in response to Advertiser demand | Derived from one Retailer vs 20–30 Traders and the Campaign wording |
| Trader employer | Unspecified | Genuine ambiguity; irrelevant to core inventory correctness |
| Advertiser | Commercial party whose advertising requirement is being fulfilled | Derived; exact contractual/payment relationship unspecified |
| Advertiser self-service | Not modelled | Scope assumption; brief specifically describes Trader interaction |
| Campaign ownership | Not asserted | Brief establishes association and Trader assembly, not formal ownership |
| Media time | Bookings run only in fixed consecutive Cycles | Explicit brief requirement |
| Hold time | Hold expiry may use workflow clock time | Necessary assumption to reconcile expiry with Cycle-based media scheduling |
| Physical capacity | Primarily Store × Format | Derived from physical units described by the brief |
| Booking context | Store × Format × Cycle | Derived from bookings being Cycle-specific |
| Unit identity | Same-Format units initially treated as fungible quantity | Simplification; independent quantity booking still preserved |
| Availability | Derived from physical capacity, Confirmed bookings and active Holds | Explicitly implied by brief |
| Hold | Blocks normal inventory while active | Explicit brief requirement |
| Partial result | Supported | Explicitly required across Stores |
| Expiry policy | Unspecified; use a simple baseline in the design | Brief supplies no duration/policy |
| Expired Hold | No longer blocks inventory | Natural consequence of “Holds expire” |
| Expired capacity | Returns to ordinary availability | Assumption; no reassignment policy supplied |
| Priority after expiry | None assumed | No waiting-list, auction or priority rule supplied |
| Exception meaning | Working model is authorised departure above normal availability/capacity | Assumption chosen because brief links exceptions to overselling |
| Hold displacement | Not assumed | Brief does not explicitly say active Holds may be stolen/overridden |
| Oversell authority | Human/commercially authorised, exact role unspecified | Brief distinguishes Traders pushing from commercial teams accepting risk |
| Exceptional capacity semantics | To be defined in solution | Brief does not say whether Holds, Confirmations or particular Campaigns consume it |

---

# 30. Ambiguities deliberately left visible

The following should **not** silently turn into invented requirements.

## A. Organisational identity of Trader

Unknown.

**Effect on current problem:** low.

The system needs to identify and authorise the actor; it does not need to know who employs them to solve inventory contention.

---

## B. Exact meaning of “exception”

Unknown.

The working solution will interpret it principally as an explicitly authorised oversell path because that best connects the surrounding sentences.

**Effect:** significant enough to state prominently.

A different business definition could change the exception workflow while leaving normal capacity correctness largely unchanged.

---

## C. Oversell approval authority

Unknown.

**Effect:** low on basic inventory modelling; important for eventual permissions and audit.

---

## D. Whether exceptional capacity may itself be Held

Unknown.

**Effect:** relevant to the exceptional-state model.

This should be selected and justified rather than accidentally assumed.

---

## E. Hold duration and extension rules

Unknown.

**Effect:** low on the core model if Hold expiry is represented explicitly.

---

## F. Individual media-unit identity

Unknown beyond each unit being independently bookable.

**Effect:** potentially substantial in a fuller fulfilment/installation system; deliberately simplified here.

---

## G. Detailed partial-success workflow

The brief permits partial confirmation but does not define what the Trader must do after receiving such a result.

**Effect:** important to Trader UX, but not to the basic definition of finite inventory.

---

# 31. Explicit scope inherited from the brief

The following are supplied assumptions and should **not** be redesigned as part of this exercise:

- Store estate is fixed for the modelled period.
- There is one Retailer.
- There is one Retailer trading calendar.
- Campaign media runs only in fixed four-week Cycles.
- Campaigns do not start or end mid-Cycle.
- Pricing is out of scope.
- Invoicing mechanics are out of scope.
- Installation logistics are out of scope.
- A Confirmed booking is not amended.

It is acceptable to note where one of these assumptions would eventually break the design, but solving those future problems is not required.

---

# 32. Things this interpretation deliberately does not invent

Unless the eventual solution explicitly chooses one as a justified extension, the brief does not require:

- an Advertiser self-service booking application;
- duplicate Advertiser and Trader booking workflows;
- Advertiser ranking;
- spend-based priority;
- auctions;
- waiting queues;
- automatic allocation when a Hold expires;
- specific physical addresses for individual media units;
- dynamic Store-estate management;
- multiple Retailers;
- arbitrary campaign dates;
- mid-Cycle changes;
- pricing models;
- installation planning;
- complex policy engines;
- a particular database technology;
- microservices;
- event sourcing;
- message queues;
- any particular cloud architecture.

These may be legitimate implementation choices or future requirements, but they should not be mistaken for problems the brief has asked us to solve.

---

# 33. The resulting technical problem

After separating stated requirements from assumptions and ambiguities, the task can be expressed more precisely:

> **A Trader uses the system to assemble advertising activity associated with an Advertiser. The activity consumes finite physical media capacity supplied by a single Retailer across its Store estate. Physical capacity exists by Store and Format; booking occupancy applies to specific four-week Cycles. Multiple Traders may concurrently attempt to secure the same capacity. Active Holds provisionally block capacity and later expire; Confirmed bookings consume capacity for the relevant Cycle; outcomes may be partial across Stores. Availability must therefore be derived from current inventory commitments rather than read from a single stored value. Normal concurrency must not accidentally allocate more than the physical capacity. At the same time, the business may deliberately accept overselling risk through an exceptional commercial decision whose exact semantics are not fully specified by the brief. Because booking data drives physical installation and invoicing downstream, accidental divergence between system state and physical reality is a material correctness failure.**

That is the core problem.

---

# 34. Questions the technical solution now needs to answer

With the interpretation frozen, the design exercise can concentrate on:

### Inventory

1. How should Stores, Formats, physical capacity and Cycles be represented?
2. Is quantity-based fungible capacity sufficient for the baseline?
3. How is current availability derived?

### Lifecycle

4. What are the meaningful states/transitions between requested, Held, expired/released and Confirmed inventory?
5. How are partial outcomes represented?

### Contention

6. What happens when multiple Traders see the same remaining availability and attempt to secure it concurrently?
7. Where is correctness ultimately enforced?
8. How are stale availability observations handled?

### Holds

9. How is Hold expiry represented?
10. What happens operationally when a Hold expires?
11. How does released capacity become available without introducing incorrect allocation?

### Exceptions / oversell

12. What exact exceptional model will the solution choose?
13. How will an authorised oversell remain distinct from physical capacity?
14. Who or what records the human authorisation?
15. Can exceptional capacity be Held, Confirmed only, or attached to a particular Campaign?

### Trader experience

16. What does the Trader see when availability changes between viewing and securing?
17. How are conflicts and partial results communicated without pretending the UI owns inventory truth?

### Operations

18. How can the system explain important allocation decisions afterwards?
19. Which invariants and metrics demonstrate that the system is behaving correctly?
20. How would operators detect accidental divergence between system commitments and physical reality?

---

# 35. Final working mental model

The shortest useful representation is:

```text
                         RETAILER
                     physical supplier
                           │
              ┌────────────┴────────────┐
              │                         │
           STORES                 TRADING CYCLES
              │
           FORMATS
              │
      PHYSICAL CAPACITY
              │
              └────────────┬────────────┘
                           ▼
                STORE × FORMAT × CYCLE
                           │
               ┌───────────┴───────────┐
               │                       │
          ACTIVE HOLDS             CONFIRMED
               │                       │
               └───────────┬───────────┘
                           ▼
                 DERIVED AVAILABILITY
                           │
                  subject to explicit
                  commercial exception


ADVERTISER A ← TRADER A ───┐
                            ├── contests this inventory
ADVERTISER B ← TRADER B ───┘
```

The exercise is therefore not primarily:

> “How do we build a calendar?”

and not primarily:

> “How do we build an advertising UI?”

It is:

> **How do we model and operate contested physical media inventory correctly while supporting the provisional, human and commercially exceptional nature of real booking workflows?**

That interpretation is sufficiently stable to begin the technical solution. Remaining uncertainties should stay visible as assumptions or explicit design choices rather than prompting further reconstruction of business terminology unless they materially block the design.