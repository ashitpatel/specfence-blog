# Evidence: bull (Node.js job queue) — part 20

Primary sources for the Node deep-dive, which reads the bull halves of runs
38 and 41 file by file. All verdicts below are static: NRUN ran under a
static-only evidence policy, so every finding is "statically confirmed +
blind-adjudicated", never demonstrated on running code (`demonstrated:
false` in every verdict file). The deployment models are UNCONFIRMED, so
caller threading is assumed where the verdicts say so. The post states both
limits; this folder lets you check them.

Names: runs 38–42 are the five Node cleanroom experiments (8 packets each);
a packet is one tester's sealed session on one tree; a claim file
(`run38-B6.json` style) holds one finding-cluster with its blind verdict.

## Runs

- **Run 38** (8 packets, socket.io + bull, 4 fixes each), engine `f891c97`,
  Nov 2026. Design, frozen scores and analysis:
  - `evals/cleanroom/run38/PREREGISTRATION.md`
  - `evals/cleanroom/run38/SCORES.json` (sha256 `9388ddbb…`)
  - `evals/cleanroom/run38/ANALYSIS.md`
- **Run 41** (8 packets, bull + async + ava), engine `f891c97`, Nov 2026:
  - `evals/cleanroom/run41/PREREGISTRATION.md`
  - `evals/cleanroom/run41/SCORES.json` (sha256 `44b16beb…`)
  - `evals/cleanroom/run41/ANALYSIS.md`
  - Program record: `docs/project/rounds/2026-11-15-NRUN-program.md`
    (+ amendment A1), results: `docs/project/rounds/2026-11-16-NRUN-results.md`

Paths are relative to the specfence repository
(https://github.com/ashitpatel/specfence, `main` branch).

## Packet trees quoted (pre-fix exports, git checkouts)

| packet | tree HEAD | date | own fix |
|---|---|---|---|
| run38 pkt-a | `7b7d04c8f4` | 2016-10-15 | `c8e8ceeef0` |
| run38 pkt-b | `23ce0af636` | 2015-08-14 | `c056cb2a00` |
| run38 pkt-c | `c056cb2a00` | 2015-08-21 | `8212cdde5f` |
| run38 pkt-d | `adf279bb11` | 2016-11-13 | `456b13d1b9` |
| run41 pkt-a | `7484e9ff06` | 2016-12-16 | `d42e60c9bc` |
| run41 pkt-b | `3492ea58c3` | 2016-11-15 | `6ccb8608a2` |
| run41 pkt-c | `49eb5eb0eb` | 2016-12-02 | `c1b42a022f` |
| run41 pkt-d | `a72d478a33` | 2017-02-13 | `0c611de963` |

## Files

- `MATCHES-bull.json`: the blind label matcher's verdicts on all 8 bull
  packets (own + open matches with reasons).
- `pkt-c-REPORT.md`: run38 pkt-c's full tester report (absolute paths
  shortened), the packet that matched its own fix exactly.
- `run38-B6.json`: the shared-`lockRenewer` cluster (2 subclaims, both
  real, race) — the exact own match.
- `run38-B5.json`: the in-place `reverse()` cluster (2 subclaims, both
  real, deterministic) — same packet, unlabelled.
- `run38-B1.json`: the NOSCRIPT-retry cluster (2 subclaims across 2
  packets, both real, deterministic).
- `run41-B9.json`: the `whenCurrentJobsFinished` cluster (3 subclaims
  across 3 packets, all real, race) — found independently three times.
