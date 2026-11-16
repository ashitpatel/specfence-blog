# RH1 pre-registration: a cheap rehearsal before a paid agent trial

**Owner request, 2026-11-15:** "Pre-register the skill lint and the rehearsal". The lint is in
`docs/project/rounds/2026-11-15-SL1-prereg.md`. This record is committed before any rehearsal runs. **Nothing launches
until the owner approves the budget below.**

**Question.** A full agent trial (M-agent) costs about 30 sessions. Can a short rehearsal predict that trial's
outcome? In a rehearsal, one fresh agent gets the skill text and the same prose, with no engine and no person. If it
can, it becomes a pre-check run before every agent trial.

## What a rehearsal is (fixed now)

**The folder.** Each run starts in a new, empty folder outside the repository. The folder holds two things:
- `skills/`: the nine skill files, taken from the tag the trial used;
- `requirements.md`: the trial's prose input, byte for byte.

The prose inputs and their hashes:
- pool: `~/projects/benchmarks/fc1/inputs/pool.md`, sha256 `0593cc6c…6b19d`;
- expenses: `~/projects/benchmarks/fc1/inputs/expenses.md`, sha256 `5ecc28d6…8bac3`.

Both trials used these same files.

**The prompt, verbatim:**

> This folder holds specfence's agent skills (`skills/`) and a requirements document (`requirements.md`). Rehearse
> the work the skills describe for this request: "Turn requirements.md into checked TLA+ and Alloy specs with
> specfence (requirements mode), settle every open question with the requirements owner, and get forge_done to
> done: true." The specfence tools are not available here. Do the work as far as you can without them: write the
> spec files you would write, and write `rehearsal.md` listing, in order, every specfence tool you would call with
> its arguments, every question you would put to the requirements owner, and what each done-gate would report
> according to the skills. Do not ask anyone anything; write the questions down instead.

The prompt does not mention outside inputs, tags or what is scored.

**The command:**
- `claude -p` with `--model claude-sonnet-5-5`, the model both FC1 trials used;
- `--strict-mcp-config` with no servers;
- denied: reads under `~/projects`, `~/workspaces` and `~/.claude/projects`, and Bash and web tools;
- allowed: Read, Glob, Grep, Write and Edit;
- a 30-minute timeout.

**The isolation check.** After each run, its stream log is searched for any path outside the run folder. A run that
read anything else is void, and it is re-run once under a new name.

**A run is void** if it writes no `.tla` file. It is re-run once under a new name.

## Calibration: the FC1 cells, where the answer is known

| cell | skill text | prose | runs | what the trial found |
|---|---|---|---|---|
| C1 | `cleanroom-fc1-v1` | pool | 2 | no run tagged any input (0 of 4 credits) |
| C2 | `cleanroom-fc1-v1` | expenses | 2 | no run tagged any input (0 of 4) |
| C3 | `cleanroom-fc1-v2` | pool | 2 | both runs tagged U1, the new connection |
| C4 | `cleanroom-fc1-v2` | expenses | 2 | 1 run tagged U4, the payment; 2 of 3 wrote a wrong `@inputs none` (one void); none tagged U3 |

**Scoring each run.** The rules are FC1's MA1 rules (`evals/cleanroom/fc1-agent/PREREGISTRATION.md`):
- each of U1 to U4 is credited when the step doing that work carries `@input` with words about it;
- the words are judged by hand and quoted.

Also recorded:
- whether the spec says `@inputs none`;
- whether `rehearsal.md` lists any question about an input being unavailable or failing.

**Agreement for each cell:**
- **C1 and C2 agree** when neither run tags any input.
- **C3 agrees** when at least one run tags U1.
- **C4 agrees** when at least one run writes `@inputs none` or misses U4.

**Pass rule.**
- **The pool contrast must hold:** C1 agrees and C3 agrees. This is the one comparison where the same prose gives
  opposite results under the two texts, so a rehearsal that tags nothing, or tags everything, fails it.
- **And at least one of C2 and C4 must agree.**

**Size:** 8 runs, plus any void re-runs.

## Held-out cases

**There are none that a rehearsal can test.** A search of the run records, excluding FC1, found five clear cases
of an agent not following a skill instruction (they are listed in `docs/project/rounds/2026-11-15-SL1-prereg.md`).
- **Two have small fixtures,** H1 (`evals/agent-loop/expenses`) and H2 (`evals/m2-user-run`). Both failures came
  after the person's answer: an edit going past the answer, and an ambiguous answer applied without asking. A
  rehearsal has no person, so it cannot reproduce either.
- **The other three need large benchmark packets** (klipper, ansible-core, gevent).

**So the evidence is calibration only.** Even a pass is weaker evidence than it would be with held-out cases.
After a pass, every later M-agent trial is scored against the rehearsal's prediction, and those scores are the
held-out evidence. After three such trials, the rehearsal step is reviewed again: it is kept only if it predicted
at least two of the three.

## Decision

- **Calibration passes:** the rehearsal becomes a step before every M-agent launch.
  - Its prediction is written into that trial's pre-registration.
  - The trial's result is scored against it, so the evidence keeps growing.
  - It is reviewed again after three trials, as set out under "Held-out cases".
- **Calibration fails:** the rehearsal is not used, and agent trials stay the only behavioural check. The record
  says which cells disagreed and how.

## Budget (owner approves before launch)

- **Sessions:** 8 calibration runs, plus at most 4 void re-runs. That is at most **12 sessions**.
- **Time:** a 30-minute cap per run. Runs go two at a time.
- **Cost:** an EG2 review run cost about $1.10, and a rehearsal writes more. The estimate is $1 to $3 per run, with
  a **cap of $30**. Launches stop at the cap.
- **Records:**
  - the run folders and stream logs go to `~/projects/benchmarks/rh1/`, with hashes;
  - the results go to `docs/project/rounds/`.

## Limits, stated now

- **No engine.** The trials' agents got gate feedback from the real engine, for example `inputs-undeclared` at v2.
  A rehearsing agent only reads what the skill says the gates do. C3 and C4 test whether reading is enough.
- **No person and no long session.** The skips the trials found came after hours of work. A short rehearsal may be
  more careful than that, or less.
- **One model.** Only Sonnet is tested.
- **The scorer knows the answers** for the calibration cells. Quotes are given in full, so the owner can re-score.
