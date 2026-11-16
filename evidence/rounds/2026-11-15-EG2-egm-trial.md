# EG2: a trial of EGM's review on specfence's own skill text, against failures already known

**Question.** Is Google's [Elephant-Goldfish Model](https://github.com/google/elephant-goldfish-model) (EGM)
review worth using when specfence's skill docs change? The test uses text where agent trials have already found
the defects: would `/egm-review` have flagged them before those trials were run?

**Owner approval:** "Run the trial", 2026-11-15. This record was written and committed before any review ran.

## What is reviewed

The step-7 text of `src/specfence_engine/skills/refine-requirements.md`, exactly as each FC1 agent trial read it:

| version | tag | what the agent trial found |
|---|---|---|
| V1 | `cleanroom-fc1-v1` (48145c4) | **D1.** The outside-input paragraph asks the agent to tag inputs first, and nothing blocks `done` when it does not. All 8 runs read the paragraph and passed it over (`2026-11-14-FC1-M-agent-results.md`). |
| V2 | `cleanroom-fc1-v2` (a5c0241) | **D2.** `\* @inputs none: <why>` clears the gate, and the instruction to show it to the person is not checked. Two expenses runs wrote a wrong "none" that nobody saw. **D3.** A rule can hide a lookup (the approver's role), and a "paid" in the prose does not say that something outside does the paying. U3 was never tagged (`2026-11-14-FC1-M-agent2-results.md`). |
| V3 | `cleanroom-fc1-v3` (e7c731b) | Not known yet. The `fc1-agent3` trial will tell. The prediction below is written before its results. |

Each version's whole skill file is the review target, because the agents read the whole file. Nothing else is
given to the reviewers: no FC1 records, no plan, and no fixtures (`GoldfishContext` is empty).

## How it runs

- **The plugin:** EGM at e4747323 (2026-11-14), cloned into the session scratchpad. It is loaded for one session
  only with `claude -p --plugin-dir`, and is not installed anywhere.
- **The command:** `/egm:egm-review` with the target, an empty `GoldfishContext`, and `Step: uninterrupted`. That
  runs EGM's consolidated five-stage review: zoom out, comprehension, critique, zoom out again, and implementation
  readiness.
- **Isolation:** each run starts in a new, empty folder outside the repository, with only the target file in it.
  - That folder has no project `CLAUDE.md` and no auto-memory.
  - No MCP servers are loaded (`--strict-mcp-config`).
  - Reads under `~/projects` and `~/.claude/projects` are denied, and so are Bash and web tools.
  - After each run, its stream log is searched for any path outside the run folder and the plugin. A run that read
    FC1 material is void.
- **Size:** one run per version, three in all. Variance is not measured: this is a cheap first look, not an
  estimate of a rate.

## What counts as a catch

A catch is a finding in any EGM report that a reader of the report, who knows nothing of FC1, would act on in the
right direction. It is quoted in the results.

- **D1, at V1:** the finding says the outside-input tagging can be skipped, or depends on the agent acting with
  nothing to prompt it, or is not checked or enforced, or reads as optional.
- **D2, at V2:** the finding says the "none" declaration is an easy way out, or can be written without real
  justification, or that showing it to the person is not checked.
- **D3, at V1 or V2 (secondary):** the finding says "outside the system" is unclear for a lookup or a decision rule
  (who has a role), or for an outcome the prose states as done ("paid").

A finding that points at the right paragraph for an unrelated reason is not a catch. **Noise** is also recorded:
the number of findings in all, and how many of them are about the outside-input paragraph at all.

## Pass rule (fixed now)

- **Worth adopting** for skill-doc changes: D1 is caught at V1 **and** D2 is caught at V2.
- **Partial:** one of the two is caught. Then it would be adopted only as an optional step, with a repeat run first.
- **Not worth it:** neither is caught.
- D3 and V3 do not decide the verdict.

## Prediction for V3 (checked later against `fc1-agent3`)

After the run, this record will list each V3 finding about the outside-input text. When `fc1-agent3` reports, each
such finding is scored hit or miss against what the agents actually did.

## Limits, stated now

- The scorer (the session that wrote this) knows the answers. The quotes are given in full, so the owner can
  re-score.
- EGM is built to review design docs, not instructions for agents. A miss may say more about that fit than about
  EGM's quality.
- One run per version can only show that a catch is possible, not how often it happens.

## Results (2026-11-15)

**Verdict, by the rule above: not worth adopting.** D1 was missed. D2 was named only as a limit to leave in place,
which this record counts as a miss, so the rule's "partial" was not reached. Read as a catch instead, the verdict
would be "partial": an optional step, with a repeat run first. Either way, EGM's review is not added to the
skill-doc process.

**The runs.**
- Three runs, at 00:05 to 00:09 on 2026-11-15, about four minutes each.
- The model was `claude-sonnet-5-5`, the CLI default, which is also the model the FC1 agents used. The cost was
  about $1.10 per run, $3.40 in all.
- Each ran stages 5 to 8. Stage 9 was skipped because the skill has no implementation plan, as EGM's rules say
  for a document without one.
- Each ran one critic round, with no resolve step and no convergence loop, and reviewers were dispatched directly.
- **Records:** `~/projects/benchmarks/eg2/` holds the reports, the stream logs, the launch script and its settings,
  with hashes in `SHA256SUMS`. EGM's commit is in `EGM_COMMIT`.

**Isolation held.**
- Every Read, Glob, Write and Agent call stayed inside the run folder and the plugin. No call was denied.
- One thing was in context: the auto-memory index of `~/workspaces`, the archived specbound v1, which the session
  loads as an additional directory. It has no FC1 content (a search for `fc1`, `outside input` and `@input` finds
  none).
- The word "FC1" appears in the logs only because the skill's own heading says "Outside inputs (FC1)".

### D1 at V1: missed

None of the 7 reviewers said that the tagging can be skipped, or that nothing holds `done` until it is done. The
outside-input paragraph drew three kinds of comment:
- **Altitude.** Zoom-out: "Altitude drops into tool-level detail (... FC1 block in step 7, lines 103-113 ...). This
  detail belongs in the tool descriptions." The consolidated action A3 is "Remove or point to the owning tool for
  the detail in ... the FC1 block".
- **Undefined terms.** "FC1", `@input` and `@unknown` are not defined (critic S-7, comprehension 1), and the run-in
  label is not a heading (S-11).
- **Choices.** "Stop, Wait ... Go ahead" are named without being introduced (comprehension, lines 110-114).

The run did look for this kind of fault. Action A4 is "Make unchecked high-risk obligations enforceable", and it
lists the acceptance step, step 3b, 5b and faithful translation. It does not list the tagging, which is the step
all four first-trial runs then skipped (eight inputs between them; corrected from "eight agents"). The one unchecked, act-first instruction in the paragraph ("tag every step that depends
on an outcome from outside the system") has no **checked** mark, so it never reached that list.

### D2 at V2: named, then advised against fixing (counted a miss)

- **The weakness was named.** Critic D8: "`@inputs none` is checked only by a human reading the claim."
- **It was kept as a limit.** Consolidated action A8: "Record as known limits, not new mechanisms: ... `@inputs
  none` checked only by human reading."
- **The Stage 8 zoom-out decided against fixing it.** "Critic D5, D7, D8 (...): record as known limits rather than
  add mechanisms."

No report says what the agent trial found: the agent chose the "none" itself, and the instruction to show it to the
person was not followed. An author acting on this report would have written the limit down. What fixed it was
FC1 step 4, a gate that makes the person confirm.

The pre-registration's D2 wording would accept "checked only by a human reading", so it could be scored a catch,
and the verdict would then be "partial". This record does not count it, for the reason stated under "What counts as
a catch": the finding has to lead a reader to act in the right direction, and this one advised against acting. The
quotes above are complete, so the owner can re-score.

### D3: missed

No finding in V1 or V2 asks whether a lookup, a role check or a "paid" counts as outside the system.

### Noise and direction

- **Where the effort went.** Each run produced about 25 to 40 findings, and two to four of them concern the
  outside-input paragraph. Most were about document form:
  - the feature-spec standard's missing sections, which the V1 run then declined to apply;
  - splitting the skill into several;
  - undefined vocabulary;
  - the 280-character limit;
  - how many questions get asked.
- **The direction is the opposite of what worked.** All three runs recommend moving the outside-input rules out of
  the skill into tool documentation, leaving "one rule: tag outside-dependent steps" (V2, A3). The FC1 records show
  that one rule alone is what failed. What raised the tagging rate was a gate that holds `done`, together with
  explicit text naming it (step 3).
- **Questioning more.** "When in doubt, tag: one question too many beats a missed one" was criticised as
  favouring over-asking (V2 zoom-out 3, critic D8). That pushes against the owner's false-positive ruling.

### V3: the prediction for `fc1-agent3`

On the outside-input text, the V3 reports raise only three things:
- altitude: move the `@input` syntax and the reason codes to tool docs;
- volume: group the tagged inputs into one question per shared guard, with a defaults-first guess;
- a duplicated sentence: "Write none only when ..." restates the `@inputs none` rule (T10).

None predicts that the agents will miss an input, write a wrong "none", or that the person's confirmation will be
skipped or rubber-stamped. Scoring, when `fc1-agent3` reports:
- **Hit** for volume, if the runs show too many input questions.
- **Miss** for each outside-input failure it does show. Examples: U3 untagged again, a wrong "none" confirmed, or a
  confirmation skipped.

### What this says

- **EGM's personas are good at document form.** They found real things a writer would fix: contradictions between
  caps, undefined terms, missing pointers. They do not model an agent reading a skill and choosing what to skip.
  The defect FC1 found is behavioural, and here only running agents showed it.
- **This fits the limit stated in advance.** EGM is built to review design docs, not instructions for agents.
- **Nothing is changed in specfence.** Its own fresh readers (EG1) remain, and their seeded-defect measurement is
  still owed. EGM stays a companion: specfence detects it and offers it, upstream and downstream, as built in EG1.
- **Limits:**
  - one run per version;
  - scored by the session that knew the answers;
  - Sonnet as the reviewer model. A stronger model might do better, and that is untested.

## V3 scored against `fc1-agent3` (2026-11-15)

`fc1-agent3` reported in 8a62436: pool 4 of 4, expenses 0 of 4.
- Both expenses runs wrote `@inputs none`.
- Step 4 put each "none" to the driver, who answered `Yes, nothing outside` both times.

Against the V3 prediction:
- **Volume: no hit.** The record shows no excess of input questions. Each run asked one per input, and expenses
  asked none.
- **A wrong "none": miss.** No V3 finding predicted that the agents would still write it.
- **The person confirming a wrong claim: miss.** No V3 finding said the confirmation could be rubber-stamped.

**V3 has no hits and two misses.** EG2's verdict stands.
