# FC1 M-agent3 results: the confirmation reached the driver, and the driver confirmed both wrong "none"s

- **Pre-registration:** `evals/cleanroom/fc1-agent3/PREREGISTRATION.md` (36429e2).
- **Owner's budget approval:** 2026-11-14, "Approve and launch" (241a2ea).
- **Engine:** `cleanroom-fc1-v3` = e7c731b, which has FC1 step 4 and, as shipped, EG1.
- **Kept outputs:** `evals/cleanroom/fc1-agent3/results/`. For each run there are the final workspace,
  `questions.jsonl`, the audit, the result, the events and `outside-paths.txt`, plus the canary and the launch log.

## Result: FAIL

| example | credited | pass needs |
|---|---|---|
| pool | **4 of 4** (one by judgement, as in the re-run) | 3 |
| expenses | **0 of 4** | 3 |

Pool passes. Expenses does not, so M-agent fails.

**What happened in expenses.** Both expenses runs wrote `@inputs none`. Step 4 put each "none" to the driver
as an `inputs-confirm` question, and the driver answered `Yes, nothing outside` both times. That is the
pre-registered risk: the question reached the person, and the person did not see the miss.

## Runs

The launch ran from 23:04 to 03:58.

| run | attempts | final | void check |
|---|---|---|---|
| canary A | 1 | pass (K1, K2, W1, W2, W3) | — |
| A-pool1 | 6 | `done: true` | `outside-paths.txt` empty |
| A-pool2 | 6 | `done: true` | empty |
| A-exp1 | 5 | `done: true` | empty |
| A-exp2 | 3 | `done: true` | empty |

**No run is void.** No transcript line names a path the manifests hide.

**Usage:** 21 of 33 dev-bot sessions (20 attempts and the canary), and 16 of 32 driver calls.

## MA1: recall

### A-pool1

The spec tags one input, `db_connect: the database opening a replacement connection`, on two steps:
- `Reap`, where the idle connection is replaced;
- `OpenReplacement`, where the longest waiter opens an empty place.

Its question `Q-65e01703` was asked, and its id is `input_question_id` of those words.

| pair | step | credited |
|---|---|---|
| U1 | `OpenReplacement` | yes |
| U2 | `Reap` | yes |

### A-pool2

The spec tags `db_open: opening a new connection to the database succeeds` on two steps: `AcquireOpen` and `Reap`.
Its question `Q-2ef4237a` was asked, and the id matches.

| pair | step | credited |
|---|---|---|
| U1 | `AcquireOpen` | yes |
| U2 | `Reap` | **by judgement** |

The U2 credit follows the same reasoning as the re-run's A-pool1. The step is the replacement, and its words name
opening a connection to the database. A replacement is such an opening, but the words do not say "replacement".

### A-exp1 and A-exp2

Neither run tags an `@input` at any point. No `@input <name>:` line appears in any of their transcripts. So U3 and
U4 are not credited in either run: **0 of 4**.

## MA6: declarations of none

| run | final spec |
|---|---|
| A-exp1 | `@inputs none: every step is a person's action on the expense records` |
| A-exp2 | `@inputs none: people act on the expense itself, nothing looked up` |

Both are wrong. Each misses the payment (U4) and the approver's role (U3). A-exp2's why denies a lookup outright.

## MA7: the confirmations

| run | question (`inputs-confirm`) | driver's answer | what the agent did next |
|---|---|---|---|
| A-pool1 | `Q-96c04b31`: "…only these come from outside…: the database opening a replacement connection. Anything else?" | `Nothing else` | nothing; the claim stood |
| A-pool2 | `Q-166cc168`: "…only these come from outside…: opening a new connection to the database succeeds. Anything else?" | `Nothing else` | nothing; the claim stood |
| A-exp1 | `Q-2c8ffe40`: "…nothing here comes from outside…: \"every step is a person's action on the expense records\". Right?" | `Yes, nothing outside` | nothing; `done` |
| A-exp2 | `Q-00e45277`: "…nothing here comes from outside…: \"people act on the expense itself, nothing looked up\". Right?" | `Yes, nothing outside` | nothing; `done` |

- **The driver's reasons.** For A-exp1 its note gives the reason: "R1 to R9 describe only people acting on expense
  records, so nothing outside is involved". It answered A-exp2 in its first call, with no note on the question.
- **No credit came from a No,** since no confirmation was answered No.

The driver plays the owner from the settled requirements, and those say nothing about systems outside. Read that
way, the "none" is true of the requirements text, so the driver confirmed it. The engine did what step 4 built: it
asked the person, in the words planned, and held `done` until the person answered.

## MA2–MA5, MA8

- **MA2, extra tags:** none.
- **MA3, controls:** neither the clock, the idle timer nor the expense amount is tagged.
- **MA4, carried through:** both pool questions were answered `Stop and tell the caller`. Both specs model that with
  `@unknown` steps: A-pool1's `ReapFail` and `OpenFail`, A-pool2's `AcquireOpenFails`. All four runs reached
  host-side `done: true`.
- **MA5, other routes:** no question of another kind asks about an input failing.
- **MA8, EG1's prompts:** no run asked the fresh-readers question, because no `custom` question appears in any log.
  The agents read step 6b of the skill and skipped it, since it is not checked when it is never asked. So no
  readings were made, and EG1 did not affect this run. Step 6b not being asked is a gap in EG1's own gate. It is
  reported here for that work, and is not FC1's.

## Against the predictions

| prediction | outcome |
|---|---|
| pool: U1 credited, U2 uncertain | U1 2/2; U2 2/2, one by judgement |
| expenses: U3 uncertain, U4 credited | U3 0/2, U4 0/2 |
| no "none" in a final spec | wrong: both expenses runs wrote one |
| a "none" is asked, the driver answers No | asked yes; answered **Yes** |

## Across the three rounds

| round | engine | pool | expenses | how expenses was missed |
|---|---|---|---|---|
| M-agent | step 2 | 0/4 | 0/4 | no tags at all |
| re-run | step 3 | 3/4 | 1/4 | one wrong "none", unseen; U3 never tagged |
| M-agent3 | step 4 | 4/4 | 0/4 | two wrong "none"s, both confirmed by the driver |

Pool now works. On expenses, each gate has been passed by the cheapest true-sounding claim. The last gate is a person
reading a "none" against requirements that do not mention outside systems.

**The driver is a stand-in.** An owner who knows that approvers' roles live in a directory, or that Finance pays
through a bank, might answer No. The settled requirements do not say so, and the driver answers from them only.
Whether a real person catches it is not what this run measured.

## What the next round could be (the owner's call)

1. **Put the steps in the question.** Name each step the "none" covers, and its words ("Pay: Finance pays approved
   expenses"), so the person reads the payment step itself rather than a summary sentence. This is cheap, but the
   driver may still read "pays" as a local act.
2. **Make the agent weigh each step before it may write a "none".** Require a short per-step line
   (`@local <step>: <why>`) for every `@req` step, so the payment step carries its own claim. A gate can check that
   every step has one, but not that each one is true.
3. **Change what the driver knows.** Give the driver an owner's note on deployment, such as "payments go through the
   bank's API; roles come from the HR directory". That tests whether a person with that knowledge catches the miss.
   It is a change to the measurement, not to the engine, and the earlier rounds would not be comparable.
4. **Stop here.** Record pool as working and expenses as what the method cannot see without an owner who knows the
   deployment.
