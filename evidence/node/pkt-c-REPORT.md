# First look: bull (Node.js job queue over redis), code/ at current commit

Service `bull` (package.json name), version 0.4.0. Library (no deployment
configuration found); all checks ran under profile `library`, which stays
UNCONFIRMED (no person available; the two library questions below are owed).
Static only: nothing was executed; the strongest evidence any finding reaches
here is "statically confirmed", and none reaches it (see why under each
finding). Verdicts below carry their bounds, profile line and neighbours as
returned. Branch conditions not checked.

## 1. Bugs

None. No counterexample confirmed, so no finding was recorded as a bug or a
deterministic defect. The four findings in section 2 are model-level.

## 2. Model-level findings

Each of these may be an abstraction artefact. Each was falsified in the model
(TLC violation below) and each holds in a control spec modelling the defect
removed (verdicts quoted). I checked every spec's Init by reading against the
code (empty `timers`/`idle=true` at construction, timer entries as armed,
`paused=false`).

### ML-1. Concurrent jobs cancel each other's lock renewal (shared 'lockRenewer' timer name)

`Queue.processJob` (lib/queue.js:513) arms its lock renewer under the single
hard-coded name `'lockRenewer'` (:518) and clears that name when its own
handler settles (:557). `Queue.run` (:379) starts one `processJobs` chain per
concurrency slot (:384), so with `concurrency > 1` two chains share one
`TimerManager` entry. When job A finishes while job B is still awaiting its
handler, A's `clear('lockRenewer')` clears B's renewer: B's lock (`PX`,
5 s, job.js:78-86) then expires mid-processing and another worker's
`processStalledJob` (queue.js:477) can take it and process B a second time.
The `TimerManager` header comment assumes "two (transient) timers", i.e. one
renewer; `process(concurrency, handler)` breaks that assumption by design.

- Spec: `LockRenewer.tla`, property `RenewerArmed` ("while a chain is
  processing its job, the stored lock-renewer handle is that chain's own").
- Verdict: `violated`, bounds `Threads=2`, profile `library (UNCONFIRMED): a
  library; used by: unknown; python runtime: unknown`, neighbours none,
  detail `Error: Invariant RenewerArmed is violated`, 5 distinct states.
- Trace: `Arm(1)` (entry 0->1), `Arm(2)` (orphan {}->{1}, entry 1->2) —
  chain 1 is running while the entry holds chain 2's handle.
- Control `LockRenewerControl.tla` (per-chain names, the fix): `clean`,
  9 states; `ArmedWhileRun` and `NoRenewAfterDone` both falsified ("Finish:
  freeze update of pc", "Fire: drop guard 't in orphan'"). The
  single-chain isolation cfg is clean too (3 states, both properties
  falsified), with Fire dead by design under its `@control` marker.
- Static confirmation `b7b012ad115a`: `uncertain`. Reasons: the step's
  functions are reached from no execution context the facts know (promise
  `.then` dispatch and user-called `process()` are not modelled); `entry`
  and `orphan` are unclassified (the Node index resolves no attribute tag,
  so no resolvable tag names `this.timers`). Switch checks pass under the
  Node memory model. Kind: not decided for Node (N4). Assumptions: the Node
  memory model text; ghost variables not compared; lock identity by name.
  Not checked: value flow through calls more than three deep; which branch
  of an action the code takes. Branch conditions not checked.
- Recorded as model-level finding, catalogue `interleaving`, evidence
  model-level, confirmation verdict uncertain.

### ML-2. Orphaned lock-renewer timeout renews a finished job's lock and re-arms itself

`TimerManager.set` (lib/timer-manager.js:47-50) overwrites `timers[name]`
without `clearTimeout` on the dropped handle. So when chain B arms
`'lockRenewer'` over chain A's pending timeout, A's handle stays live; when
it fires it runs the `lockRenewer` closure (queue.js:516-519), which renews
A's lock even after A finished and re-arms itself under the same name — a
self-perpetuating timeout that also flips `idle` back to false after close.

- Spec: `LockRenewer.tla`, property `NoRenewAfterDone` ("a finished job's
  lock is never renewed again").
- Verdict: `violated`, bounds `Threads=2`, same UNCONFIRMED library profile
  line and no neighbours, 15 distinct states.
- Trace: `Arm(1)`, `Arm(2)` (orphan {1}), `Finish(1)` (entry 2->0),
  `Fire(1)` (renewedAfterDone FALSE->TRUE, orphan {1}->{}).
- Controls: same `LockRenewerControl.tla` run as ML-1 (`clean`, property
  falsified there); single-chain cfg clean.
- Static confirmation `76ac623c3627`: `uncertain`, same two structural
  reasons (no execution context; unclassified `entry`/`orphan`). Locations
  resolved exactly: Arm lib/queue.js:518, Finish :557, Fire :517. Kind not
  decided for Node. Same assumptions and not-checked list as ML-1. Branch
  conditions not checked.
- Recorded as model-level finding, catalogue `async-ordering`, evidence
  model-level, confirmation verdict uncertain.

### ML-3. close() disconnects while the fired delay timer's redis work is still in flight

The `delayTimer` fire callback (lib/queue.js:408-414) runs
`timers.clear('delayTimer')` (:409) and only then calls the async
`updateDelaySet` (:410). The clear drains `whenIdle` listeners synchronously
(timer-manager.js:64-69), so a `close()` waiting at queue.js:177 starts
`disconnect()` while the timer's own redis round trip is still outstanding —
the exact failure the `TimerManager` header comment says the idle gate
exists to prevent ("we can't close queues while these timers are active").
A close arriving just after the fire takes the same path via the immediate
`whenIdle` branch.

- Spec: `DelayClose.tla`, property `CloseAfterWork` ("a close disconnects
  only after the fired timer's scheduled redis commands have completed").
- Verdict: `violated`, no bounds declared, same UNCONFIRMED library profile
  line and no neighbours, 8 distinct states.
- Trace: `Tfire(1)` (entry 1->0), `Close(2)`, `Disconnected(2)` (discDone
  FALSE->TRUE) with `redisDone` still FALSE.
- Control `DelayCloseControl.tla` (clear moved behind the redis round
  trip): `clean`, 9 states; `CloseAfterWork` falsified ("Disconnected: drop
  guard 'pc[t] = \"disconnected\"'").
- Static confirmation `7f557e7bc45d`: `uncertain`: Tfire's functions are
  reached from no execution context the facts know; `entry`, `waited`,
  `discDone` unclassified (same attribute-tag gap). Locations exact:
  lib/queue.js:409, :177, :163. Kind not decided for Node. Same assumptions
  and not-checked list. Branch conditions not checked.
- Recorded as model-level finding, catalogue `async-ordering`, evidence
  model-level, confirmation verdict uncertain.

### ML-4. pause() then resume() leaves two priority-queue processing loops running

`PriorityQueue.pause` (lib/priority-queue.js:153) only sets the `paused`
flag (:156); the in-flight cycle keeps running until its boundary check
(:128). `resume` (:163) clears the flag (:165) and calls `run()`
unconditionally (:170). A pause immediately followed by resume, while a
cycle is inside its awaits (e.g. a slow job handler at :115), starts a
second `loop()` chain while the first is still cycling — and the first
chain's next boundary check sees `paused == false` and keeps looping too.
Two loops then process concurrently. (Secondary observation, not modelled:
this path calls `processJob` at :115 without the `takeLock` gate the plain
`Queue` path has at queue.js:502, so an overlapping loop — or another
process's stalled scan in the window before the renewer's first SET lands —
can process one job twice.)

- Spec: `PrioLoop.tla`, property `MaxOneLoop` ("at most one processing loop
  is active per priority queue").
- Verdict: `violated`, no bounds declared, same UNCONFIRMED library profile
  line and no neighbours, 9 distinct states.
- Trace: `Pause(2)`, `ResumeStart(2)`, `ResumeRun(2)`, `Lstart(3)` — two
  chains in `cycleend`.
- Control `PrioLoopControl.tla` (guarded `run()`, the fix): `clean`,
  11 states; `MaxOneLoop` falsified ("CycleEnd: drop guard
  'pc[t] = \"cycleend\"'").
- Static confirmation `535b1c3866f5`: `uncertain`: ResumeRun's functions
  reached from no execution context; `paused` unclassified (same gap).
  Locations exact: lib/priority-queue.js:156, :165, :170, :134. Kind not
  decided for Node. Same assumptions and not-checked list. Branch
  conditions not checked.
- Recorded as model-level finding, catalogue `interleaving`, evidence
  model-level, confirmation verdict uncertain.

## 3. Structural weaknesses

Census (structural weaknesses, not bugs). Boundary: "this tree holds
2 package(s), 6 JS/TS file(s), 2 bundle(s)"; history "none: the Node census
reads no history (N1)"; switch note "Node: no interpreter lock; workers are
OS threads on separate heaps, the main thread one cooperative loop";
"Node: no detector for SQL writes, state machines, retries, timeouts,
consumers or in-process state; those counts are 0 by absence, not measured
(N1)". Counts: 0 import cycles, 0 check-then-act, 0 retries, 0 workers,
0 lock sites, 0 multi-writer attributes/states, 0 mismatches; concurrency
0 candidates / 1 context (caller) / 0 questions / 307 unresolved calls.
Note: `clone_boundary_report` on this tree answers through the Python path
("this clone holds 0 modules ... python_target 3.x"), which does not
describe a Node tree; the census boundary above is the applicable one.

Data model: refused — "no schema found: no CREATE TABLE in the code or in
.sql files (ORM models are not read yet)". The redis key layout (wait /
active / delayed / completed / failed) is documented only in prose
(lib/queue.js:19-31) and was read, not checked.

Tool-signal provenance for the record: `propose_slices` sources were "0
race candidates, 0 sharing questions", "0 bug patterns", "history from 136
non-merge commits", "module risk prior over 2 modules (top score 0)",
"the sequential track is Python only (I1)", "0 structural candidates".
F1 is not asked on Node trees (E8); F2 has no objects and no residual
questions; F4 lists no operations; F5 best-guesses every failure "no".

## How you chose each slice

The proposer returned zero proposals on all four tracks (budget in force:
depth_slices 3, screen 50/track), so there was nothing to screen and nothing
to skip; the screen reminder is empty. All depth work is own reading, using
all 3 budget slots (recorded as own-8de9ec2bb6, own-6efbfc89f2,
own-ddf30af57e). I read all four lib files fully (1,387 lines with index
and gulpfile). I chose: (1) the shared lock-renewer name — the hot path of
every concurrent queue, breaking lock safety; (2) delay-timer clear-before-
work vs close — contradicts the module's own documented contract; (3) the
priority pause/resume double loop — a second active loop is never intended.
Read but not modelled: job.js lock scripts and the addJob/pauseResume Lua
scripts (each is one atomic server round trip, no interleaving surface);
`empty()`/`clean()` (documented best-effort, with an atomicity TODO);
`TimerManager.clear`'s synchronous listener drain (a re-arming listener
would break idle accounting, but no in-tree `whenIdle` caller re-arms —
only `close()` registers); the priority path's missing `takeLock`
(recorded as ML-4's secondary observation).

## The specs you wrote

- code/lib/.specforge/LockRenewer.tla — ML-1/ML-2 main spec (2 chains, shared renewer name, orphan fire).
- code/lib/.specforge/LockRenewer.cfg — Threads=2, checks RenewerArmed (violated).
- code/lib/.specforge/LockRenewer_orphan.cfg — Threads=2, checks NoRenewAfterDone (violated).
- code/lib/.specforge/LockRenewer_control.cfg — Threads=1 isolation control, @control-marked (clean).
- code/lib/.specforge/LockRenewerControl.tla — per-chain names control (clean; Fire dead by design).
- code/lib/.specforge/LockRenewerControl.cfg — Threads=2 control cfg, @control-marked.
- code/lib/.specforge/DelayClose.tla — ML-3 main spec (timer fire vs closer).
- code/lib/.specforge/DelayClose.cfg — checks CloseAfterWork (violated).
- code/lib/.specforge/DelayCloseControl.tla — clear-after-completion control (clean).
- code/lib/.specforge/DelayCloseControl.cfg — control cfg.
- code/lib/.specforge/PrioLoop.tla — ML-4 main spec (pause/resume/old+new loop).
- code/lib/.specforge/PrioLoop.cfg — checks MaxOneLoop (violated).
- code/lib/.specforge/PrioLoopControl.tla — guarded-run() control (clean).
- code/lib/.specforge/PrioLoopControl.cfg — control cfg.

## Questions for the person

- Library, node_runtime: which Node.js major version runs this code?
- Library, library_usage: is this a library, and how do applications call it — worker threads, promises on one loop, or one thread only? (Confirm the `library` deployment model.)
- F4: no waiting operations were detected; is there any operation that must return once called (and within how many seconds)? F5: must the code survive worker kill, connection drop, silent peer, peer/broker restart, process restart, hostile peer, SIGTERM (drain/kill/no)?
- Is `Queue.process` used with concurrency greater than 1 (ML-1/ML-2 reach)?
- Is `close()` called while delayed jobs or timers may be pending (ML-3 reach)?
- Is `PriorityQueue.pause/resume` used with jobs in flight, and is a second processing loop acceptable there (ML-4)?
- Is duplicate processing of one job (lock steal) acceptable (at-least-once), or must processing be exactly once (decides ML-1 severity)?

## Confidence

- ML-1: high. The mechanism is synchronous code reading (one hard-coded name, clear-whatever-is-stored) plus a public API (`process(concurrency, handler)`) that overlaps jobs; the interleaving needs only ordinary promise scheduling. Capped at model-level by the tool gaps below, not by doubt in the trace.
- ML-2: high. `set()` provably drops the old handle uncleared (3 lines), and the orphaned closure provably re-arms; deterministic once two jobs overlap.
- ML-3: high. Clear-before-async-work is in one callback (:409 before :410) and the drain runs the disconnect listener synchronously; the race is between two ordinary async completions.
- ML-4: medium-high. The code reading is exact, but it needs pause-then-resume during an in-flight cycle, and priority-queue is not the package's main export (used by tests).
- Why nothing confirmed: two engine gaps on this Node tree, both evidenced in §2 records — the extractor records no instance-state accesses and no promise-await windows (307 unresolved calls; `this.timers`/`this.paused` invisible), and user-called public methods sit in no execution context — so every confirmation ends `uncertain` however precise the model. The counterexamples themselves map line-for-line to code I read.

## Coverage map

(The `coverage` output, unedited:)
{
 "modules": [
  {
   "bundle": "lib",
   "members": [
    "lib"
   ],
   "state": "examined",
   "depth": "partial spec",
   "specs": [
    "lib/.specforge/DelayClose.tla",
    "lib/.specforge/DelayCloseControl.tla",
    "lib/.specforge/LockRenewer.tla",
    "lib/.specforge/LockRenewerControl.tla",
    "lib/.specforge/PrioLoop.tla",
    "lib/.specforge/PrioLoopControl.tla"
   ],
   "exported": 3,
   "modelled": [],
   "unmodelled": [
    "Job",
    "Queue",
    "TimerManager"
   ],
   "stale_tags": [],
   "risk": 0.0,
   "modelled_private": [
    "lib.priority-queue.$init.pause",
    "lib.priority-queue.$init.resume",
    "lib.priority-queue.$init.run",
    "lib.queue.$init.close",
    "lib.queue.$init.disconnect",
    "lib.queue.$init.processJob",
    "lib.queue.$init.processJob.lockRenewer",
    "lib.queue.$init.updateDelayTimer",
    "lib.queue.updateDelaySet"
   ],
   "skeletons": [],
   "unparsed_files": [],
   "py2_rewritten": [],
   "file": null,
   "parse_error": null
  },
  {
   "bundle": "(default)",
   "members": [
    "(default)"
   ],
   "state": "examined",
   "depth": "census only",
   "specs": [],
   "exported": 0,
   "modelled": [],
   "unmodelled": [],
   "stale_tags": [],
   "risk": 0.0,
   "modelled_private": [],
   "skeletons": [],
   "unparsed_files": [],
   "py2_rewritten": [],
   "file": null,
   "parse_error": null
  }
 ],
 "parse": {
  "files": 6,
  "unparsed": 0,
  "py2_rewritten": 0,
  "parsed_directly": 6,
  "unparsed_files": [],
  "py2_rewritten_files": [],
  "note": "every file was parsed"
 }
}
