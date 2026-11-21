# ST2 scores: structural calibration (2026-11-20)

Prereg: `docs/project/rounds/2026-11-20-ST2-plan.md`.
Labels: `cases.json` (21 triage reads, frozen before ranking runs).
Runs: `runs/*.json` (6 trees + 6 mutant clones). Gate: `gate/`.

## Arm P (triage precision): 2/21 ≈ 0.095 — STOP RULE FIRED

Prediction was ≥ 0.3; the < 0.15 stop rule fires. Per the prereg, ST2 delivers
precision + gate only; ranking/ablation is deferred with cause (below).

Reals: T5 (RPC unregister routes-leak on cancel-throw), T11 (MemoryDataStore
stale `_schedules_by_task_id` + leak). Everything else: single-writer pairs
with no divergence (T2, T10, T12, T13, T14), init co-writes (T1), maintained
invariants (T3, T4, T6, T7, T8, T9, T16), doc prose with no property (T15, T18,
T19, T20), bare shape (T17), temporal-not-structural (T21).

Precision by rule: co-write 2/13 (both where a partial writer genuinely
diverges); invariant-doc 0/6 (every fire was prose without a property);
parent-link 0/1 (shape alone). The finder is noise-dominated by
single-writer/init pairs and property-less doc/shape fires.

## Arm R (generation diagnostic; NOT scored — stop rule fired first)

Mutants proposed at any rank: 5/6 (m1, m2, m3, m4-fixed, m6 yes; m5 no).
M4's first miss was an invalid mutant (empty loop body = SyntaxError, file
unparsed); fixed (pass body) it proposes tr=1. M5's miss is genuine: deleting
the joint cache write removed the co-write evidence, so B01.code.shared
vanished while its unmutated twin stayed. Breaking sync can un-propose —
a generation-recall shape ST3 must close (rank the pair, not the joint write).

## Arm A (ablation): DEFERRED with cause

Tuning rank constants on 2/21 precision is tuning on noise. No constant ships;
the baseline stands. Owed: a generation-precision round first (fence init-only
and single-writer pairs; require a checkable property for invariant-doc and
parent-link fires), then re-ablate.

## Arm G (ST1 gate): HOLDS, with one false refute

| case | verdict | hand-reading |
|---|---|---|
| m1 (injected) | confirmed | true |
| m2 (injected) | confirmed | true |
| m3 (injected) | confirmed | true |
| m4 (injected omission) | uncertain | fence case: stutter, 0 attributed steps |
| T5 (real) | confirmed | true (pops precede the raisable await) |
| T11 (real) | refuted | FALSE: model faithful, index misses `self.d[k].add()` writes (read-only) |

Confirm-rate 4/6 = 0.67 ≥ 0.50 ✓. Confirmed-precision 4/4 = 100% ✓.
ST1's claim stands (no narrowing to mapping-level needed).

## Engine bugs found (measured, not fixed)

1. **Index misses subscript-receiver mutators** (`index/concurrency.py:4324`):
   `self.d[k].add(v)` records read×2, no write (direct-attribute `.append`
   and subscript assigns work). Caused T11's false refute; affects temporal
   races through the same shape. Minimal probe: /tmp/idxprobe (3 methods).
2. **Index misses a subclass-method inherited-attr write** (T12):
   `AndTrigger.next` writes `_next_fire_times`, absent from the writers map.
   Unpinned (observation only).
3. **ST1 op→method mapping is name-rigid**: variant ops (`Op_unregister_throw`)
   resolve nothing; models must name ops exactly after methods (may-semantics
   documented per op instead).
4. **ST1 cannot attribute omissions**: stutter traces (M4) yield 0 attributed
   steps → uncertain. Fenced, not wrong; expected-write checks are ST3 work.
