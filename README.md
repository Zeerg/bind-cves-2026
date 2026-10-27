# BIND CVE Labs 2026

Four isolated Docker labs compare vulnerable BIND 9.20.29 with fixed BIND
9.20.30. Each lab requires specific evidence and uses only internal networks.

| CVE | Result | Required exposure |
| --- | --- | --- |
| [CVE-2026-19630](CVE-2026-19630/README.md) | Measured CPU amplification and DNS timeouts under bounded load | An untrusted DNS message reaches the resolver |
| [CVE-2026-94300](CVE-2026-94300/README.md) | Intermittent ASan heap-buffer-overflow and double-free aborts; no code execution demonstrated | GSS-TSIG, a live negotiated key, and concurrent signed traffic |
| [CVE-2026-96888](CVE-2026-96888/README.md) | Reliable assertion failure and resolver exit | A redirect zone with strict QNAME minimization |
| [CVE-2026-97313](CVE-2026-97313/README.md) | Reliable assertion failure and resolver exit | Two RPZ zones with mixed `recursive-only` settings and RD=1/RD=0 queries |

Each CVE directory contains its runner, Docker topology, evidence gates, and
research notes. Read its `README.md` and `notes.md` before running the lab.
The full source review is in
[`bind-9.20.30-vulnerability-research.md`](bind-9.20.30-vulnerability-research.md).
Repository rules are in [`AGENTS.md`](AGENTS.md).

## Safety

- All Compose networks are internal.
- No service publishes a host port.
- Probes use fixed lab-only targets.
- Resource and time limits bound each lab.
- CVE-2026-94300 does not pursue code execution.

Run these labs only on systems that you own or have permission to test.

## Acknowledgements

These labs were developed and reviewed with
[TurenOS](https://turen.io/turenos), a local-first security-engineering
workbench. Shoutout to Daybreak for the support and collaboration.
