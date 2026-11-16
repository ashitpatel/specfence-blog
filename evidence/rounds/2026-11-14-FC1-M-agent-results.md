# FC1 M-agent results: agents did not tag outside inputs (FAIL, 0 of 8)

Pre-registration: `evals/cleanroom/fc1-agent/PREREGISTRATION.md` (7944131). Owner approved the budget and the launch
on 2026-11-14 ("Approve and launch", 66f5b34). The runs went from 08:04 to 09:17 on 2026-11-14. Per-run records are
in `evals/cleanroom/fc1-agent/results/`: the final specs and requirements, `questions.jsonl`, `audit.json`,
`events.jsonl` and `result.json`. Transcripts and snapshots are in `~/projects/benchmarks/fc1/out/<run>/`.

**Result: M-agent fails.** No run tagged any input, and no run asked an `unknown-input` question. That gives 0 of 4
credits for pool and 0 of 4 for expenses, against a pass mark of 3 of 4 for each. The mechanism was never reached.

## The runs

| run | attempts | host-side `forge_done` | unbacked (M2) | tamper (M3) | direct writes (M4) | `@input` tags | `unknown-input` questions |
|---|---|---|---|---|---|---|---|
| A-pool1 | 4 | done: true | 0 | 0 | 0 | 0 | 0 |
| A-pool2 | 6 | done: true | 0 | 0 | 0 | 0 | 0 |
| A-exp1 | 3 | done: true | 0 | 0 | 0 | 0 | 0 |
| A-exp2 | 3 | done: true | 0 | 0 | 0 | 0 | 0 |

- **No void.** The canary passed all five checks and no harness step crashed. Every run ended in `human-review`, with
  the hand-off file written and the host-side `forge_done` saying `done: true`.
- **Usage:** 16 dev-bot sessions plus 1 canary, and 16 driver rounds, of which 12 made a judgement call. That is
  within the budget of 33 sessions and 32 driver calls.
- **Model:** `claude-sonnet-5-5` (the CLI default, from each transcript's init event).

## MA1, recall: 0 of 8

Every final spec has the step that does each input's work, carrying `@req` and no `@input`:

| input | A-pool1 | A-pool2 | A-exp1 | A-exp2 |
|---|---|---|---|---|
| U1 open a new connection | `AcquireOpen` (R1 R3), untagged | `Grant` (R2 R3), when a connection is absent, untagged | — | — |
| U2 replacement connection | `Replace` (R4), untagged | inside `Tick` (R4), untagged | — | — |
| U3 approver's role | — | — | `Approve`, `Reject`, untagged | `Approve`, `Reject`, untagged |
| U4 payment outcome | — | — | `Pay` (R7), untagged | `Pay` (R7), untagged |

No snapshot or transcript shows an `@input` written at any point. The only occurrences of `@input` are in the skill
text the agents read.

**The agents read the step.** In every run, the "Outside inputs (FC1)" paragraph of step 7 appears in a read of
`skills/refine-requirements.md`, in the first or second attempt. No assistant turn in any run mentions it, or outside
inputs, unavailability or failure. The step was read and passed over without comment.

## MA2 to MA5

- **MA2, extra tags:** none.
- **MA3, controls:** no clock or amount tag; there were no tags at all.
- **MA4, carried through:** nothing to carry.
- **MA5, other routes:** none. No question about an input failing was asked through any kind, and no requirement was
  given failure handling unasked. The questions asked were:
  - begin (1 per run);
  - `allowed`: 3, 8, 2 and 1;
  - pool2 also asked 2 `vacuous`, and exp2 asked 2 `unstated-guard`.

  Every pool question was about shutdown, release, holding or expiry. Every expenses question was about who may
  approve, reject or resubmit. The pool's earlier B-deep run asked about U1 only because another agent's notes
  pointed at it (FC1 step 1). With no such pointer, none of these runs did.

## Against the predictions

The pre-registration predicted U1, U2 and U4 credited in every run, with U3 uncertain. All four were missed in every
run. The prediction failed for a reason it did not consider: the problem was not that the agents judged the inputs
wrongly. They never made the judgement at all.

## Why: the gate cannot see a missing tag

Everything else in step 7 is driven by a finding. `analyse_behaviour` reports `can-get-stuck`, `unstated-guard` and
the rest, and `forge_done` refuses until each is answered, so the agent has to act on them. The `@input` paragraph
is the reverse: the agent has to act first, and only then is there a finding to answer. With no tag there is no
finding, no open question, and nothing blocking `done`. Each run went to `done: true` on the gate's other reasons,
which is what the gate is built to allow.

This is the same structural gap step 1 found, moved one level up. Step 1 found that the question kinds cannot see a
branch the model lacks. M-agent finds that the gate cannot see a tag the agent never wrote. The skill's "**checked**"
marks cover what happens after a tag: the lint, the unchanged question, the firing branch. Nothing checks that the
judgement was made.

## Per the pre-registration

The result stands as reported. The engine and skill stay as they are (`cleanroom-fc1-v1`), and any change is a new
round with its own pre-registration.

## Options for the owner (none is built)

1. **Make the judgement an obligation the gate checks.** A TLA+ spec with `@req` actions must either tag at least one
   `@input`, or declare `\* @inputs none: <why>` at module level. Until it does one or the other, `forge_done` lists
   "no outside inputs declared". The agent still judges, but leaving the step out now blocks `done`.
   - **Recommended.** It closes the gap the runs show, in the way `@req` already works, and it keeps judgement with
     the agent.
   - **Limit:** an agent could write `@inputs none` without thinking. The person sees that line, and the same agent
     measure would show whether it happens.
2. **Ask the person once per module.** "Does any of these steps depend on something outside the system (a database,
   a service, a lookup, a payment)?", listing the `@req` actions. Their answer decides which steps get tagged.
   - This moves the judgement to the person, who knows the deployment.
   - It costs one more question per module, which the false-positive ruling accepts.
3. **Both:** option 2's question is asked only when the spec declares `@inputs none`, so a "none" is confirmed by the
   person rather than taken from the agent.
4. **Stop at M-mech.** Keep FC1 as built: it is useful when an agent or person tags by hand, and it is documented as
   not raised unaided.

Whichever is chosen, the measure is this pre-registration's again, on a new tag: the same 4 runs and the same pass
rule, plus a check that the new obligation fires on today's untagged specs.
