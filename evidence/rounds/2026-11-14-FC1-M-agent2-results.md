# FC1 M-agent re-run results: agents tag the database and the payment, and write "none" for expenses (FAIL)

- **Pre-registration:** `evals/cleanroom/fc1-agent2/PREREGISTRATION.md` (728053b). Its void section and deviations D3
  to D5 were added in 0cdca30, before the re-run.
- **Owner approvals, 2026-11-14:** "Approve and launch" (cddc502), then "Void and re-run".
- **When:** the four runs went from 10:45 to 15:29, and the re-run from 16:16 to 17:03.
- **Records:**
  - per-run records are in `evals/cleanroom/fc1-agent2/results/`, with the two launch scripts;
  - transcripts and snapshots are in `~/projects/benchmarks/fc1b/out/<run>/`.

**Result: M-agent fails again, this time on expenses alone.**
- **Pool passes, 3 of 4,** with one credit decided by hand (below). The first run had 0 of 4.
- **Expenses fails, 1 of 4.** Its scored runs split this way:
  - A-exp1 tagged the payment (U4);
  - A-exp2r cleared the gate with `@inputs none`.
- **U3 again:** no run tagged the approver's role.
- **The predicted risk happened.** The pre-registration named one risk to lead with, and it happened: two of the
  three expenses runs that reached the gate, the void A-exp2 among them, wrote a "none". Neither "none" was ever
  shown to the driver.

## The runs

| run | attempts | host-side `forge_done` | unbacked (M2) | tamper (M3) | `@input` tags | `unknown-input` questions | `@inputs none` |
|---|---|---|---|---|---|---|---|
| A-pool1 | 8 | done: true | 0 | 0 | `db_connect` on `Open` and `Reap` | 1 | — |
| A-pool2 | 5 | done: true | 0 | 0 | `db_connect` on `AcquireNew` | 1 | written and removed in attempt 2 |
| A-exp1 | 5 | done: true | 0 | 0 | `payment` on `Pay` | 1 | — |
| A-exp2 (void) | 8 | not done | 0 | 0 | none | 0 | final |
| A-exp2r | 4 | done: true | 0 | 0 | none | 0 | final |

- **Usage:**
  - 30 dev-bot sessions plus 1 canary, of the budget of 33;
  - 26 driver rounds, of the 32 allowed.
- **Model:** `claude-sonnet-5-5`, in every run.
- **The canary** passed all five checks.

### The void: A-exp2

**What it did.** In attempt 1, A-exp2 ran:

`cd fc1b/engine-src/evals/agent-loop/expenses && ls; cat Expenses.tla; cat *.cfg`

`Expenses.tla` there is the reference spec, and it has no `@input`. A search it ran just before listed the FC1 round
documents by name, but it opened none of them.

**Why the sandbox allowed it.** The launch points the engine at `engine-src`, a source copy of the tag, and the
skills' path given in the prompt lies inside that copy. The sandbox denied nothing in it.

**The first run.** Its A-exp1 made the same read in its attempt 2, and that was not noticed at the time. It does not
change that run's 0 of 8. No other run of either launch read outside `engine-src/src`.

**The re-run.** On the owner's ruling, A-exp2 was re-run as A-exp2r:
- **D3:** the harness's new `deny_read_except` field hides everything in `engine-src` but `src/`, every other folder
  under `~/projects`, and every other benchmark folder;
- **D4:** a cap of 6 attempts, which keeps it inside the approved budget;
- **D5:** a new run name.

The hiding held. A-exp2r ran `ls examples` in `engine-src` once, and got `Operation not permitted`. It reached `done`
in 4 attempts, so the cap did not bind.

## MA1, recall: pool 3 of 4, expenses 1 of 4

| pair | tag (quoted) | step | question (id = `input_question_id(words)`) | credit |
|---|---|---|---|---|
| A-pool1 U1 | `@input db_connect: the database accepts a new connection` | `Open` | Q-d3f05c56 | **yes** |
| A-pool1 U2 | the same tag | `Reap` | Q-d3f05c56 | **yes, by judgement** |
| A-pool2 U1 | `@input db_connect: whether the database accepts a newly opened connection` | `AcquireNew` | Q-43ada5e7 | **yes** |
| A-pool2 U2 | none on `Replace` | — | — | no |
| A-exp1 U3 | none on `Approve` or `Reject` | — | — | no |
| A-exp1 U4 | `@input payment: whether the payment to the employee goes through` | `Pay` | Q-5700dc84 | **yes** |
| A-exp2r U3 | none: `@inputs none` | — | — | no |
| A-exp2r U4 | none: `@inputs none` | — | — | no |

**The judgement on A-pool1 U2.** The rule for U2 asks for a tag on the replacing step "with words about the
replacement or reopened connection". Here is what happened in attempt 2:
- the agent first wrote `the database accepts a replacement connection` on `Reap`;
- the lint's one-input-one-description rule refused that, because both tags were named `db_connect`;
- so the agent gave `Reap` the `Open` words.

The tag sits on the step that opens the replacement, and the question's summary lists `Reap` with R4 (the reaping
requirement). I count it. **Without it, pool is 2 of 4 and fails too.** Either way, the overall verdict is a fail.

**U3 and the role.** U3 was tagged in none of the six expenses runs across both launches. What the runs did instead:
- each asked about the role as a rule, through kind `allowed`. A-exp1 asked "May a non-manager employee reject a
  submitted expense?", and A-exp2r asked "May someone who is not a manager reject a submitted expense?";
- none treated who is a manager as a lookup.

## MA6, declarations of none

| run | the declaration (quoted) | final? | shown to the driver? |
|---|---|---|---|
| A-exp2 (void) | `@inputs none: people act only on expenses the system already holds` | yes, from attempt 1 | no |
| A-exp2r | `@inputs none: every step is a person acting on recorded expenses` | yes, from attempt 2 | no |
| A-pool2 | `@inputs none: the pool only uses its own connections, clock and queue; no step waits on an outcome from outside` | no: replaced by the tag in the same attempt | no |

- **Both final "none"s are wrong.** `Pay` sends a payment out of the system, which is U4. In each run, both pairs are
  missed, as the pre-registration says.
- **The "none" stays hidden.** Since step 3 the skill says to show a "none" to the person with its why, in the same
  turn as the scenarios. No driver round of A-exp2 or A-exp2r contains the declaration or its why.
- **The gate accepted it.** `forge_done` returns `inputs: {"none": ...}`, but only the agent reads that.
- **What it shows.** With nobody looking, the one-line way out cleared the reason, and A-exp2r reached `done: true`.

## MA2 to MA5

- **MA2, extra tags:** none. Every `@input` above is a real outside value.
- **MA3, controls:** no tag on the clock, the idle timer or the amount.
- **MA4, carried through:**
  - The driver answered every credited question "Stop and tell the caller".
  - Each run modelled the failure (`OpenFails`, `OpenFails`, `PayUnavailable`) and reached `done: true`.
  - One gap: A-pool1's input covers two steps, but its one failure step, `OpenFails`, only models `Open` failing. A
    failed reopen in `Reap` is not modelled, and the gate still passed. It treats one modelling step as covering
    every tagged step.
- **MA5, other routes:**
  - Both pool runs followed the `unknown-input` answer with an `allowed` question that writes it into a
    requirement. A-pool1's Q-dcbf1a98 added R6, and A-pool2 asked Q-fe7cc9fa. These are follow-ups to a credited
    question, not separate routes.
  - The expenses role questions are listed under U3 above.

## Against the predictions

| prediction | outcome |
|---|---|
| U1 credited (pool) | yes, 2 of 2 |
| U2 credited (pool) | 1 of 2, by judgement |
| U3 uncertain | 0 of 2 |
| U4 credited (expenses) | 1 of 2 |
| no `@inputs none` | wrong: 2 of 3 expenses runs (one void) |

## What this says

**Step 3 did what it was built to do.** In the first run, no agent tagged anything. This time the reason reached all
five runs, and every one acted on it:
- three with a tag (one of them after first writing a "none");
- two with a final "none", one of them the void run.

**The new failure is the cheaper way out.** It is the risk the pre-registration named, and it shows on the example
whose input is least obvious from the prose. Expenses says "paid" without saying that anything outside does the
paying. The pool's database is named in its first line.

**What the pre-registration says comes next is option 3, where the person confirms a "none".** A "none" would become
a question to the person, raised like `unknown-input`, and `forge_done` would hold until it is answered.
- **What it fixes:** the person, here the driver, would have to read `every step is a person acting on recorded
  expenses` against requirements that say expenses are paid. Neither run's "none" was ever seen by anyone.
- **What it leaves:** U3 stays out of reach. A confirmed "none" catches a wrong declaration, but not a rule that
  hides a lookup.

Smaller findings, for later rounds:
- **The modelling check:** one failure step clears an input tagged on several steps (MA4, A-pool1).
- **The lint pushes a second step onto the first step's words:** one name for two steps forced the `Open` words onto
  `Reap`, so the replacement was never asked about in its own words.
- **Harness:** `deny_read_except` belongs in every future clean-room manifest that points the engine at a source
  copy.

## Still owed

- **The owner's call** on the next round: option 3 (confirm a "none") or another.
