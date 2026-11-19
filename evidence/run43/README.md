# Evidence: run 43, first service run (Saleor + Healthchecks)

Primary sources for the services post. Same honesty header as the other
drawers: run 43 ran under a static-only evidence policy, so every verdict here
is "statically confirmed + blind-adjudicated", never demonstrated on running
code (`demonstrated: false` throughout). The deployment models are UNCONFIRMED,
so multi-threaded sharing is assumed where the verdicts say so.

Names: run 43 set 8 blinded testers on 8 pre-frozen trees (5 Saleor, 3
Healthchecks), one packet each. A claim file (`H1.json` … `S7.json`) holds one
adjudicator's verdicts over the numbered sub-claims; the sub-claim's tree
(`R1`–`R8`) is the packet tree it was judged in, and the third field of each
claim in the run's claims file (not vendored) names the packet and finding
index. `MATCHES-run43.json` is the label matching: per packet, the own fix and
any later (open) fixes against the tester's findings. `run43-SCORES.json` is the
frozen scores file (sha256 `b0164b4e…`).

Counts worth checking against the post: 31 findings, 30 adjudicated real
(only `H4.1` not real); own-fix recall 0/8; one open partial
(healthchecks/pkt-c finding 1 against fix `311f7064dc`), adopted from the
independent label matcher over the curator's initial none. The pre-registration
and frozen scores live in the specfence repository
(`evals/cleanroom/run43/`), which is private for now.
