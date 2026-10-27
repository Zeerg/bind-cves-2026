# Agent Guide

## Scope

This repository contains four isolated BIND 9 vulnerability labs:

- `CVE-2026-19630`: verified bounded resource-exhaustion PoC (measured CPU
  amplification and worker-blocking delay at lab scale).
- `CVE-2026-94300`: verified intermittent sanitizer PoC (ASan
  heap-buffer-overflow and double-free aborts on 9.20.29).
- `CVE-2026-96888`: verified assertion PoC.
- `CVE-2026-97313`: verified assertion PoC.

Read the local `README.md` and `notes.md` before changing a lab. Keep both
documents synchronized with any behavior or classification change.

## Non-Negotiable Safety Rules

- Keep every Docker network `internal: true`.
- Do not publish container ports to the host.
- Keep CPU, memory, process, request, and command limits bounded.
- Do not use production DNS servers, Kerberos realms, credentials, or keys.
- Always run cleanup. Treat incomplete cleanup as a failure.

## Evidence Rules

Do not classify a result from an exit code alone. Require the exact evidence
documented by the lab.

| Lab | Required vulnerable evidence |
| --- | --- |
| CVE-2026-19630 | BIND 9.20.29 accepts the shared-budget, amplified-TCP, and amplified-UDP packets; burns >= 15ms measured `named` CPU per message at >= 10x the fixed server's per-message cost; elevates the queued-query tail (>= 50ms) behind a parallel burst; and produces query timeouts under the sustained stream; 9.20.30 returns `FORMERR` with a flat tail and zero timeouts |
| CVE-2026-94300 | A recognized abort, double-free, use-after-free, ASan, or UBSan marker in the server log |
| CVE-2026-96888 | The expected `db.c` non-NULL database `REQUIRE`, assertion-exit marker, and stopped resolver |
| CVE-2026-97313 | `INSIST(st->r.r_type == type)`, assertion-exit marker, and stopped resolver |

Treat timeouts, response mismatches, pytest failures, and unrelated exits as
inconclusive unless the required evidence is also present.

Each runner preserves raw artifacts under `evidence/<version>/` inside its
lab directory (probe output, server logs, attempt ledgers, and a manifest
with UTC time, git SHA, image IDs, and measured values; the 19630 lab, which
tests both versions in one run, uses `evidence/latest/`). Keep these files
in commits so results can be authenticated without re-running the lab. Do
not document a claimed result that no retained artifact supports.

The CVE-2026-19630 lab proves measured CPU exhaustion and queued-query tail
delay at lab scale; it does not demonstrate a production outage. The
CVE-2026-94300 lab reproduced intermittent sanitizer aborts on 9.20.29 in
two classes (heap-buffer-overflow read and double-free); functional
MIC-corruption failures alone are not accepted as crash evidence. The
double-free confirms the advisory's CWE-415 primitive is reachable.
Non-sanitized runs showed functional race symptoms without a crash, but did
not directly observe heap corruption; code execution remains undemonstrated.
Do not overstate either result.

## Change Workflow

1. Inspect the current runner, probe, Compose file, README, and notes.
2. Compare the fixture with the cited upstream BIND regression test.
3. Make the smallest change that preserves the upstream trigger chain.
4. Validate both vulnerable and fixed paths after behavior changes.
5. Confirm the evidence gate rejects unrelated failures.
6. Confirm cleanup removes all lab containers and networks.
7. Update the local README and notes with directly observed evidence.

## Required Checks

Run these checks for every affected lab:

```sh
docker compose config --quiet
```

Run ShellCheck on changed shell scripts and `python3 -m py_compile` on changed
Python files. After runtime testing, confirm that no lab resources remain:

```sh
docker ps -a --filter name=cve-2026-
```

Use the public `run.sh` entry point for final runtime evidence. Do not replace
a parent-side run with an agent summary.

## Repository Hygiene

The repository root is also a larger local workspace. The allowlist
`.gitignore` intentionally tracks only this guide, the root README, and the
four CVE directories. Do not force-add unrelated workspace files.
